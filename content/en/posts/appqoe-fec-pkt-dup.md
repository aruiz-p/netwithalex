---
author: Alex
category:
  - sdwan
  - appqoe
draft: false
date: "2025-06-03T00:00:00+00:00"
title: "AppQoe Series: Packet Duplication"
description: Learn how Packet Duplication can help you reliably transmit critical traffic over unreliable transports to making sure your application. 
summary: Learn how Packet Duplication can help you reliably transmit critical traffic over unreliable transports to making sure your application. 
url: /appqoe-pkt-dup-en/
tag:
  - Packet Duplication
---
## Introduction

On my [previous post](/appqoe-fec-en/) I talked about FEC, a technique to fight packet loss by sending a parity packet to reconstruct missing data. In this post, we’ll explore another powerful SD-WAN feature designed to tackle lossy transports: **_Packet Duplication_**. While both features aim to improve reliability, they achieve it in very different ways.

Let's dive in:

## What is Packet Duplication

As the name suggests, Packet Duplication involves sending identical packets over multiple transport paths to increase the likelihood of successful delivery. If one path experiences transient loss or is inherently unreliable, the duplicate packet sent over a second, more stable transport can make up for it, keeping the user experience unaffected.

To put it visually: 

![](/wp-content/uploads/2025/05/pkt-dup1.png)

The sender uses **both** the bronze and gold transports to send the duplicate information. The bronze path suffers from packet loss, while the gold path delivers all packets successfully. The receiver discards duplicates and forwards only the earliest received copy to the destination, effectively eliminating loss from the user's perspective — regardless of which path the packet took.

Before duplicating a packet, the wan edge will compare its size against the Path Maximum Transmission Unit (PMUT), if the packet length is smaller than the PMUT, packet gets duplicated. 

