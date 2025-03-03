---
author: alex
category:
  - sd-routing
date: "2024-02-24T12:06:47+00:00"
guid: https://netwithalex.blog/?p=541
title: 'A New Chapter: SD-Routing''s Revolution in Network Management'
summary: Explore how SD-Routing can simply and effectively help you manage and monitor your WAN network. 
url: /a-new-chapter-sd-routings-revolution-in-network-management/

---
### Introduction

Software Defined Networks (SDNs) came with the promise of simplifying network management, enabling network teams to automate and adopt a programmatic approach. Cisco created solutions like - Catalyst Center, ACI, Meraki and _Viptela SD-WAN_. The latter introduced new controllers that changed the rules that dictate how the WAN network operates.

Organization that were not ready to take the full SD-WAN leap, while still desiring the benefits of SDN principles had limited options, such as integrating with third-party systems or relying on Catalyst Center. While these solutions addressed some challenges, a significant gap remained unfilled until... SD-Routing!

### What is SD-Routing?

Simply put, SD-Routing is the middle ground between SDNs and SD-WAN. With SD-Routing, organizations could gradually adopt software-defined networking in their existing infrastructure. The idea behind it is to manage your non-sdwan (a.k.a "traditional") network from a Single Pane of Glass called _SD-WAN Manager_ (formerly vManage). It is possible to manage SD-WAN devices at the same time, of course!

As opposed to SD-WAN, there is no need to connect to the _SD-WAN controller_ (formerly vSmart), thus the existing routing protocols remain in place and you get an additional security layer with the _SD-WAN Validator_ (formely vBond) to allow the right devices to connect.

### Benefits

Some of the benefits of SD-Routing include

- **Onboarding** \- New devices could be easily and quickly onboarded using SD-WAN Manager.
- **Configuration** \- Manage the configuration of your devices from a single place through a reusable method, called _Configuration Groups_. that will allow to scale quicker. Streamlined workflows for security policies and cloud connectivity.
- **Monitoring** \- Monitor your devices, sites and applications. Get alerts and events, take advantage of SD-WAN's Manager notification capabilities.
- **Software management** \- Distribute, install and activate software images simply and quickly.
- **Troubleshooting** \- Run different operations from vManage like SSH sessions, speed tests, traceroutes and more.
- **Transition to** **SD-WAN**\- If you have plans to deploy SD-WAN, SD-Routing will be a great place to start getting familiar with the SD-WAN Manager and simplify the migration.

### SD-WAN Manager Overview

Let me give you a quick tour of the SD-WAN Manager and a brief view of some of the features. If you are familiar with it, you are probably still going to be surprised by the new look on 20.13. Check it out!

#### Network Overview

When we first log-in, the network overview is presented so we can quickly determine how many devices and controllers are actively connected, application information, and more.

Hey, where did the SD-WAN Controller go?!

![](/wp-content/uploads/2024/02/SDR-20.png)SD-WAN Manager 20.13

The Magnetic look and feel will remind you of other Cisco Security Products or Meraki Dashboard, this is great to keep the experience consistent. Notice top right there is a SD-Routing toggle button, this is very useful to show the information concerning SD-Routing and the reason why we don't see the SD-WAN Controller.

#### Monitor Devices

If we want to see more detailed device information we can visit the _Devices_ tab. SD-WAN Manager is constantly refreshing this information so we have an accurate view. Notice devices are listed as _**SD-Routing**_.

![](/wp-content/uploads/2024/02/SDR-12-1.png)

BR20 is not having a good health due to high memory utilization, could we find out when this started? Let's double click on it.

![](/wp-content/uploads/2024/02/SDR-14.png)

It started going beyond 75% at around 12:30, this was because I activated performance monitor to get app performance information.

Let's focus on the left menu, look to all the available **monitoring** options including **applications, security features** and **real time information**. The **troubleshooting** section is the place to go to use the tools I [listed earlier](#tshoot).

#### Configuration

In 20.13/17.13 support for _SD-Routing Configuration Groups_ was added. With them, you can build Feature Profiles based on parcels, which are single elements that together conform the entire router configuration. In 20.13 the following parcels are available.

![](/wp-content/uploads/2024/02/SDR-31.png)![](/wp-content/uploads/2024/02/SDR-30.png)

We have the CLI configuration Profile to push whatever is not available through parcels. We can define variables to make our Profile reusable across multiple devices. You can also use an entire CLI Profile instead of using parcels.

![](/wp-content/uploads/2024/02/SDR-33.png)![](/wp-content/uploads/2024/02/SDR-22.png)

By the way, if you have a mix of SD-WAN and SD-Routing devices, you will see both Configuration Groups listed there. SD-WAN has (for now) a richer set of parcels, this should be gradually shifted to support more non-CLI based configuration for SD-Routing.

#### Workflows

The workflows library will help us easily achieve certain actions like **onboard devices, security configuration, software upgrades** and more.

![](/wp-content/uploads/2024/02/SDR-23.png)

![](/wp-content/uploads/2024/02/SDR-24.png)![](/wp-content/uploads/2024/02/SDR-25.png)

Instead of clicking around on multiple pages, we can navigate step by step as we are presented with everything we need. I personally like how workflows simplify things for us.

#### Cloud Connectivity

Most likely your organization will use some kind of cloud connectivity, either to access apps or run workloads. SD-Routing has got you covered.

You can automate the connection to the cloud provider so your network will be extended to access those resources.

![](/wp-content/uploads/2024/02/SDR-6.png)![](/wp-content/uploads/2024/02/SDR-26.png)

There are multiple options here. Some of this is also provided for SD-WAN so I recommend checking the [config guides](https://www.cisco.com/c/en/us/td/docs/routers/cloud_edge/c8300/software_config/cat8300swcfg-xe-17-book/m-cloud-onramp-for-sd-routing.html) to see what is available for SD-Routing.

### Conclusion

The purpose of this post is to show what is possible with SD-Routing and the gap that it's aiming to close. Adopting this technology could positively impact your operational workflows, enhance network agility, and optimize resource utilization. If your organization has not yet adopted any form of SDN, I invite you to think about your day-to-day processes and identify the main challenges, things that could be done more efficiently and how SD-Routing could help you with them.

Let me know what you think and I will see you on the next one!
