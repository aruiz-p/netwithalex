---
author: Alex
category:
  - sdwan
date: "2025-05-05T14:20:31+00:00"
draft: false
title: "Cisco SD-WAN On-Prem ZTP: Automating Router Onboarding"
description: Plug and Play (PnP) lets you onboard SD-WAN devices automatically via Cisco Cloud. This post explains how to achieve zero-touch provisioning (ZTP) in an airgapped, on-premises environment.  
summary: Plug and Play (PnP) lets you onboard SD-WAN devices automatically via Cisco Cloud. This post explains how to achieve zero-touch provisioning (ZTP) in an airgapped, on-premises environment. 
url: /ztp-on-prem-en/
tag:
  - ZTP
  - onboarding
---
## Introduction

When deploying a new remote site you want the process to be as easy as possible. You don’t want to ship the device to an intermediate location just to preload a configuration, only to ship it again to the final destination. You also don’t want to travel onsite yourself, connect a console cable, and manually configure each device. Now multiply that effort by dozens or hundreds of sites 🤯

To solve this challenge, Cisco created an automated onboarding process called Plug and Play (**PnP**) or Zero Touch provisioning (**ZTP**). The idea is simple: a router powers on, obtains an IP address, locates its SD-WAN overlay, connects to the controllers, and downloads its configuration—all without human intervention beyond plugging it in.

A key step in this process is helping the router discover the SD-WAN overlay. This can be achieved via the Cisco Cloud or, in air-gapped environments, by hosting your **own On-Prem ZTP Server**. Today we are going to explore the second option.  

## PnP/ZTP operation

The following happens when the router boots up with no configuration:

![](/wp-content/uploads/2025/05/ztp5.png)

**Note** this process is available for **hardware devices** only. 

