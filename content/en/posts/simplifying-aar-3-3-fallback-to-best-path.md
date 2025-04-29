---
author: alex
category:
  - aar
  - policies
  - sd-wan
date: "2024-02-11T12:11:06+00:00"
guid: https://netwithalex.blog/?p=509
title: 'Simplifying AAR: 3/3 Fallback to best path'
description: Learn how AAR can help improve the application user experience with Cisco SD-WAN
summary: Learn how AAR can help improve the application user experience with Cisco SD-WAN
url: /simplifying-aar-3-3-fallback-to-best-path/
---
## Introduction

For the final post of this series, let's explore the remaining option to handle traffic when SLA is not met: _Fallback to best path_. It was introduced on 20.5/17.5 and it provides more flexibility and enhanced path selection compared to the other options. Let's understand why it was created.

## Motivation

With the [previous methods](/demystifying-aar-understanding-different-scenarios/), traffic would either:

- Be dropped - Rarely used, specific use cases that apply to a small amount of environments.
- Be load balanced on the available paths - Widely used, however traffic could be using the worst performing path.

Take the following example

![](/wp-content/uploads/2024/02/S3-aar.png)

Tunnel 2 is clearly having the worst performance, however with the _load balance_ method, traffic could still use it based on the hashing algorithm. How do we overcome this situation? You probably guessed it: _**Fallback to best path**_

## Fallback to Best Path

Let's see how the documentation describes it:

> When the data traffic does not meet any of the SLA class requirements, this feature allows you to select the best tunnel path criteria sequence using the Fallback Best Tunnel.
>
> Cisco SD-WAN Manager uses best of worst (BOW) to find a best tunnel when no tunnel meets any of the SLA class requirements.
>
> https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/application-aware-routing.html

### Best of Worst

Let's see how BOW works with the following example:

![](/wp-content/uploads/2024/02/s3-aar5.png)

The SLA latency requirement is set to 8. None of the tunnels satisfy it, but Tunnel 1 is the closest one, making it the best of worse with a latency of 10.

The criteria(s) to choose the BOW is extremely flexible, in this example we used latency, other options could be:

- latency - Only latency
- jitter - Only jitter
- loss - Only loss
- latency/jitter - First latency, if they are equal, then jitter
- latency/loss - First latency, if they are equal, then loss
- jitter/latency - First jitter, if they are equal, then latency
- . . .
- loss/jitter/latency - First loss, then jitter, then latency

### Variance

Let's go a step further, Tunnel 3 is also very close to the _8 ms_ latency, it wouldn't be a bad idea to send traffic also on that tunnel. How do we achieve this? Well, we can implement a _variance_ to accommodate small variations when choosing the best paths.

Continuing with this example, let's take a look at the BOW selection with a `variance of 5 ms`.

```
BOW range = (best latency, best latency + variance)
BOW range = (10, 15)
```

The best latency across tunnels is 10 (Tunnel 1), notice this is **not** the latency configured on the SLA. With this variance, if any other tunnel has a latency between 10 - 15, it will also be chosen to send traffic. In our example, Tunnel 3 satisfies the condition, so now Tunnel 1 and Tunnel 3 will be used as _Fallback Tunnels_.

As you can see, Tunnel 2 is no longer considered. Great!

## Configuration

#### SLA Class

To use this method, the first thing we need to do is modify our sla-class to indicate that it should look for the best performing path when SLA is not met. Our configuration looks like this:

![](/wp-content/uploads/2024/02/s3-aar7.png)

* * *

```
sla-class Custom-SLA
 loss    1
 latency 250
 jitter  100
 fallback-best-tunnel
  criteria loss
  loss-variance 2
```

Notice we selected the _Criteria_ to be `Loss` and a _variance_ of `2`. Variance is an **optional** parameter.

#### AAR Policy

Next, on the AAR policy we specify the action when SLA is not met: _Fallback to Best path_

![](/wp-content/uploads/2024/02/S3-AAR22.png)

* * *

```
sequence 1
 match
  source-ip 172.16.10.0/24
  destination-ip 172.16.20.0/24
 action
  sla-class Custom-SLA
  no sla-class strict
  sla-class preferred-color mpls
  sla-class fallback-to-best-path
```

