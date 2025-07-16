---
author: Alex
draft: false
category:
  - sdwan
date: "2025-07-15T00:00:00+00:00"
title: Simplifying Tunnel Management with Tunnel Groups in Cisco SD-WAN
summary: Learn how Tunnel Groups in Cisco SD-WAN simplify tunnel management at scale. Discover how they work, how they differ from control policies, and when to use them with real-world examples.
description: Learn how Tunnel Groups in Cisco SD-WAN simplify tunnel management at scale. Discover how they work, how they differ from control policies, and when to use them with real-world examples.
url: /tunnel-groups-en/
tag:
  - bfd
  - topology
---
## Introduction

SD-WAN provides a highly flexible overlay architecture that allows building tunnels across multiple transport types. A key aspect of this flexibility is how traffic is forwarded across tunnels between edge devices based on transport colors such as **private** (_mpls, private1, etc_) and **public** (_public-internet, biz-internet, etc_)

There are a couple of ways to control what tunnels are created, most notably [Control Policies](/policies-intro-en) and Tunnel Groups. 

**Tunnel Groups** allow network admins to logically group tunnels (TLOCs) under a common label, simplifying how tunnels are established between devices. This approach makes tunnel management more scalable and easier to maintain. In this post, we’ll explore what Tunnel Groups are, how they work, and how they can serve as an alternative to Control Policies in certain scenarios.

## What are tunnel groups?

A Tunnel Group is a _numeric label_ that represents a set of TLOCs (tunnels) grouped logically based on a defined intent. 

{{< figure src="/wp-content/uploads/2025/07/tg1.png" title="Figure 1" caption="Tunnel Group configuration on Feature Template" >}}

Once a Tunnel Group ID is configured the router will:

- Establish a tunnel with TLOCs that have the same tunnel group ID
- Establish a tunnel with TLOCs that have no tunnel group ID (**value of 0**)

The Tunnel Group ID is propagated through OMP, just like other TLOC attributes. One key difference between Tunnel Groups and Control Policies is where they take effect: Tunnel Groups control tunnel establishment locally on the device, while Control Policies determine which TLOCs are advertised by the Controller (vSmart) at a central level. Both mechanisms can be used independently or together, depending on your design requirements.

## Tunnel Group Behavior: Real Output Walkthrough

Let's take this topology as a reference for this scenario. 

{{< figure src="/wp-content/uploads/2025/07/tg3.png" alt="Topology" title="Figure 2" caption="Reference Topology" >}}

Let's validate how **Tunnel group IDs** influence which tunnels are actually formed by looking at OMP TLOCs and BFD sessions from a device in site 20.

```
Barcelona_BR20-1#show sdwan omp tlocs table
...

                                                                                                                                                                                   PUBLIC           PRIVATE                      AFFINITY    
ADDRESS                                           TENANT                                                        PSEUDO                   PUBLIC                   PRIVATE  PUBLIC  IPV6    PRIVATE  IPV6     BFD                 GROUP       
FAMILY   TLOC IP          COLOR            ENCAP  ID        FROM PEER        STATUS    TENANT NAME              KEY     PUBLIC IP        PORT    PRIVATE IP       PORT     IPV6    PORT    IPV6     PORT     STATUS  REGION ID   NUMBER      
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ipv4     1.1.10.1         mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.11.0.2        12426   21.11.0.2        12426    ::      0       ::       0        down    None        None        
         1.1.10.1         biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.11.0.2        12346   31.11.0.2        12346    ::      0       ::       0        down    None        None        
         1.1.10.2         mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.12.0.2        12346   21.12.0.2        12346    ::      0       ::       0        down    None        None        
         1.1.10.2         biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.12.0.2        12346   31.12.0.2        12346    ::      0       ::       0        down    None        None        
         1.1.20.1         mpls             ipsec  0         0.0.0.0          C,Red,R   [Default]                1       21.21.0.2        12346   21.21.0.2        12346    ::      0       ::       0        up      None        None        
         1.1.20.1         biz-internet     ipsec  0         0.0.0.0          C,Red,R   [Default]                1       31.21.0.2        12346   31.21.0.2        12346    ::      0       ::       0        up      None        None        
         1.1.20.2         mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.22.0.2        12346   21.22.0.2        12346    ::      0       ::       0        down    None        None        
         1.1.20.2         biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.22.0.2        12346   31.22.0.2        12346    ::      0       ::       0        down    None        None        
         1.1.100.1        mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.101.0.2       12406   21.101.0.2       12406    ::      0       ::       0        up      None        None        
         1.1.100.1        biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.101.0.2       12346   31.101.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.100.2        mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.102.0.2       12346   21.102.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.100.2        biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.102.0.2       12346   31.102.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.200.1        mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.201.0.2       12346   21.201.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.200.1        biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.201.0.2       12346   31.201.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.200.2        mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.202.0.2       12346   21.202.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.200.2        biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.202.0.2       12386   31.202.0.2       12386    ::      0       ::       0        down    None        None        
```

