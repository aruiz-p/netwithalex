---
author: alex
category:
  - aar
  - policies
  - sd-wan
date: "2024-02-01T20:27:37+00:00"
guid: https://netwithalex.blog/?p=349
title: 'Simplifying AAR: 2/3 Understanding different scenarios'
url: /demystifying-aar-understanding-different-scenarios/

---
### Introduction

Welcome back to the second installment of my series on Application Aware Routing (AAR). In my [previous post](/demystifying-aar-1-3-the-foundations/), we discussed the essential concepts of Bidirectional Forwarding Detection (BFD) and Service Level Agreements (SLAs), laying the foundation for understanding how AAR optimizes network performance based on application requirements. We also briefly touched on tha AAR configuration with a simple example.

Now, we continue by digging deeper on different AAR configurations, more specifically we will concentrate on _Strict/Drop_ and _Backup SLA Preferred Color_ behaviors.

### Initial Topology and Configuration

Let's start with a simple topology

![](/wp-content/uploads/2024/01/AAR-topology.png)

I am not limiting the tunnels between colors so we have a total of 4 tunnels on each wan edge.

- mpls - mpls
- mpls - biz-inet
- biz-inet- mpls
- biz-inet - biz-inet

This is the BFD hello interval and app-route configuration for all the devices - this is the most relevant to AAR. If you are not familiar with it, I recommend you check [my last post](/demystifying-aar-1-3-the-foundations/) before continuing.

```
hello-interval 1000
bfd app-route multiplier 3
bfd app-route poll-interval 120000
```

### Scenario 1: SLA not Met - Strict/Drop

Our AAR policy is testing against Business-Critical SLA. I will be playing with **packet loss** to demonstrate how traffic shifts.

```
BR10#show sdwan policy from-vsmart
from-vsmart sla-class Business-Critical
 loss    1
 latency 250
 jitter  100
from-vsmart app-route-policy _VPN10_AAR
 vpn-list VPN10
  sequence 1
   match
    source-ip      172.16.10.0/24
    destination-ip 172.16.20.0/24
   action
    sla-class       Business-Critical
    sla-class strict
    sla-class preferred-color mpls
  sequence 11
   match
    source-ip      172.16.20.0/24
    destination-ip 172.16.10.0/24
   action
    sla-class       Business-Critical
    sla-class strict
    sla-class preferred-color mpls
from-vsmart lists vpn-list VPN10
 vpn 10
```

Let's read what the documentation says about our config:

`sla-class preferred-color mpls`

> sla-class _sla-class-name_ preferred-color _color_—To set a specific tunnel to use when data traffic matches an SLA class, include the preferred-color option, specifying the color of the preferred tunnel. If more than one tunnel matches the SLA, traffic is sent to the preferred tunnel. If a tunnel of the preferred color is not available, traffic is sent through any tunnel that matches the SLA class. If no tunnel matches the SLA, data traffic is sent through any available tunnel
>
> https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/application-aware-routing.html

Here's a diagram to better visualize it:

![](/wp-content/uploads/2024/01/ARR100.png)

Note than even if both biz-internet **and** MPLS match the SLA, only MPLS will be used.

Yes, Biz-internet will be used even if it's not specified as a preferred color - from my experience, this is a frequent source of confusion as people expect that if mpls is not matching SLA, the _SLA not met_ action will be executed without considering the rest of the colors.

`sla-class strict`

> Click **Strict/Drop** to perform strict matching of the SLA class. If no data plane tunnel is available that satisfies the SLA criteria, traffic is dropped.
>
> https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/application-aware-routing.html

Now the diagram looks like this:

![](/wp-content/uploads/2024/01/AAR101.png)

#### Test 1 - MPLS/Biz-internet compliant

I will initiate traffic from Branch10 -> Branch 20 and capture it with NWPI. Both MPLS and Biz-Internet are having perfect KPI metrics (0 loss, 0 latency, 0 jitter)

```
BR10-PC1#ssh -l admin 172.16.20.2
Password:
```

![](/wp-content/uploads/2024/02/aar-1.png)

We can see the _Actual Color_ is `mpls` and _Tunnel Match Reason_ is `Matched sla and pref encap color`. Everything working according to our policy definition.

#### Test 2 - Biz-Internet compliant, MPLS non-compliant

Let's introduce 3% of packet loss on the mpls link and check the results.

![](/wp-content/uploads/2024/02/aar-2.png)![](/wp-content/uploads/2024/02/AAR-4-e1706772628492.png)

