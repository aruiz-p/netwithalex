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
### Introduction

In my previous [series of posts](/demystifying-aar-1-3-the-foundations), I talked in depth about Application Aware Routing (AAR), a technology used to direct traffic through best performing paths. AAR is a core functionality of SD-WAN and has been around for years. Naturally, new ideas to improve it came around and gave birth to a revamped implementation which Cisco called **_Enhanced_** Application Aware Routing (EAAR). Let´s take a look.

### AAR Limitations

Before diving into EAAR, let's understand why it was created.

The current AAR implementation measures the path quality using BFD, sending packets at a defined interval, 1 second by default. Loss, latency and jitter are derived from those packets and these values are placed into rotating buckets to calculate an average tunnel health metric. This process would typically take between 10 and 60 minutes and with some configuration tweaks we could achieve times of 2 - 10 mins.

For those networks needing faster detection, some problems arise:
- Lowering the hello interval to less than 1s affects the device's tunnel scale
- Lowering bfd multiplier and poll intervals could lead to false positives, switching traffic even with a transient network condition.
- Switching traffic back and forth, there is no mechanism to determine if a transport is stable again after a network degradation event. 
- 

### Enhanced AAR

In a nutshell these are the advantages you will get by deploying EAAR:
- Improves performance measurements using **inline data**. In other words, the data plane packets are used to get loss, latency and jitter.
- Dampening implemented for stability purposes, ensuring transports are stable before forwarding traffic through them. 
- Steer traffic in seconds rather than minutes
- Per queue measurement of network performance


#### Taking a deeper look 

Loss measurement is done per queue
Latency is the RTT
Jitter is unidirectional, which means it is measured at the receiver and reported to the sender

There are three pre-defined modes to run EEAR
| Mode | Poll Interval | Multiplier | Dampening | 


In **Continuous Integration (CI)**, a crucial idea is the frequent uploading of code into the code base, often multiple times a day. This involves the constant merging of developer work with the code base, and the early identification of issues through the use of testing.

![](/wp-content/uploads/2024/01/Screenshot-2024-01-17-at-15.23.10.png)CI Diagram


### CI tools

These are some of the most common CI tools out there. In the next post we are going to explore them with more detail.

CI ToolsURL**_Travis CI_**[https://travis-ci.org](https://travis-ci.org)**_Jenkins_**[https://jenkins.io](https://jenkins.io)**_Drone_**[https://drone.io](https://drone.io)**_GitLab CI_**[https://gitlab.com](https://gitlab.com)

Click [here](/ci-tools/) to learn about them!There are three pre-defined modes to run EEAR
| Mode | Poll Interval | Multiplier | Dampening | 