There are 14 TLOCs received from vSmart, but only few tunnels up:

```
Barcelona_BR20-1#sh sdwan bfd sessions 
                                      SOURCE TLOC      REMOTE TLOC                                      DST PUBLIC                      DST PUBLIC         DETECT      TX                              
SYSTEM IP        SITE ID     STATE       COLOR            COLOR            SOURCE IP                                       IP                                              PORT        ENCAP  MULTIPLIER  INTERVAL(msec  UPTIME          TRANSITIONS   
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
1.1.100.1        100         up          mpls             mpls             21.21.0.2                                       21.101.0.2                                      12406       ipsec  7           1000           0:02:23:34      2             
1.1.100.2        100         up          mpls             mpls             21.21.0.2                                       21.102.0.2                                      12346       ipsec  7           1000           0:02:23:33      1             
1.1.200.1        200         up          mpls             mpls             21.21.0.2                                       21.201.0.2                                      12346       ipsec  7           1000           6:03:57:35      0             
1.1.200.2        200         up          mpls             mpls             21.21.0.2                                       21.202.0.2                                      12346       ipsec  7           1000           6:03:57:36      0             
1.1.100.1        100         up          biz-internet     biz-internet     31.21.0.2                                       31.101.0.2                                      12346       ipsec  7           1000           0:02:23:34      1             
1.1.100.2        100         up          biz-internet     biz-internet     31.21.0.2                                       31.102.0.2                                      12346       ipsec  7           1000           0:02:23:33      1             
1.1.200.1        200         up          biz-internet     biz-internet     31.21.0.2                                       31.201.0.2                                      12346       ipsec  7           1000           6:03:57:35      0             
1.1.200.2        200         up          biz-internet     biz-internet     31.21.0.2                                       31.202.0.2                                      12426       ipsec  7           1000           0:00:00:08      0             
```
If we examine the TLOC details, we can see there are 4 TLOCs with a non-matching group-id of 10. The rest either match or are unconfigured ([0]). Remember devices within the same site won't form BFD tunnels. 