Let's keep the same dynamic and build a diagram:

![](/wp-content/uploads/2024/02/s3-aar8.png)

## Scenarios

Using the same topology let's explore some situations

![](/wp-content/uploads/2024/02/AAR-Topo2.png)

#### MPLS compliant, Biz-internet/Private1 non-compliant

I will initiate traffic from Branch10 -> Branch 20 and capture it with NWPI. MPLS has perfect KPI metrics (0, 0, 0)

![](/wp-content/uploads/2024/02/s3-aar1-1-1.png)

Notice how _fallback to best path_ is set to `no` and traffic is matching the SLA and preferred color. Let's also see the following verification command from BR10.

```
BR10#show sdwan tunnel sla
<. . .>
tunnel sla-class 1
 sla-name    Custom-SLA
 sla-loss    1
 sla-latency 250
 sla-jitter  100
                                                                                                                                   FALLBACK
                                         REMOTE    T                                           SLA                                 SLA
                             SRC   DST   SYSTEM    LOCAL  T REMOTE      MEAN  MEAN     MEAN    CLASS                               CLASS
PROTO  SRC IP     DST IP     PORT  PORT  IP        COLOR  COLOR         LOSS  LATENCY  JITTER  INDEX  SLA CLASS NAME               INDEX
---------------------------------------------------------------------------------------------------------------------------------------------
gre    21.1.10.2  21.1.20.2  0     0     1.1.0.20  mpls   mpls          0     0        0       0,1    __all_tunnels__, Custom-SLA  None
gre    21.1.10.2  31.1.20.2  0     0     1.1.0.20  mpls   biz-internet  0     0        0       0,1    __all_tunnels__, Custom-SLA  None
gre    21.1.10.2  41.1.20.2  0     0     1.1.0.20  mpls   private1      0     0        0       0,1    __all_tunnels__, Custom-SLA  None
```

Some comments about this output:

1. The fact that we see tunnels listed under Custom-SLA, tells us that there are tunnels meeting the loss, latency and jitter. This is expected as our MPLS is having perfect metrics.
1. See that this sla-class has a numeric identifier of 1 - `tunnel sla-class 1`. You will see reference to this number shortly.
1. _Fallback SLA class index_ is set to `none`, this mean that these tunnels are not being used as fallback tunnels, this will become clear in a second.

#### MPLS/Biz-inernet/Private1 non-compliant but meeting variance

Now that we have no transports meeting the SLA, let's check how NWPI will show it:

![](/wp-content/uploads/2024/02/s3-aar10.png)

Notice that now NWPI is indicating that _Fallback to Best Path_ is in use.

Private1 was chosen to send this particular flow, but are there any other tunnels that could be used? Let's check BR10 again.

```
BR10# show sdwan tunnel sla
tunnel sla-class 0
 sla-name    __all_tunnels__
 sla-loss    0
 sla-latency 0
 sla-jitter  0
                                                                                                                              FALLBACK
                                         REMOTE                                                       SLA                     SLA
                             SRC   DST   SYSTEM    T LOCAL       T REMOTE      MEAN  MEAN     MEAN    CLASS                   CLASS
PROTO  SRC IP     DST IP     PORT  PORT  IP        COLOR         COLOR         LOSS  LATENCY  JITTER  INDEX  SLA CLASS NAME   INDEX
----------------------------------------------------------------------------------------------------------------------------------------
gre    21.1.10.2  21.1.20.2  0     0     1.1.0.20  mpls          mpls          18    0        0       0      __all_tunnels__  None
gre    21.1.10.2  31.1.20.2  0     0     1.1.0.20  mpls          biz-internet  9     0        0       0      __all_tunnels__  None
gre    21.1.10.2  41.1.20.2  0     0     1.1.0.20  mpls          private1      11    0        0       0      __all_tunnels__  None
gre    31.1.10.2  21.1.20.2  0     0     1.1.0.20  biz-internet  mpls          4     0        0       0      __all_tunnels__  1
gre    31.1.10.2  31.1.20.2  0     0     1.1.0.20  biz-internet  biz-internet  2     0        0       0      __all_tunnels__  1
gre    31.1.10.2  41.1.20.2  0     0     1.1.0.20  biz-internet  private1      4     0        0       0      __all_tunnels__  1
gre    41.1.10.2  21.1.20.2  0     0     1.1.0.20  private1      mpls          3     0        0       0      __all_tunnels__  1
gre    41.1.10.2  31.1.20.2  0     0     1.1.0.20  private1      biz-internet  6     0        0       0      __all_tunnels__  None
gre    41.1.10.2  41.1.20.2  0     0     1.1.0.20  private1      private1      3     0        0       0      __all_tunnels__  1

tunnel sla-class 1
 sla-name    Custom-SLA
 sla-loss    1
 sla-latency 250
 sla-jitter  100

BR10#
```

