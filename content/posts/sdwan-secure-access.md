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

Secure Access is a robust solution that addresses these challenges. It offers top notch security by integrating advanced technologies and access controls. This means that users can get a _**secure and direct**_ connectivity from SD-WAN sites to the internet or private/public clouds.

Let's see how it works 

## SD-WAN Meets Secure Access

Starting with SD-WAN version 20.13/17.13, an integration with Secure Access is now available out of the box.

With this integration, automatic tunnels are established to the primary and secondary Secure Access Data Centers closest to your router’s location, ensuring optimal performance. These tunnels route traffic while enforcing your organization's security policies, providing a simple and powerful way to improve both security and connectivity.

At the core of Secure Access are [Network Tunnel Groups (NTGs)](https://docs.sse.cisco.com/sse-user-guide/docs/manage-network-tunnel-groups), which manage IPSec connections. Each NTG includes a primary and a secondary Secure Access Data Center. While configuring tunnels to both data centers is not mandatory, it is highly recommended to ensure high availability in case one becomes unavailable.

![](/wp-content/uploads/2025/02/sse-topo.png)

Each tunnel supports up to 1 Gbps of throughput in either direction. It is possible to configure up to 16 tunnels, 8 active and 8 backup, allowing for load balancing across active tunnels to increase the available bandwidth. 

To automatically establish the tunnels, the SD-WAN router should have internet connectivity and DNS lookup enabled. This is needed so the device can determine it's own public ip address, communicate it to vmanage and 


## Lessons learned 

- Using inline data, the number of samples increases dramatically compared to BFD sample size. 📈
- EAAR can steer traffic in seconds, rather than minutes. ⏩
- EAAR delivers the greatest benefits on transports with QoS, such as MPLS. 🚀
- Even on transports without QoS, inline data measurements increases sample size and accuracy. ⏳
- The dampening timer is useful to ensure transports are stable before marking them as valid. ✅
- Interoperability between devices running EAAR and devices running AAR is possible 🔄

I hope you have learned something useful! See you on the next one 👋