```
Barcelona_BR20-1#show sdwan omp tlocs | include tloc|mpls|internet|groups|site
tloc entries for 1.1.10.1      <<< non-matching Group ID
                 mpls
     site-id           10
     groups            [ 10 ]
     site-type        not set
tloc entries for 1.1.10.1       <<< non-matching Group ID
                 biz-internet
     site-id           10
     groups            [ 10 ]
     site-type        not set
tloc entries for 1.1.10.2       <<< non-matching Group ID
                 mpls
     site-id           10
     groups            [ 10 ]
     site-type        not set
tloc entries for 1.1.10.2       <<< non-matching Group ID
                 biz-internet
     site-id           10
     groups            [ 10 ]
     site-type        not set
tloc entries for 1.1.20.1    <<<< own TLOC 
                 mpls
     site-id           20
     groups            [ 20 ]
     site-type        not set
     site-id           20
     groups            [ 20 ]
)     site-type        not set
tloc entries for 1.1.20.1    <<<< own TLOC
                 biz-internet
     site-id           20
     groups            [ 20 ]
     site-type        not set
     site-id           20
     groups            [ 20 ]
)     site-type        not set
tloc entries for 1.1.20.2   <<<< same site
                 mpls
     site-id           20
     groups            [ 20 ]
     site-type        not set
tloc entries for 1.1.20.2   <<<< same site
                 biz-internet
     site-id           20
     groups            [ 20 ]
     site-type        not set
tloc entries for 1.1.100.1       #1
                 mpls
     site-id           100
     groups            [ 0 ]
     site-type        not set
tloc entries for 1.1.100.1       #2
                 biz-internet
     site-id           100
     groups            [ 0 ]
     site-type        not set
tloc entries for 1.1.100.2       #3
                 mpls
     site-id           100
     groups            [ 0 ]
     site-type        not set
tloc entries for 1.1.100.2        #4
                 biz-internet
     site-id           100
     groups            [ 0 ]
     site-type        not set
tloc entries for 1.1.200.1       #5
                 mpls
     site-id           200
     groups            [ 20 ]
     site-type        not set
tloc entries for 1.1.200.1       #6
                 biz-internet
     site-id           200
     groups            [ 20 ]
     site-type        not set
tloc entries for 1.1.200.2       #7
                 mpls
     site-id           200
     groups            [ 20 ]
     site-type        not set
tloc entries for 1.1.200.2       #8
                 biz-internet
     site-id           200
     groups            [ 20 ]
     site-type        not set
```

## Tunnel Group + restrict option

As you can see above, only tunnels between matching colors are established, meaning that the restrict option is enabled on the devices:

{{< figure src="/wp-content/uploads/2025/07/tg2.png" title="Figure 3" caption="Tunnel Group configuration with 'restrict' option" >}}

If the restrict option is removed from MPLS tunnel, let's see what happens with the BFD sessions:

```
Barcelona_BR20-1#show sdwan bfd sessions 
                                      SOURCE TLOC      REMOTE TLOC                                      DST PUBLIC                      DST PUBLIC         DETECT      TX                              
SYSTEM IP        SITE ID     STATE       COLOR            COLOR            SOURCE IP                                       IP                                              PORT        ENCAP  MULTIPLIER  INTERVAL(msec  UPTIME          TRANSITIONS   
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
1.1.100.1        100         up          mpls             mpls             21.21.0.2                                       21.101.0.2                                      12406       ipsec  7           1000           0:03:14:45      2             
1.1.100.2        100         up          mpls             mpls             21.21.0.2                                       21.102.0.2                                      12346       ipsec  7           1000           0:03:14:44      1             
1.1.200.1        200         up          mpls             mpls             21.21.0.2                                       21.201.0.2                                      12346       ipsec  7           1000           6:04:48:46      0             
1.1.200.2        200         up          mpls             mpls             21.21.0.2                                       21.202.0.2                                      12346       ipsec  7           1000           6:04:48:47      0             
1.1.100.1        100         up          mpls             biz-internet     21.21.0.2                                       31.101.0.2                                      12346       ipsec  7           1000           0:00:03:37      0             
1.1.100.2        100         up          mpls             biz-internet     21.21.0.2                                       31.102.0.2                                      12346       ipsec  7           1000           0:00:03:37      0             
1.1.200.1        200         up          mpls             biz-internet     21.21.0.2                                       31.201.0.2                                      12346       ipsec  7           1000           0:00:03:37      0             
1.1.200.2        200         up          mpls             biz-internet     21.21.0.2                                       31.202.0.2                                      12426       ipsec  7           1000           0:00:03:37      0             
1.1.200.1        200         up          biz-internet     mpls             31.21.0.2                                       21.201.0.2                                      12346       ipsec  7           1000           0:00:03:33      0             
1.1.200.2        200         up          biz-internet     mpls             31.21.0.2                                       21.202.0.2                                      12346       ipsec  7           1000           0:00:03:35      0             
1.1.100.1        100         up          biz-internet     biz-internet     31.21.0.2                                       31.101.0.2                                      12346       ipsec  7           1000           0:03:14:45      1             
1.1.100.2        100         up          biz-internet     biz-internet     31.21.0.2                                       31.102.0.2                                      12346       ipsec  7           1000           0:03:14:44      1             
1.1.200.1        200         up          biz-internet     biz-internet     31.21.0.2                                       31.201.0.2                                      12346       ipsec  7           1000           6:04:48:46      0             
1.1.200.2        200         up          biz-internet     biz-internet     31.21.0.2                                       31.202.0.2                                      12426       ipsec  7           1000           0:00:51:19      0             
```