Comments about this output:

1. There are **no** tunnels meeting our _Custom-SLA_
1. Even if MPLS is the preferred color, it is **not** considered because it doesn't meet the loss variance range.
1. There are 5 tunnels that satisfy the variance. Notice how some tunnels have _Fallback SLA class index_ set to `1`, meaning that they are serving as fallback for sla-class 1 (Custom-SLA). In this case the **BOW** is _biz-internet - biz-internet_ tunnel with a mean loss of `2`. The _variance_ is set to 2, so BOW range is 2-4. Tunnels satisfying the BOW range, will be used to forward traffic as well.

Depending on the load-balance hash, different tunnels will be chosen

```
BR10#show sdwan policy service-path vpn 10 interface GigabitEthernet 3 source-ip 172.16.10.2 dest-ip 172.16.20.2 protocol 6 dest-port 22
Next Hop: GRE
  Source: 31.1.10.2 Destination: 21.1.20.2 Local Color: biz-internet Remote Color: mpls Remote System IP: 1.1.0.20

BR10#show sdwan policy service-path vpn 10 interface GigabitEthernet 3 source-ip 172.16.10.56 dest-ip 172.16.20.2 protocol 6 dest-port 24
Next Hop: GRE
  Source: 31.1.10.2 Destination: 41.1.20.2 Local Color: biz-internet Remote Color: private1 Remote System IP: 1.1.0.20
```

We can see two different tunnels `biz-internet - mpls` and `biz-internet - private`

#### Latency out of compliance

So far we have been playing only with loss, because the fallback criteria was set to loss. Let's see what happens when a different criteria goes out of compliance. I will set the latency of the SLA to 15 ms. .

```
BR10#show sdwan policy from-vsmart
from-vsmart sla-class Custom-SLA
 loss    1
 latency 15
 jitter  100
 fallback-best-tunnel
  criteria      loss
  loss-variance 2

```

After introducing some latency, we see something interesting:

```
BR10#show sdwan tunnel sla
tunnel sla-class 0
 sla-name    __all_tunnels__
 sla-loss    0
 sla-latency 0
 sla-jitter  0
                                                                                                                              FALLBACK
                                         REMOTE                                                       SLA                     SLA
                             SRC   DST   SYSTEM    T LOCAL       T REMOTE      MEAN  MEAN     MEAN    CLASS                   CLASS
PROTO  SRC IP     DST IP     PORT  PORT  IP        COLOR         COLOR         LOSS  LATENCY  JITTER  INDEX  SLA CLASS NAME   INDEX
----------------------------------------------------------------------------------------------------------------------------------------
gre    21.1.10.2  21.1.20.2  0     0     1.1.0.20  mpls          mpls          0     20       1       0      __all_tunnels__  1
gre    21.1.10.2  31.1.20.2  0     0     1.1.0.20  mpls          biz-internet  0     20       1       0      __all_tunnels__  1
gre    21.1.10.2  41.1.20.2  0     0     1.1.0.20  mpls          private1      0     20       0       0      __all_tunnels__  1
gre    31.1.10.2  21.1.20.2  0     0     1.1.0.20  biz-internet  mpls          0     21       2       0      __all_tunnels__  1
gre    31.1.10.2  31.1.20.2  0     0     1.1.0.20  biz-internet  biz-internet  0     21       2       0      __all_tunnels__  1
gre    31.1.10.2  41.1.20.2  0     0     1.1.0.20  biz-internet  private1      0     21       2       0      __all_tunnels__  1
gre    41.1.10.2  21.1.20.2  0     0     1.1.0.20  private1      mpls          0     16       1       0      __all_tunnels__  1
gre    41.1.10.2  31.1.20.2  0     0     1.1.0.20  private1      biz-internet  0     16       0       0      __all_tunnels__  1
gre    41.1.10.2  41.1.20.2  0     0     1.1.0.20  private1      private1      0     16       0       0      __all_tunnels__  1

tunnel sla-class 1
 sla-name    Custom-SLA
 sla-loss    1
 sla-latency 15
 sla-jitter  100

BR10#
```

