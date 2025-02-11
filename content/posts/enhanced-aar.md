---
author: alex
category:
  - sdwan
  - networking
date: "2025-01-05T14:20:31+00:00"
title: Enhanced Application Aware Routing
draft: true
url: /enhanced-aar/

---
## Introduction

In my previous [series of posts](/demystifying-aar-1-3-the-foundations), I explored **Application Aware Routing (AAR)** in depth, a key SD-WAN technology that steers traffic over best-performing paths. While AAR has been a fundamental capability for years, the evolution of networking brought new ideas to enhance its effectiveness. This led to the introduction of **_Enhanced_ Application Aware Routing (EAAR)**. 

## AAR Limitations

Before diving into EAAR, let's understand why it was created.

The current AAR implementation measures the path quality using BFD, sending probes at a defined interval (1s by default). Loss, latency and jitter are derived from those packets and these values are placed into rotating buckets to calculate an average tunnel health metric. This process would typically take between 10 to 60 minutes and with some configuration tweaks we could achieve times of 2 to 10 mins.

For those networks that require faster detection, some challenges arise:
- Lowering the hello interval to less than 1s affects the device's tunnel scale
- Lowering bfd multiplier and poll intervals could lead to false positives, switching traffic even with transient network conditions.
- Switch traffic back and forth, there is no mechanism to determine if a transport is stable again after a network degradation event. 

## Enhanced AAR

_So, what is EAAR and how it improves its predecessor?_

In a nutshell these are the advantages:
- Improves performance measurements using <u>inline data</u>. In other words, the data plane packets are used to measure loss, latency and jitter.
- Steer traffic in <u>seconds</u> rather than minutes
- Dampening implemented for <u>stability</u> purposes, ensuring transports are stable before forwarding traffic through them. 
- More <u>accurate measurement</u>s of loss, latency and jitter

### Taking a deeper look 

When EAAR is enabled, data packets will be used to measure loss, latency and jitter. Let's understand the key differences: 

#### Loss measurement 

The SD-WAN routers will use inline data along with IPSEC sequence numbers to measure loss. 

There is an in-built mechanism that allows the routers to determine if the loss is local to the router (**local loss** - typically due to QoS drops) or external to the router (**WAN loss** - any packet loss outside the router). To calculate the local loss, the router will determine the amount of packets it generated against the amount of packets that actually were sent. To get the WAN loss, peer SD-WAN routers will report the amount of packets received and will use BFD (Path Monitor TLVs) to share this information back to the originating router. This is done per tunnel. 

Up to this point, there is an important improvement in how loss measurement is done, however, this could be further improved by leveraging per queue loss measurements. To achieve this, we need to associate an SLA class with an App Probe Class. Let's see an example. 

![](/wp-content/uploads/2025/02/app-probe.png)

With this App Probe Class, the router will use (and generate BFD) packets with DSCP 18, mimicking less important traffic that will be subject to different rules and paths on the local and external routers. This will provide a more accurate measurement of loss for this type of traffic. 

If there is no inline data, BFD is used to get measurements.

**Note** If using GRE, per queue measurement is not available. 

Here is a visual to better understand how loss will be measured depending on multiple factors

|Encapsulation  | App Probe Class |Measurement type | Public tunnels                        | Private Tunnels                           |
|---------------|-----------------|-----------------|---------------------------------------|-------------------------------------------|
| IPsec         | Yes             | Per SLA         | Total WAN loss + local loss per queue | WAN loss per queue + local loss per queue |
| IPsec         | No              | All SLAs        | Total WAN loss + total local loss     | Total WAN loss + total local loss         |
| GRE           | -               | All SLAs        | Total WAN loss + total local loss     | Total WAN loss + total local loss         |

#### Latency 

To measure latency, the router will simply calculate the time taken to send and receive packets between source and destination devices. Inline data is used and it can get to App Probe granularity.

#### Jitter

Another worth mentioning change is that the jitter is computed per direction (receive or transmit). The jitter is computed at the receiver and reported to the sender using BFD TLVs. Inline data is used and BFD is the fallback mechanism if no data traffic is available. 

### SLA Dampening
One of the benefits of EAAR is steering traffic in seconds rather than minutes, but what would happen if there are transient network conditions causing the transports to not meet the sla every few minutes? Traffic would be constantly switching between transports which is not a desirable scenario and the reason why dampening was introduced. 