Now, our mpls tunnel is above the 1% loss threshold, so it's no longer eligible. We confirm the Biz-Internet tunnel is used because it matches the SLA - _Tunnel match reason_ is ` matched sla and color any`

#### Test 3 - MPLS/Biz-Internet non-compliant

Our last test for this scenario will be to mess the SLAs for both transports. Again, will introduce 3% packet loss on **both** mpls and biz-internet.

![](/wp-content/uploads/2024/02/aar3-e1706772589421.png)

As expected, now the traffic is getting dropped on BR10 as there are no transports matching the SLA class. Note the _DROP\_REPORT_ indicates `SdwanDataPolicyDrop`

### Scenario 2: Backup SLA Preferred Color

Let's explore a new scenario, we will add one transport for this one.

![](/wp-content/uploads/2024/02/AAR-Topo2.png)

Our policy configuration will have slight changes. One thing to remark is that when using _Backup SLA preferred color_ option, the only available action when SLA is not met will be _Load Balance_.

```
BR10#show sdwan policy from-vsmart
from-vsmart sla-class Business-Critical
 loss    1
 latency 250
 jitter  100
from-vsmart app-route-policy _VPN10_AAR
 vpn-list VPN10
  sequence 1
   match
    source-ip      172.16.10.0/24
    destination-ip 172.16.20.0/24
   action
    backup-sla-preferred-color private1
    sla-class       Business-Critical
    no sla-class strict
    sla-class preferred-color mpls
  sequence 11
   match
    source-ip      172.16.20.0/24
    destination-ip 172.16.10.0/24
   action
    backup-sla-preferred-color private1
    sla-class       Business-Critical
    no sla-class strict
    sla-class preferred-color mpls
from-vsmart lists vpn-list VPN10
 vpn 10
```

Again, let's see what the documentation says:

`backup-sla-preferred-color private1`

> **When no tunnel matches the SLA**, you can choose how to handle the data traffic:
>
> backup-sla-preferred-color _colors_—Direct the data traffic to a specific tunnel. Data traffic is sent out the configured tunnel if that tunnel interface is available; if that tunnel is unavailable, traffic is sent out another available tunnel. You can specify one or more colors.
>
> https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/application-aware-routing.html

To put it visually

![](/wp-content/uploads/2024/02/AAR200.png)

#### Test 1 - MPLS/Private1 compliant, Biz-Internet non-compliant

We will start with the assumption that biz-internet is not compliant with the SLA, so we have two transports available: MPLS and Private1. Let's initiate our traffic.

```
BR10-PC1#ssh -l admin 172.16.20.2
Password:
```

![](/wp-content/uploads/2024/02/S2-AAR1-e1706776034732.png)

Notice _SLA strict_ is set to `No` and mpls is compliant and in use.

#### Test 2 - Private1 compliant, MPLS/Biz-Internet non-compliant

For the second test we are Introducing loss on the mpls transport. With this, now the only compliant transport is Private1, which also happens to be the Backup SLA preferred color. Let's see how it looks.

![](/wp-content/uploads/2024/02/S2-AAR101.png)

From the _Tunnel Match Reason_ we can clearly see that the SLA is met through a color that is not the preferred one (private1). What do you think will happen if private1 becomes non-compliant?

#### Test 3 - MPLS/Private1/Biz-Internet non-compliant

Private1 is the only transport complying with the SLA, I will take care of that by introducing packet loss.

![](/wp-content/uploads/2024/02/S2-102.png)

Again (!) private1 is used, BUT now traffic is matching the Default SLA, in other words, there are no tunnels matching the SLA so the tunnel marked as backup preferred, will be used.

#### Bonus

**Left - Private1 unavailable**  
The result after shutting down private1 interface, still without any tunnel matching the SLA. We can see a loose match is done on the tunnels (any available tunnel could be picked)

**Right - Biz-internet compliant, MPLS/Private1 non-compliant**  
The result after biz-internet is brought back to zero loss values with private1 not matching the SLA. Even if it's not the preferred color, it still matches the SLA so it is selected.

![](/wp-content/uploads/2024/02/S2-103.png)![](/wp-content/uploads/2024/02/S2-104.png)

As a final note, keep in mind AAR will always try to use tunnels that match the specified SLA, **don't get confused because the color names are not explicitly mentioned on the configuration**.

In the [next post](/simplifying-aar-3-3-fallback-to-best-path), we will explore the remaining option _Fallback to best path_. Meet you there!
