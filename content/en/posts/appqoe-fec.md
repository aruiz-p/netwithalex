---
author: Alex
category:
  - sdwan
  - appqoe
draft: false
date: "2025-05-19T14:20:31+00:00"
title: "AppQoe Series: Forward Error Correction (FEC)"
description: Learn how Forward Error Correction (FEC) works in Cisco SD-WAN to improve application performance over lossy links. Explore use cases, configuration insights, and testing results.
summary: Learn how Forward Error Correction (FEC) works in Cisco SD-WAN to improve application performance over lossy links. Explore use cases, configuration insights, and testing results.
url: /appqoe-fec/
tag:
  - Loss Correction
---
## Introduction

Delivering consistent application performance over unreliable or congested links is a constant challenge for most networks. Even with great features like [Enhanced Application Aware Routing](/enhanced-aar/) or [TCP Optimization](/appqoe-tcp-opt/), there are link conditions that go beyond what failover, load balancing or optimization can solve. 

By adding a recovery mechanism at the packet level, FEC allows Cisco SD-WAN to mask packet loss and maintain application performance without relying on retransmissions.

In this post, we will explore how effective FEC can be in SD-WAN. By simulating lossy conditions, and measuring recovery rates.

If you evaluating FEC for your deployment or just curious about how it works, this post will walk you through both the theory and practice. 

Let's go! 

## What is Forward Error Correction (FEC)?

Forward Error Correction (FEC) is a technique that improves data transmission reliability by adding redundant information to packets before they’re sent across the network. Instead of waiting for retransmissions when packets are lost, the receiver uses this redundancy to reconstruct missing data on the fly.

In the Cisco SD-WAN implementation, FEC creates blocks of 4 data packets + 1 _parity packet_. If _**one**_ of those 4 packets is lost along the way, the receiver can fill in the gaps using the parity packet by performing an XOR operation.

Let's see it in a diagram

![](/wp-content/uploads/2025/05/fec1.png)

The sender transmits information to the receiver, but packet 3 is lost in transit. The receiver can use the parity packet to reconstruct packet 3 and avoid retransmissions and delays that would impact application experience.

Notice that if more than 1 packet is lost, including the parity packet, the reconstruction is not possible. The block size is always 4 and it cannot be changed. Blocks can contain packets from multiple flows.

> **Note** The fact that FEC adds 1 parity packet for every block of 4 packets increases BW consumption. 