The general idea is that when a transport link goes out of compliance, traffic is rerouted to an alternate path. Once the transport becomes compliant again, the device **does not** immediately move traffic back. Instead, it starts a timer to ensure the link remains stable for a specified period before reusing it.

In the end, dampening helps prevent unnecessary traffic shifts, which could negatively impact performance due to transport instability

## Configuring EAAR

To enable EAAR we have three predefined options:

| Mode         | Poll Interval | Poll Multiplier         | Dampening Multiplier |
|--------------|---------------|-------------------------|----------------------|
| Aggressive   | 10s           | 6 (10s-60s)             | 120 (20 mins)        |
| Moderate     | 60s           | 5 (60s-300s)            | 40  (40 mins)        |
| Conservative | 300s          | 6 (300s-1800s)          | 12  (60 mins)        |

**Note** To use custom timers, configuration needs to be done through CLI templates.

The fundamental principle behind EAAR is similar to AAR in that they will both use rotating buckets and calculate average values for loss, latency and jitter. With the _Aggressive_ mode, traffic would take between 10-60 seconds to shift, depending on how severe the impairment is. The dampening window (poll interval x dampening multiplier) is 20 minutes, meaning that before switching traffic back to a transport, it needs to be stable for 20 minutes. 

In my lab, I am using Configuration Groups, however this is [available through templates](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/m-enhanced-application-aware-routing.html#configure-enhanced-application-aware-routing-using-a-feature-template-in-cisco-sd-wan-manager) as well. 

![](/wp-content/uploads/2025/02/eaar-config-groups.png)

You could decide to use a variable, instead of a global value, to account for devices that wouldn't be running EAAR. In this case, EAAR enabled devices will use traditional AAR with non-enabled EAAR devices. 

The following config is added to the devices:
```
  bfd enhanced-app-route enable
  bfd enhanced-app-route pfr-poll-interval 10000
  bfd enhanced-app-route pfr-multiplier 6
  bfd sla-dampening enable
  bfd sla-dampening multiplier 120
```
Let's see how it works

## Demo
In my lab, I use Manager 20.16.1 and my devices are running 17.13.1a

**Note** The minimum version required is 20.12/17.12 

Let's start with some verifications after pushing the configuration. 

To check the configured timers and multipliers

```
BR10#show sdwan app-route params
Enhanced Application-Aware routing
    Config:                  :Enabled   
    Poll interval:           :10000     
    Poll multiplier:         :6         

App route 
    Poll interval:           :120000    
    Poll multiplier:         :5         

SLA dampening  
    Config:                  :Enabled   
    Multiplier:              :120    
```

To verify what sessions are using EAAR, look for the _FLAGS_ column
```
BR10#show sdwan bfd sessions alt

                                  SOURCE TLOC      REMOTE TLOC                     DST PUBLIC    DST PUBLIC                                  
SYSTEM IP   SITE ID   STATE       COLOR            COLOR            SOURCE IP      IP            PORT        ENCAP  BFD-LD    FLAGS   UPTIME    
-------------------------------------------------------------------------------------------------------------------------------------------------
1.1.1.20    200       up          biz-internet     biz-internet     30.1.10.2      30.1.20.2     12406       ipsec  20006     EAAR    0:00:20:31 
1.1.1.20    200       up          mpls             mpls             30.2.10.2      30.2.20.2     12366       ipsec  20002     EAAR    0:00:20:38 
1.1.1.20    200       up          private1         private1         30.3.10.2      30.3.20.2     12366       ipsec  20003     EAAR    0:00:20:37 
```  
To get more details about a specific tunnel. 
```
BR10#show sdwan app-route stats summary 
Generating output, this might take time, please wait ...
app-route statistics 30.1.10.2 30.1.20.2 ipsec 12386 12406
 remote-system-ip         1.1.1.20
 local-color              biz-internet
 remote-color             biz-internet
 sla-class-index          0,1,2
 fallback-sla-class-index None
 enhanced-app-route       Enabled
 sla-dampening-index      None
 app-probe-class-list None
  mean-loss    0.000
  mean-latency 0
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      0             0             0        0        0             0             0             0             
1      64            0             0        0        0             0             0             0             
2      0             0             0        0        0             0             0             0             
3      0             0             0        0        0             0             0             0             
4      0             0             0        0        0             0             0             0             
5      64            0             0        0        0             0             0             0             
```

Notice that there is no traffic between my devices, thus the total packet count on each bucket is low.

Same information is available through the Manager's UI, using the _Real Time_ dashboard and using the _App Routes Statistics_ Device Option.  

![](/wp-content/uploads/2025/02/eaar-1.png)

### Scenario 1 - Slight impairment

My lab has two devices with three tunnels: mpls, private1 and biz-internet colors. 

For this first test, I will use the following SLA parameters:

```
SLA_Real-Time 
Loss: 3%
Latency: 150ms
Jitter: 100ms
```
The AAR policy received from the Controller indicate to use mpls as primary, private1 as backup and load-balance when SLA not met. 
I am matching traffic between 172.16.10.0/24 and 172.16.20.0/24. 
```
BR10#show sdwan policy from-vsmart
from-vsmart app-route-policy app_route_AAR
 vpn-list vpn_Corporate_Users
  sequence 1
   match
    source-data-prefix-list      BR10
    destination-data-prefix-list BR20
   action
    backup-sla-preferred-color private1
    sla-class       SLA_Real-Time
    no sla-class strict
    sla-class preferred-color mpls
  sequence 11
   match
    source-data-prefix-list      BR20
    destination-data-prefix-list BR10
   action
    backup-sla-preferred-color private1
    sla-class       SLA_Real-Time
    no sla-class strict
    sla-class preferred-color mpls
```

Initial state without network issues
```
BR10#show sdwan app-route stats summary | i color|damp|mean         
 local-color              biz-internet
 remote-color             biz-internet
 sla-dampening-index      None
  mean-loss    0.000
  mean-latency 1
  mean-jitter  0
 local-color              mpls
 remote-color             mpls
 sla-dampening-index      None
  mean-loss    1.212
  mean-latency 1
  mean-jitter  0
  mean-loss    1.212
  mean-latency 1
  mean-jitter  0
 local-color              private1
 remote-color             private1
 sla-dampening-index      None
  mean-loss    0.000
  mean-latency 0
  mean-jitter  0
  mean-loss    0.000
  mean-latency 0
  mean-jitter  0
```
Notice that the number of packets per bucket increased dramatically
```
BR10#show sdwan app-route stats remote-color mpls summary  
Generating output, this might take time, please wait ...
app-route statistics 30.2.10.2 30.2.20.2 ipsec 12366 12366
 remote-system-ip         1.1.1.20
 local-color              mpls
 remote-color             mpls
 sla-class-index          0,1
 fallback-sla-class-index None
 enhanced-app-route       Enabled
 sla-dampening-index      None
 app-probe-class-list None
  mean-loss    1.176
  mean-latency 0
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      131136        1501          0        0        100084        20846         0             0             
1      131072        1438          0        0        95287         19605         0             0             
2      131072        1400          0        0        100985        20937         0             0             
3      131072        1781          0        0        85553         18271         0             0             
4      64            0             0        0        72618         15942         0             0             
5      131072        1589          0        0        65198         14226         0             0             
          
```
Traffic using mpls as primary transport
```
BR10#        
Number of possible next hops: 1
Next Hop: IPsec
  Source: 30.2.10.2 12366 Destination: 30.2.20.2 12366 Local Color: mpls Remote Color: mpls Remote System IP: 1.1.1.20
```

I will introduce 3% packet loss on the mpls transport and see how long it takes to switch traffic. Since there is around 1% loss already, 3% should be enough to trigger
a change. 

After 56 seconds, the traffic shifted and any of the remaining compliant transports could be used 
```
BR10#$et4 source-ip 172.16.10.10 dest-ip 172.16.20.10 protocol 6 all
Number of possible next hops: 2
Next Hop: IPsec
  Source: 30.3.10.2 12366 Destination: 30.3.20.2 12366 Local Color: private1 Remote Color: private1 Remote System IP: 1.1.1.20
Next Hop: IPsec
  Source: 30.1.10.2 12386 Destination: 30.1.20.2 12366 Local Color: biz-internet Remote Color: biz-internet Remote System IP: 1.1.1.20

```
The mpls transport has more than 3% loss
```
BR10#show sdwan app-route stats remote-color mpls summary 
Generating output, this might take time, please wait ...
app-route statistics 30.2.10.2 30.2.20.2 ipsec 12366 12366
 remote-system-ip         1.1.1.20
 local-color              mpls
 remote-color             mpls
 sla-class-index          0
 fallback-sla-class-index 1
 enhanced-app-route       Enabled
 sla-dampening-index      None
 app-probe-class-list None
  mean-loss    3.125          <<<<<<<<<<
  mean-latency 0
  mean-jitter  0
```
If I take the loss away we can see the dampening mechanism gets activated. So, if the transport is stable for 20 minutes, it will be used again as preferred path. 
```
BR10#show sdwan app-route stats remote-color mpls summary 
Generating output, this might take time, please wait ...
app-route statistics 30.2.10.2 30.2.20.2 ipsec 12366 12366
 remote-system-ip         1.1.1.20
 local-color              mpls
 remote-color             mpls
 sla-class-index          0
 fallback-sla-class-index 1
 enhanced-app-route       Enabled
 sla-dampening-index      1        <<<<<<<<<<
 app-probe-class-list None
  mean-loss    0.000
  mean-latency 0
  mean-jitter  0
```
#### Scenario 2 - Severe impairment

In this case, I will introduce 10% packet loss to biz-internet transport, making private 1 the only compliant transport. 

After around 45 seconds, loss for biz-internet was 12%

```
BR10#show sdwan app-route stats local-color biz-internet summary         
Generating output, this might take time, please wait ...
app-route statistics 30.1.10.2 30.1.20.2 ipsec 12386 12366
 remote-system-ip         1.1.1.20
 local-color              biz-internet
 remote-color             biz-internet
 sla-class-index          0
 fallback-sla-class-index 1
 enhanced-app-route       Enabled
 sla-dampening-index      None
 app-probe-class-list None
  mean-loss    12.500
  mean-latency 0
  mean-jitter  0
```

Traffic shifted to private1 only
```
BR10#$et4 source-ip 172.16.10.10 dest-ip 172.16.20.10 protocol 6 all
Number of possible next hops: 1
Next Hop: IPsec
  Source: 30.3.10.2 12366 Destination: 30.3.20.2 12366 Local Color: private1 Remote Color: private1 Remote System IP: 1.1.1.20
```
After removing the packet loss, biz-internet has the dampening mechanism activated
```
BR10#show sdwan app-route stats local-color biz-internet summary 
Generating output, this might take time, please wait ...
app-route statistics 30.1.10.2 30.1.20.2 ipsec 12386 12366
 remote-system-ip         1.1.1.20
 local-color              biz-internet
 remote-color             biz-internet
 sla-class-index          0
 fallback-sla-class-index 1
 enhanced-app-route       Enabled
 sla-dampening-index      1         <<<<<<<<<
 app-probe-class-list None
  mean-loss    1.562
  mean-latency 0
  mean-jitter  0
```

Scenario 3 - Multiple App Probe Classes

For this final scenario the configuration is more complex as it involved QoS, App Probe Classes and AAR Policy. 

I configured QoS on the router. I have a shaper to 3 mbps and will have two simultaneous downloads:

```
BR10#show sdwan app-route stats local-color mpls summary 
Generating output, this might take time, please wait ...
app-route statistics 30.2.10.2 30.2.20.2 ipsec 12366 12366
 remote-system-ip         1.1.1.20
 local-color              mpls
 remote-color             mpls
 sla-class-index          0,1,2
 fallback-sla-class-index None
 enhanced-app-route       Enabled
 sla-dampening-index      None
 app-probe-class-list None
  mean-loss    0.000
  mean-latency 1
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      16384         0             0        0        35827         111807        0             0             
1      49152         0             1        0        37788         118236        0             0             
2      49216         0             1        0        36859         115551        0             0             
3      16384         0             1        0        23894         77280         0             0             
4      32768         0             1        1        33179         103759        0             0             
5      32768         0             1        0        21485         71702         0             0             

 app-probe-class-list Real_Time_Probe_Class
  mean-loss    0.000
  mean-latency 0
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      0             0             0        0        -             -             -             -             
1      32768         0             1        0        -             -             -             -             
2      32768         0             0        0        -             -             -             -             
3      0             0             0        0        -             -             -             -             
4      32768         0             1        2        -             -             -             -             
5      0             0             0        0        -             -             -             -             

 app-probe-class-list Transactional-Probe_Class
  mean-loss    0.000
  mean-latency 1
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      16384         0             0        0        -             -             -             -             
1      49152         0             1        0        -             -             -             -             
2      49216         0             1        0        -             -             -             -             
3      16384         0             1        0        -             -             -             -             
4      32768         0             1        1        -             -             -             -             
5      32768         0             1        0        -             -             -             -             

```

### Conclusions 

App probes offer higher benefits if there are private transports that offer quality of service
The total number of samples increase dramatically