**Note** Starting on version 17.15.1a, packets can be duplicated even if length is greater than PMUT, using [VRF and Underlay Fragmentation](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/system-interface/ios-xe-17/systems-interfaces-book-xe-sdwan/configure-interfaces.html#g-fht-vfr-underlay-fragmentation). 

Starting from version 16.12.1b this feature is applicable for
- _IPv4 traffic over IPv4 tunnels_. 

Starting 17.15.1a to support is expanded to:
- IPv4 traffic over IPv6 tunnel 
- IPv6 traffic over IPv4 tunnels
- IPv6 traffic over IPv6 tunnels

## Considerations

Here are a few key points to keep in mind when enabling Packet Duplication:

- For packet duplication to work, there should be **at least** two available transports, this is not a suitable solution for single transport sites.
- The amount of traffic will be **doubled** keep it mind based on transport capacities. 
- The receiver will need to buffer and discard packets, adding processing **overhead**
- Should be enabled for **critical traffic** only, based on previous considerations. 

## Configuration

Using Policy Groups, we can configure Packet Duplication through a data policy that matches intended traffic and applies an action of _Loss Correction_ and select _Packet Duplication_

![](/wp-content/uploads/2025/05/pkt-dup2.png)

This configuration should be applied on both directions, so the complete policy looks like this:

```
 data-policy data_service_Packet-Duplication
  vpn-list vpn_Corporate_Users
   sequence 1
    match
     source-ip      172.16.100.0/24
     destination-ip 172.16.10.0/24
    !
    action accept
     loss-protect pkt-dup
     loss-protection packet-duplication
    !
   !
   sequence 11
    match
     source-ip      172.16.10.0/24
     destination-ip 172.16.100.0/24
    !
    action accept
     loss-protect pkt-dup
     loss-protection packet-duplication
    !
   !
   default-action accept
  !
 !
!
apply-policy
 site-list site_10_100
  data-policy data_service_Packet-Duplication from-service
 !
!
```
**Note** Packet duplication is not supported along with the following data policy actions:
- local tloc
- remote_tloc

You can read the [configuration guide](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/packet-duplication.html) to get more information on restrictions and more.

## Verification

To ensure Packet Duplication is working there are a couple of things we can validate. 

From the device we can run:

```
Lisbon_10-1#show sdwan tunnel statistics pkt-dup 
Generating output, this might take time, please wait ...
tunnel stats gre 21.11.0.2 21.101.0.2 0 0
 pktdup-rx       4844
 pktdup-rx-other 1
 pktdup-rx-this  4844
 pktdup-tx       8710
 pktdup-tx-other 4693
 pktdup-capable  true
tunnel stats gre 31.11.0.2 31.101.0.2 0 0
 pktdup-rx       1
 pktdup-rx-other 4844
 pktdup-rx-this  1
 pktdup-tx       4693
 pktdup-tx-other 8710
 pktdup-capable  true
 ```
- pktdup-rx - original packets received on primary tunnel
- pktdup-rx-other - duplicate packets received on second tunnel
- pktdup-tx - duplicate packets sent on primary tunnel
- pktdup-tx-other - duplicate packets sent from secondary tunnel
- pktdup-capable - Capability exchange with other devices

The same information is available through the Manager's UI in the _Real Time_ information section. 

![](/wp-content/uploads/2025/05/pkt-dup3.png)

Additionally, a counter can be added the the policy to confirm the sequence is getting matched.

## Testing Packet Duplication 

In my setup, I have two active tunnels between the SD-WAN devices: one over MPLS and another over Biz-Internet.

To test Packet Duplication, I’ll introduce artificial packet loss on the MPLS transport and run a basic iperf test between two clients to observe how the feature handles it.

Here’s the test command I’ll use:

>iperf3 -c 172.16.100.11 -u -b 1M -t 20 --dscp ef

This command initiates a 20-second UDP test targeting the remote client, using a bandwidth of 1 Mbps and marking the traffic with DSCP EF.

**Note** I am not focusing on the exact percentage of packet loss introduced on MPLS. The idea isn’t to test how much packet duplication can recover, but rather to observe the mechanism in action. In theory, even with 100% loss on one transport, the traffic should still arrive via the other — as long as Packet Duplication is properly configured.

The sender uses biz-internet as the **primary** transport

```
Lisbon_10-1#show sdwan policy service-path vpn 10 interface gigabitEthernet 2 source-ip 172.16.10.11 dest-ip 172.16.100.11 protocol 17 dscp 46 all            
Number of possible next hops: 1
Next Hop: GRE
  Source: 31.11.0.2 Destination: 31.101.0.2 Local Color: biz-internet Remote Color: biz-internet Remote System IP: 1.1.100.1
```

Mpls has 80% loss and my iperf results were the following:

```
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-20.00  sec  2.38 MBytes  1.00 Mbits/sec  0.000 ms  0/4188 (0%)  sender
[  5]   0.00-20.04  sec  2.38 MBytes   998 Kbits/sec  0.255 ms  0/4188 (0.0%)  receiver
```


Let's check the packet duplication statistics on both sender and receiver sides

Sender 
```
Lisbon_10-1#show sdwan tunnel statistics pkt-dup
Generating output, this might take time, please wait ...
tunnel stats gre 21.11.0.2 21.101.0.2 0 0
 pktdup-rx       14
 pktdup-rx-other 0
 pktdup-rx-this  14
 pktdup-tx       0
 pktdup-tx-other 4206
 pktdup-capable  true
tunnel stats gre 31.11.0.2 31.101.0.2 0 0
 pktdup-rx       0
 pktdup-rx-other 14
 pktdup-rx-this  0
 pktdup-tx       4206
 pktdup-tx-other 0
 pktdup-capable  true
 ```

Center your attention on:

|Role   | Pkts sent primary tunnel| Pkts dup secondary tunnel|
|-------|-------------------------|--------------------------|
|sender | 4206                    | 4206                     |

The exact same amount of packets were transmitted over both mpls and biz-internet transports. 

Receiver 

```
Munich_DC100-1#show sdwan tunnel statistics pkt-dup
Generating output, this might take time, please wait ...
tunnel stats gre 21.101.0.2 21.11.0.2 0 0
 pktdup-rx       0
 pktdup-rx-other 758
 pktdup-rx-this  0
 pktdup-tx       14
 pktdup-tx-other 0
 pktdup-capable  true
tunnel stats gre 31.101.0.2 31.11.0.2 0 0
 pktdup-rx       4206
 pktdup-rx-other 0
 pktdup-rx-this  758
 pktdup-tx       0
 pktdup-tx-other 14
 pktdup-capable  true
```

|Role     |Pkts Received primary | Pkts received secondary|
|---------|----------------------|------------------------|
|Receiver | 4206                 | 758                    |

Since the secondary transport was experiencing heavy packet loss, only 18% of packets were received through it. However, this was not an issue because the primary transport delivered 100% of the packets. 

If the situation were reversed, with the primary path experiencing loss and the secondary path remaining healthy, the roles would simply swap. The primary would deliver 18% and secondary 100%, and once again, the end user wouldn’t experience any packet loss.

## Conclusion

Packet Duplication is a simple yet powerful feature in Cisco SD-WAN that significantly improves the reliability of traffic. By sending identical packets over multiple transport paths, it ensures that even if one path experiences loss, the application performance remains unaffected.

While it does come at the cost of increased bandwidth usage and processing overhead, it can be a game-changer for critical traffic like voice, video, or remote desktop sessions, especially in environments with unreliable circuits.

As with any SD-WAN feature, the key is to use it **strategically**: enable it for the right traffic, monitor the results, and adjust policies based on your network’s behavior. Have you used Packet Duplication in production? Let me know your experience in the comments or connect with me on [LinkedIn](https://www.linkedin.com/in/alexruizs/) to discuss SD-WAN deep dives!