There are 2 modes of operation 
- **Always**: FEC will be applied to all traffic matching the policy sequence regardless of packet loss levels.
- **Adaptive**: Set a packet loss threshold to start using FEC. For example, with 2% or more packet loss  start applying FEC to the traffic. Loss percent is computed with [BFD packets](/demystifying-aar-1-3-the-foundations/#bfd). 

FEC is particularly useful in real-time applications like voice, video, or interactive sessions, where waiting for retransmissions would cause severe delays.

Importantly, FEC operates between SD-WAN edge devices, making it completely transparent to the applications, there's no need to modify client or server behavior. However, it only works when using IPSec encapsulation; it's **not supported over GRE tunnels**.

One critical implementation detail is packet size: if packets are too large and end up being fragmented, FEC's ability to reconstruct them is significantly reduced. To get the most out of FEC, make sure the payload size stays below the path MTU to avoid fragmentation.

## Configuration

Using Policy Groups, we can configure FEC through a data policy that matches interesting traffic and applies an action of _Loss Correction_

![](/wp-content/uploads/2025/05/fec2.png)

In my case, I matched all the traffic between 172.16.10.0/24 and 172.16.100.0/24. Notice we have the two modes of operations available: _Always and Adaptive_

If FEC Adaptive is selected, the available thresholds are between 1%-5% loss. 

Here is the full configuration of my policy:

```
vsmart_1# show running-config policy 
policy
 data-policy data_all_FEC
  vpn-list vpn_Corporate_Users
   sequence 1
    match
     source-ip      172.16.100.0/24
     destination-ip 172.16.10.0/24
    !
    action accept
     loss-protect fec-always
     loss-protection forward-error-correction always
    !
   !
   sequence 11
    match
     source-ip      172.16.10.0/24
     destination-ip 172.16.100.0/24
    !
    action accept
     loss-protect fec-always
     loss-protection forward-error-correction always
    !
   !
   default-action accept
  !
 !
 lists
  vpn-list vpn_Corporate_Users
   vpn 10
  !
  site-list site_10_100
   site-id 10
   site-id 100
  !
 !
 apply-policy
 site-list site_10_100
  data-policy data_all_FEC from-service
 !
!
!
```

## Verifying FEC

There aren't a lot of commands, we can confirm FEC is operational with the following command:

```
Munich_DC100-1#show sdwan tunnel statistics fec 
tunnel stats ipsec 21.101.0.2 21.11.0.2 12346 12346
 fec-rx-data-pkts     16243
 fec-rx-parity-pkts   4075
 fec-tx-data-pkts     7
 fec-tx-parity-pkts   1
 fec-reconstruct-pkts 935
 fec-capable          true
 fec-dynamic          false
```
The _fec-reconstruct-pkts_ indicate that 935 packets have been recovered. 

Also, notice that we can easily see how many parity packets are sent and received which are roughly 1/4 of total sent/received data packets. 

The same information is also available through the _real-time_ information on the Manager's UI 

![](/wp-content/uploads/2025/05/fec3.png)

## Testing FEC

Let's run some tests to see FEC in action and the amount of packet loss that can be recovered. I will show different results to understand where FEC delivers better results. 

**Note** there is loss outside of the SD-WAN routers I cannot control so in order to have more precise results I had to found the rate at which I got 0% packet loss **most of the time** with my iperf3 results and start introducing controlled loss from there. 

> iperf -c 172.16.100.11 -u -b 450k -t 30 -l 361 --dscp ef 

A bandwidth of 450k is **around** 5 VoIP calls and using a payload of 361 bytes. 

In this case, I am running unidirectional tests, but keep in mind FEC works in both directions.

|Loss% introduced|Total Sent Packets|Total Received Packets|Recovered Packets|Effective Loss %|
|-----------------|------------------|----------------------|-----------------|---------------|
|1                |4693              |4639                  |54               |0              |
|2                |4694              |4588                  |96               |0,24           |
|3                |4693              |4558                  |111              |0,58           |
|4                |4694              |4524                  |147              |0,51           |
|5                |4693              |4448                  |195              |0,68           |
|6                |4693              |4401                  |231              |1,3            |
|7                |4693              |4374                  |238              |1,8            |
|8                |4693              |4331                  |283              |1,8            |
|9                |4693              |4292                  |297              |2,2            |
|10               |4693              |4215                  |304              |3,8            |
|12               |4693              |4122                  |348              |3,8            |
|15               |4695              |3941                  |382              |8              |
|18               |4696              |3815                  |356              |11             |
|20               |4696              |3731                  |368              |13             |

Let's see some interesting visuals:

![](/wp-content/uploads/2025/05/fec4.png)
![](/wp-content/uploads/2025/05/fec5.png)
![](/wp-content/uploads/2025/05/fec6.png)

As packet loss increases, the number of recovered packets also grows - up to a certain point. This is expected: FEC adds redundancy, and the more packets are lost, the more recovery is needed. However, there's a natural limit to this capability. If two or more packets within the same FEC block are lost, including the parity packet, recovery becomes impossible, and _effective loss_ starts to climb.

It's also important to note that FEC is a **resource intensive** feature, hence it should be activated for critical traffic and ideally using a packet loss threshold rather than always. 

While this lab setup isn’t a perfect replica of real-world conditions, the results are still insightful. FEC was able to recover nearly all lost packets with up to 5% introduced loss and continued to recover around 70% of packets at ~9% loss. Beyond that, recovery efficiency starts to drop. That said, it’s uncommon to see consistent 10%+ loss in production WAN transports and even more so in both directions.

Finally, although these tests were unidirectional, it’s worth noting that FEC can be applied independently on each direction. This means a well-tuned deployment could tolerate about 5% packet loss per direction while maintaining good performance.

## Conclusion

Forward Error Correction (FEC) is a proactive technique that adds redundancy before packet transmission, allowing the receiver to recover from certain losses without needing retransmissions. This makes it especially valuable for real-time applications like voice and video, where waiting for retries would introduce harmful delays.

Remember that FEC isn't free as it introduces overhead. The receiving SD-WAN edge must use additional processing power to reconstruct lost packets, use it for critical traffic and ideally after a specified packet loss threshold. 

FEC is not a replacement for fixing poor network links. Instead, it acts as a smart mitigation layer that helps smooth over transient or moderate packet loss, keeping user experience consistent even when the network isn’t perfect.

Overall, when deployed correctly, FEC can be a powerful tool in your SD-WAN toolbox — helping ensure consistent application performance over imperfect networks.

💭 What’s your take on Forward Error Correction in SD-WAN? Have you used it before? Do you have questions about how it works or when to enable it?

Drop your thoughts or doubts in the comments! I’d love to hear how others are approaching FEC in real world deployments. Let’s learn from each other!