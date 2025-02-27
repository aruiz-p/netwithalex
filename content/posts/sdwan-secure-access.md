---
draft: true
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

Cisco Secure Access is a robust solution that addresses these challenges. It offers top notch security by integrating advanced technologies and access controls. This means that users can get a _**secure and direct**_ connectivity from SD-WAN sites to the internet or private/public clouds.

Let's see how it works 

## SD-WAN Meets Secure Access

Starting with SD-WAN version 20.13/17.13, an integration with Secure Access is now available out of the box.

With this integration, automatic IPSec tunnels can be established to the primary and secondary Secure Access Data Centers closest to your router’s location, ensuring optimal performance. These tunnels route traffic while enforcing your organization's security policies, providing a simple and powerful way to improve both security and connectivity.

At the core of Secure Access are [Network Tunnel Groups (NTGs)](https://docs.sse.cisco.com/sse-user-guide/docs/manage-network-tunnel-groups), which manage IPSec connections. Each NTG includes a primary and a secondary Secure Access Data Center. While configuring tunnels to both data centers is not mandatory, it is highly recommended to ensure high availability in case one becomes unavailable.

![](/wp-content/uploads/2025/02/sse-topo.png)

It is possible to configure up to 16 tunnels, 8 active and 8 backup, allowing for load balancing across active tunnels to increase the available bandwidth. 

To _automatically_ establish the tunnels, the SD-WAN Manager and router need internet connectivity and DNS lookup enabled. This allows the device to determine its own public ip address, communicate it to the Manager and get assigned the nearest primary and secondary SSE Data Centers.

![](/wp-content/uploads/2025/02/get-ip.png)

Once the tunnels are established, traffic going through them will be secured by the [core security features of SSE](https://www.cisco.com/c/en/us/products/collateral/security/secure-access/hybrid-workforce-cloud-agile-security-ds.html#CiscoSecureAccessproductoverview): FWaaS, CASB, ZTNA and SWG and more.  

## Configuration Steps

### Create API Key 

To start, on SSE [create an API key](https://docs.sse.cisco.com/sse-user-guide/docs/add-secure-access-api-keys) for the Manager to securely connect. Make sure the following privileges are granted:

- Deployment / Network Tunnel Group - **Read/Write**
- Deployment / Tunnels - **Read/Write**
- Deployment / Regions - **Read**

You will get your _API Key and Key Secret_

### Enter Credentials on the Manager
Next, input the information on the SD-WAN Manager under **_Administration > Settings > Cloud Credentials > SSE_**

![](/wp-content/uploads/2025/02/sse-config.png)

### Create SSE Policy on the Manager

You need to create a new SSE Policy. This is where you input information about the IPSec tunnels. 

From **_Configuration > Policy Groups > Secure Service Edge_**

The following is the minimum you need:
- Tracker IP - Used to confirm the tunnel is operational.
- Tunnel - At least 1 tunnel. Enter _tunnel name, source interface_ and select _primary/secondary DC_
- Interface Pair - Specify _active and backup tunnels_. Select _none_ as backup if there's only 1 tunnel. 

**Note** You can select the SSE region of your preference or use _Auto_ to automatically select it.

![](/wp-content/uploads/2025/02/tunnel-config.png)

Then, create a Policy Group, associate a device and add the SSE policy

![](/wp-content/uploads/2025/02/policyg.png)

### Redirect service side traffic

Now, we need to redirect traffic from users to the tunnel. There are two options:

#### Service Route of type SSE 

On the _Service VPN_ feature, add a _Service Route_ with SSE Provider **_Cisco-Secure Access_**

![](/wp-content/uploads/2025/02/static-sse.png)

This will be added in the following way:

```
ip sdwan route vrf 10 0.0.0.0/0 service sse Cisco-Secure-Access
```

You can select what traffic gets forwarded to SSE by modifying the route, however this approach doesn't give a lot of flexibility compared to the second option.

#### Data Policy

Data Policies provide more flexibility to redirect traffic. We can not only match on the destination, but also on other useful elements such as source, applications and more. 

![](/wp-content/uploads/2025/02/match-sse.png)

This is an example matching all traffic coming from my VPN 10 users and setting an action of Secure Service Edge. Notice that we can enable the option to _Fallback to routing_ in case tunnels are unavailable. 

![](/wp-content/uploads/2025/02/data-sse.png)

### Validations from the Manager




### Validations from the User

To verify that users are using SSE we have a couple of options

The first one is to visit policy.test.sse.com, if traffic is correctly redirected it will show something like this:
![](/wp-content/uploads/2025/02/policy-sse.png)

On SSE side, there is a policy denying traffic to social media apps, let's see the result of trying to access x.com 

![](/wp-content/uploads/2025/02/x-sse.png)

Lastly, accessing welcome.umbrella.com will let us know if user is protected  

![](/wp-content/uploads/2025/02/umbrella-sse.png)


## Additional considerations

- ECMP is available when multiple active tunnels are active.
- Un-equal load balance can be achieved through weight assignment to IPSec tunnels. 
- Fallback to routing is available when tunnels aren't available
- Trackers are customizable - custom URL and thresholds to meet desired SLAs

I hope you have learned something useful! See you on the next one 👋