All of the tunnels are used as fallback tunnels because all of them have 0% loss! Is this the ideal situation? This is arguable, maybe for some types of traffic it's fine, but for others you probably want to have a second or third criteria to pick the best fallback tunnels.

#### Multiple BOW criteria

For the last test, let's see what happens when we select multiple criteria to select the BOW. I will add latency to the SLA criteria.

```
BR10#show sdwan policy from-vsmart
from-vsmart sla-class Custom-SLA
 loss    1
 latency 15
 jitter  100
 fallback-best-tunnel
  criteria      loss latency
  loss-variance 2
```

What we expect is that if any of the KPIs go out of compliance, the BOW will be decided based on:

1. Lowest mean loss. If there is a tie, then
1. Lowest latency

Let's verify

```
BR10#show sdwan tunnel sla
tunnel sla-class 0
 sla-name    __all_tunnels__
 sla-loss    0
 sla-latency 0
 sla-jitter  0
                                                                                                                              FALLBACK
                                         REMOTE                                                       SLA                     SLA
                             SRC   DST   SYSTEM    T LOCAL       T REMOTE      MEAN  MEAN     MEAN    CLASS                   CLASS
PROTO  SRC IP     DST IP     PORT  PORT  IP        COLOR         COLOR         LOSS  LATENCY  JITTER  INDEX  SLA CLASS NAME   INDEX
----------------------------------------------------------------------------------------------------------------------------------------
gre    21.1.10.2  21.1.20.2  0     0     1.1.0.20  mpls          mpls          0     20       1       0      __all_tunnels__  None
gre    21.1.10.2  31.1.20.2  0     0     1.1.0.20  mpls          biz-internet  0     20       1       0      __all_tunnels__  None
gre    21.1.10.2  41.1.20.2  0     0     1.1.0.20  mpls          private1      0     20       1       0      __all_tunnels__  None
gre    31.1.10.2  21.1.20.2  0     0     1.1.0.20  biz-internet  mpls          0     20       1       0      __all_tunnels__  None
gre    31.1.10.2  31.1.20.2  0     0     1.1.0.20  biz-internet  biz-internet  0     20       1       0      __all_tunnels__  None
gre    31.1.10.2  41.1.20.2  0     0     1.1.0.20  biz-internet  private1      0     21       1       0      __all_tunnels__  None
gre    41.1.10.2  21.1.20.2  0     0     1.1.0.20  private1      mpls          0     16       0       0      __all_tunnels__  1
gre    41.1.10.2  31.1.20.2  0     0     1.1.0.20  private1      biz-internet  0     16       0       0      __all_tunnels__  1
gre    41.1.10.2  41.1.20.2  0     0     1.1.0.20  private1      private1      0     16       0       0      __all_tunnels__  1

tunnel sla-class 1
 sla-name    Custom-SLA
 sla-loss    1
 sla-latency 15
 sla-jitter  100

BR10#
```

It's clear that the loss wasn't used to come up with the BOW, otherwise we would see all the tunnels acting as fallback for SLA Class index 1. Instead, tunnels with lowest latency were selected. Notice that I could have added a latency variance to include other tunnels with similar numbers.

## Conclusion

Through the last three posts, we have witnessed AAR being a critical SD-WAN functionality to protect the SLA of our applications. I hope that after explaining and verifying different scenarios, you now have a better understanding and feel more confident to try AAR within your SD-WAN infrastructure.. The [configuration guide](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/application-aware-routing.html#config-variance-best-tunnel-path) is very complete, so get familiar with it and use it when you need it.

Let me know your thoughts in the comments.

See you soon!
