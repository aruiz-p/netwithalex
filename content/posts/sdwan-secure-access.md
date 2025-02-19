---
author: alex
category:
  - sdwan
  - security
  - secure access
date: "2025-02-05T14:20:31+00:00"
title: Securing the Edge with Cisco SD-WAN and Secure Access
summary: Discover how Cisco SD-WAN and Cisco Secure Access work together to enhance network performance and security in a cloud-first world.
url: /sdwan-sse-integration/
tag:
  - sse
  - sdwan
  - security
---
## Introduction

Security has always been a top of mind for organizations, but protecting every angle of the network remains a challenge. At the same time, ensuring an optimal application and user experience is equally important. Organizations have often had to choose between security-focused and performance-driven solutions, leading to increased management and operational complexity. 

Secure Access is a robust solution that addresses these challenges. It offers top notch security by integrating advanced technologies and access controls. This means that users can get a _**secure and direct**_ internet connectivity from SD-WAN sites.

Let's see how it works 

## SD-WAN Meets Secure Access

Starting with SD-WAN version 20.13/17.13, an integration with Secure Access is now available out of the box.

With this integration, automatic tunnels are established to the primary and secondary Secure Access Data Centers closest to your router’s location, ensuring optimal performance. These tunnels route internet traffic while enforcing your organization's security policies, providing a simple and powerful way to improve both security and connectivity.

At the center of Secure Access there are Network Tunnels Groups (NTG) to manage IPSec connections. These NTGs will contain a primary and secondary Secure Access Data Center. Although it's not mandatory to have a tunnel to primary and secondary DC, it is highly recommended to achieve high availability in case one of them is not available. 

![](/wp-content/uploads/2025/02/sse-topo.png)

You can configure up to 16 tunnels, 8 active and 8 backup and load balance across active tunnels is possible. 


## Lessons learned 

- Using inline data, the number of samples increases dramatically compared to BFD sample size. 📈
- EAAR can steer traffic in seconds, rather than minutes. ⏩
- EAAR delivers the greatest benefits on transports with QoS, such as MPLS. 🚀
- Even on transports without QoS, inline data measurements increases sample size and accuracy. ⏳
- The dampening timer is useful to ensure transports are stable before marking them as valid. ✅
- Interoperability between devices running EAAR and devices running AAR is possible 🔄

I hope you have learned something useful! See you on the next one 👋