Now there are tunnels between different colors, but honoring the Tunnel Group ID. 

So in conclusion, if the restrict option is configured along with Tunnel groups, tunnels will form if:
1. The TLOC color matches
2. Tunnel Group IDs match or one side doesn't have Tunnel Group configured (Group 0)

## Use Cases

Let's talk about some real life use cases for this feature. 

### Grouping private and public colors

There are multiple labels for private and public colors. Depending on your network design, you could end up having different color names per region. Example:

|Continent| Public Colors | Private Colors  |
|---------|-------------|-------------------|
|Europe   | biz-internet|   mpls            |
|America  | gold        | metro-ethernet    |

Using Tunnel Groups, we can easily separate tunnels between private and public colors:

{{< figure src="/wp-content/uploads/2025/07/tg4.png" title="Figure 4" caption="Color Grouping based on Tunnel Group ID" >}}


Tunnel Group 10 assigned to private colors will ensure only tunnels between mpls and metro-ethernet are established. Likewise, Tunnel Group 20 will allow public color tunnels, gold and biz-internet. 

### Horizontal Scaling 

When head-end routers need to build tunnels with many remote sites, each using multiple transport types, they can hit the platform's tunnel scale limit. Tunnel groups help in these situations by spreading out tunnel creation across different routers and remote branches, making the overall design more scalable.

{{< figure src="/wp-content/uploads/2025/07/tg5.png" title="Figure 5" caption="Horizontal Scaling Approach with Tunnel Groups" >}}

By doing this, each branch will be connected to both data centers while keeping the number of session on the head-end routers under control.  

### Creating Partial Mesh

If there is a need to create partial mesh, tunnel groups can come very handy. 

{{< figure src="/wp-content/uploads/2025/07/tg6.png" title="Figure 5" caption="Partial Mesh Topology" >}}

In this case devices with Tunnel Group 10 and with no Tunnel Group (0) will establish tunnels between them. Likewise, devices with Tunnel Group 20 and with no Tunnel Group (0) will create tunnels. 

## Conclusion

Tunnel Groups in Cisco SD-WAN provide a scalable way to control tunnel establishment across small or large deployments. By tagging TLOCs with numeric group IDs, you gain the ability to:

- Create logical tunnel groups without relying solely on color matching.
- Reduce the complexity of centralized control policies.
- Design more predictable and scalable topologies like partial mesh, hub-and-spoke, or horizontal scaling.

Whether you’re dealing with multi-region color naming inconsistencies, scaling head-end routers, or simply want a cleaner way to manage tunnels, Tunnel Groups offer a powerful tool to simplify your design.

When combined with the restrict option, Tunnel Groups give you fine-grained control over which tunnels are allowed — by color, by group, or both. And with the right tunnel group strategy, you can significantly reduce tunnel bloat while maintaining high availability and performance.
