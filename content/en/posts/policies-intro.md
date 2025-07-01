---
author: Alex
category:
  - sdwan
  - policies
draft: false
date: "2025-07-01T00:00:00+00:00"
title: "Quick Guide: Control vs Data Policies in Cisco SD-WAN"
description: Learn the key differences between Cisco SD-WAN control and data policies, when to use each, and how they impact routing and traffic forwarding 
summary: Learn the key differences between Cisco SD-WAN control and data policies, when to use each, and how they impact routing and traffic forwarding
url: /policies-intro-en/
tag:
  - policies
---

## Introduction

Cisco SD-WAN provides with powerful tools to shape both routing behavior and how traffic flows through the overlay. At the core of this flexibility are two key policy types: **Control Policies** and **Data Policies**. Understanding the difference between them is crucial for designing scalable and optimized SD-WAN deployments.

In this post, we’ll explore down what each policy type does, how they differ, and when to use one over the other.

## Centralized and Localized Polices

Policies are categorized based on where they are configured and applied, they could be _Centralized_ and _Localized_ policies. The Manager uses **netconf/yang** to send policy configuration.  

**Centralized Policies**

Configured on the Controller (vSmart) and pushed to a **selected group** of SD-WAN routers through Overlay Management Protocol (**OMP**). Includes:
- Control policies 
- Data policies
- App-aware routing policies

**Localized Policies**

Configured directly on the SD-WAN routers and **only affect that specific device**. Includes: 
- Local ACLs
- Route policies
- QoS settings

![](/wp-content/uploads/2025/06/policies1.png)


## What Are Control Policies?

Control policies in Cisco SD-WAN are applied **on the control plane** - specifically, they define how routing information is exchanged between Controllers and SD-WAN routers.

**Key characteristics:**

* **Applied on Controllers**
* Influence route advertisements and TLOC propagation
* Define what routes are accepted, modified, or rejected

**Typical use cases:**

- Create topologies (hub/spoke, partial mesh, etc)
- Controlling route leaks between VPNs
- Filtering or tagging OMP routes

Think of control policies as a way to shape the SD-WAN topology itself by deciding which routes exist and where they are allowed to go.

## What Are Data Policies?

Data policies, on the other hand, operate on the **data plane**. They define how actual traffic is handled.

**Key characteristics:**

* Also configured on the Controllers but **applied on SD-WAN routers**
* Operate at forwarding level, not related to route advertisements
* Can match on various fields: source/destination IPs, ports, applications, etc.

**Typical use cases:**

* Application-aware traffic steering
* Applying QoS classes
* Blocking certain types of traffic (for example, using ACL-style rules)

Using an analogy: Control policies are the city planners deciding where roads are built, data policies are the traffic rules that govern how vehicles move through them. 

## Key Differences at a Glance

| Feature              | Control Policy                | Data Policy                      |
| -------------------- | ----------------------------- | -------------------------------- |
| **Plane**            | Control plane                 | Data plane                       |
| **Applied on**       | Controller                    | Wan Edge             |
| **Affects**          | Route advertisements          | Actual traffic forwarding        |
| **Common Use Cases** | Route filtering, TLOC mapping | Traffic steering, QoS, ACLs      |
| **Directionality**   | Between controllers           | From controller to edge routers  |

## Policy Execution Order

The following steps happen when a packet is passing through a WAN Edge router:

![](/wp-content/uploads/2025/06/policies2.png)

**Local ingress policies**, such as interface level ACLs and QoS settings, are applied first. These can filter packets and apply marking or re-marking for QoS.

**Centralized Application-Aware Routing** is evaluated next. It makes forwarding decisions only between equal-cost paths already present in the routing table.

Then, the **centralized data policy** is evaluated. It has the ability to override any forwarding decisions made by AAR.

Once policies are applied, the system performs **routing and forwarding** lookups to identify the appropriate egress interface.

**Queueing and scheduling** mechanisms are applied to shape traffic according to QoS settings.

Finally, the **local egress policy** is applied before the packet is placed on the wire.

Understanding this order is key to avoiding misconfigurations and ensuring your policies behave as expected. A common mistake is unintentionally overriding an AAR routing decision by applying a centralized data policy that matches the same traffic and redirects it elsewhere.

## When to Use What

Choosing between control and data policies depends on **what problem you’re solving**:

* Want to **limit which routes are shared** between sites? → Use a **control policy**.
* Need to **steer certain applications** over MPLS or DIA? → That’s a **data policy** job.
* Trying to **prevent route advertisement loops** or control inter-VPN leakage? → Control policy.
* Looking to **prioritize VoIP traffic** or drop peer-to-peer file sharing? → Data policy.

In many designs, both are used together. For example, a control policy might ensure only selected prefixes are reachable between sites, while a data policy handles how the matching traffic is prioritized and forwarded.

## Conclusion

Control and data policies serve distinct but complementary purposes in Cisco SD-WAN. One governs the **topology and routing decisions**, the other shapes **how traffic moves** through that topology.

Understanding these differences allows you to build smarter and more efficient SD-WAN environments. Want to go deeper? Check out Cisco’s official docs on [Control Policies and Data Policies](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/policy-overview.html).

Also, check out my deep dive on [Application-Aware Routing](/demystifying-aar-1-3-the-foundations/), and learn how SD-WAN can automatically move traffic to best performing paths and guarantee a good user experience.