The ZTP domain is defined on the DHCP server, for example, if the domain name is **_cisco.com_**, the router will try to resolve ztp.**_cisco.com_**. For more information you can check [Cisco's documentation](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/sdwan-xe-gs-book/cisco-sd-wan-overlay-network-bringup.html#c_Start_the_Enterprise_ZTP_Server_7841.xml)

## Configuration

We need to complete the following tasks to make this work. 

1. Add and configure ZTP server
2. Upload device list to ZTP server
3. Prepare device configuration on vManage
4. Configure DHCP and DNS servers
5. Trigger PnP process 

### Add and configure ZTP Server

The ZTP server uses the same image as a regular Validator. Follow the usual process to bring up a new VM with connectivity to the Manager. 

The following is the minimum configuration needed:

```
vbond# show run system
system
 host-name               ztp-server
 system-ip               10.10.10.194
 site-id                 5
 sp-organization-name    SDWAN-LAB123
 organization-name       SDWAN-LAB123
 vbond 192.168.200.2 local ztp-server 

vpn 0
 interface eth0
  ip dhcp-client
  ipv6 dhcp-client
  no shutdown
 !
 interface ge0/0
  ip address 192.168.200.3/24
  no shutdown
 !
 ip route 0.0.0.0/0 192.168.200.253
!
 ```
Notice the **_ztp-server_** suffix, this will indicate the device to act as a ZTP server

From the Manager, add the ZTP server to the Controller list: 

![](/wp-content/uploads/2025/05/ztp1.png)

Depending on the Controller Authentication method, generate and sign the CSR. 

If you are using Enterprise Certificates, you will need to install the root certificate and the signed certificate. 

For example: 

```
ztp-server# request root-cert-chain install home/admin/root-ca.crt
ztp-server# request certificate install home/admin/ztp.crt
```
**Note** The ZTP server doesn't have any control connections to the Manager or any other controller.

### Upload device list to ZTP server

Now that the ZTP server is installed, we need to provide the device list that will connect with the ZTP server. 

The easiest way is to get the _serialFile.viptela_ from the PnP portal and copy it locally. 

```
ztp-server:~$ ls -l | grep serial
-rw-r--r-- 1 admin admin  2364 Apr 24 21:41 serialFile.viptela
```
then execute 

```
ztp-server# request device-upload chassis-file home/admin/serialFile.viptela 
Uploading chassis numbers via VPN 0
Copying ... /home/admin/serialFile.viptela via VPN 0
file:  /tmp/tmp.CbUWf8GnSN/viptela_serial_file
PnP
Verifying public key received from PnP against production root cert
is_public_key_ok against production root ca:  OK
Signature verified for viptela_serial_file
final file:  /tmp/tmp.CbUWf8GnSN/viptela_serial_file
Signature verification Suceeded.
Success: Serial file is /tmp/tmp.CbUWf8GnSN/viptela_serial_file
INFO: Input File specified was '/usr/share/viptela/chassis_numbers.tmp'
INFO: # of complete chassis entries written: 12
Json to CSV conversion succeeded!
Successfully loaded the chassis numbers file to the database.
```

To verify the list:

```
ztp-server# show ztp entries 
                                                                                                                                                  ROOT     
                                                                                                                           VBOND  ORGANIZATION    CERT     
INDEX  CHASSIS NUMBER                                   SERIAL NUMBER                             VALIDITY  VBOND IP       PORT   NAME            PATH     
-----------------------------------------------------------------------------------------------------------------------------------------------------------
...
23     ASR1001-HX-XXXXXXXXXXX                           XXXXXXXX                                  valid     192.168.200.1  12346  SDWAN-LAB123  default   
```

### Prepare device configuration on vManage

To get this done I will use a configuration group, but it can be done with templates too. 

![](/wp-content/uploads/2025/05/ztp2.png)

I create the device configuration and push it. The task is scheduled as device is offline. 

![](/wp-content/uploads/2025/05/ztp3.png)

### Configure DHCP and DNS servers

To make it simple I use an intermediate switch as my DHCP and DNS Server with the following configuration:

```
ip dhcp pool ASR
 vrf MPLS
 network 192.168.11.4 255.255.255.252
 default-router 192.168.11.6 
 dns-server 192.168.11.6 
 domain-name cisco.com

ip host vrf MPLS ztp.cisco.com 192.168.200.3
ip host vrf MPLS devicehelper.cisco.com 192.168.200.3
```
Ok, we are all set to see it in action

### Trigger PnP process 

To trigger the PnP process the device has to have blank configuration. I will use the following command to reset the config and trigger the process. 

```
Router#request platform software sdwan config reset 

%WARNING: Bootstrap file doesn't exist and absence 
of it can cause loss of connectivity to the controller.
For saving bootstrap config, use:
request platform software sdwan bootstrap-config save
Proceed to reset anyway? [confirm]

Backup of running config is saved under /bootflash/sdwan/backup.cfg
Config reset requested from a console session.
Waiting for up to 60 seconds for IOS to initiate reload or report failure.
IOS return status: "cfgreset_proceed"
Config reset is raised successfully, device will reload shortly.
```

You need console access to see the following logs: 

Device boots and PnP discovery start. 

```
*May  4 04:18:42.659: %PNP-6-PNP_DISCOVERY_STARTED: PnP Discovery started
```
The router gets an ip address, default router, domain name and dns server. 

```
Autoinstall trying DHCPv4 on GigabitEthernet0/0/0,GigabitEthernet0/0/1,GigabitEthernet0/0/2,GigabitEthernet0
...
*May  4 04:19:44.999: %PKI-2-NON_AUTHORITATIVE_CLOCK: PKI functions can not be initialized until an authoritative time source, like NTP, can be obtained.
Acquired IPv4 address 192.168.11.5 on Interface GigabitEthernet0/0/1
Received following DHCPv4 options:
        domain-name     : cisco.com
        dns-server-ip   : 192.168.11.6

```

Device tries to resolve domains and gets redirected to _ztp.cisco.com_

```
*May  4 04:20:08.694: %PNP-3-PNP_CCO_SERVER_IP_UNRESOLVED: CCO server (devicehelper.cisco.com.) can't be resolved (1/5) by (pid=619, pname=PnP Agent Discovery, time=04:20:08 UTC Sun May 4 2025)
...
*May  4 04:20:20.696: %IOSXE_SDWAN_CONFIG-5-PNP_REDIRECT: PnP Redirect Msg: Org name "" Host "ztp.cisco.com." port 0 intf GigabitEthernet0/0/1
*May  4 04:20:42.010: %PNP-6-PNP_REDIRECTION_DONE: PnP Redirection done (1) by (pid=619, pname=PnP Agent Discovery)
*May  4 04:20:42.010: %PNP-6-PNP_SDWAN_STARTED: PnP SDWAN started (1) via (pnp-sdwan-vbond-ztp-discovery) by (pid=619, pname=PnP Agent Discovery)
*May  4 04:20:42.811: %PNP-6-PNP_DISCOVERY_DONE: PnP Discovery done successfully (PnP-VBOND-ONPREM-ZTP-IPV4) profile (pnp-zero-touch) 
```
We can confirm _ztp.cisco.com_ was resolved 
```
ASR1K-2#show pnp trace | i  ztp
[05/04/25 04:20:10.695 UTC B7 619] 1: VBOND_ONPRIME_ZTP hostname ztp.cisco.com. resolved to 192.168.11.6 on interface GigabitEthernet0/0/1
[05/04/25 04:20:10.695 UTC B8 619] host_name is ztp.cisco.com. vbond_ipv4_address is 192.168.11.6, interface is GigabitEthernet0/0/1
```
Next, the router connects to the ZTP-Server and gets redirected the Validator
```
*May  4 04:21:17.991: %Cisco-SDWAN-Router-vdaemon-6-INFO-1400002: Notification: 5/4/2025 4:21:17 control-connection-state-change severity-level:major host-name:"Router" system-ip::: personality:vedge peer-type:vbond peer-system-ip::: peer-vmanage-system-ip:0.0.0.0 public-ip:192.168.200.3 public-port:12346 src-color:default remote-color:default uptime:"0:00:00:00" new-state:up
...
*May  4 04:21:19.242: %Cisco-SDWAN-Router-vdaemon-6-INFO-1400002: Notification: 5/4/2025 4:21:19 org-name-change severity-level:minor host-name:"Router" system-ip::: old-organization-name:"" new-organization-name:"SDWAN-LAB123"
*May  4 04:21:21.597: %Cisco-SDWAN-Router-vdaemon-6-INFO-1400002: Notification: 5/4/2025 4:21:21 control-connection-state-change severity-level:major host-name:"Router" system-ip::: personality:vedge peer-type:vbond peer-system-ip::: peer-vmanage-system-ip:0.0.0.0 public-ip:192.168.200.1 public-port:12346 src-color:default remote-color:default uptime:"0:00:00:00" new-state:up
```

Eventually, the device connected to the Manager and Controller and pulls the configuration
```
*May  4 04:21:23.924: %Cisco-SDWAN-Router-vdaemon-6-INFO-1400002: Notification: 5/4/2025 4:21:23 control-connection-state-change severity-level:major host-name:"Router" system-ip::: personality:vedge peer-type:vmanage peer-system-ip:10.10.10.2 peer-vmanage-system-ip:0.0.0.0 public-ip:192.168.100.1 public-port:12746 src-color:default remote-color:biz-internet uptime:"0:00:00:00" new-state:up
*May  4 04:21:24.299: %Cisco-SDWAN-CSS-SDWAN-POD1-ASR1K-2-OMPD-5-NTCE-400003: Operational state changed to UP
*May  4 04:21:43.981: %DMI-5-AUTH_PASSED: R0/0: dmiauthd: User 'vmanage-admin' authenticated successfully from 10.10.10.2:42962  for netconf over ssh.
```
![](/wp-content/uploads/2025/05/ztp4.png)

```
ASR1K-2#show sdwan control connections | i up
vsmart  dtls 10.10.10.3      5          1      192.168.100.2                           12346 192.168.100.2                           12346 SDWAN-LAB123          mpls            No    up     0:02:02:51 0           
vbond   dtls 0.0.0.0         0          0      192.168.200.1                           12346 192.168.200.1                           12346 SDWAN-LAB123          mpls            -     up     0:02:02:54 0           
vmanage dtls 10.10.10.2      5          0      192.168.100.1                           12946 192.168.100.1                           12946 SDWAN-LAB123          mpls            No    up     0:02:02:49 0           
```

## Conclusion

On-Prem ZTP server extends the ability to onboard hardware devices to those network where internet access is restricted. 

Automating router onboarding with On-Prem ZTP provides a great alternative for organizations that need full control over their provisioning process or operate in air-gapped environments. By replicating the Plug and Play process locally, you eliminate the need for manual configuration at remote sites while keeping sensitive environments isolated.

With the right setup, onboarding new sites becomes an easy and repetable process that will save precious time, reduce human error and scales SD-WAN deployments.

👉 Interested in setting up On-Prem ZTP for your network? Leave a comment or reach out, I’d love to help or answer your questions!