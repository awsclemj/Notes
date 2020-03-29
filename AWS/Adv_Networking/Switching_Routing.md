- [Switching and Routing Fundamentals](#switching-and-routing-fundamentals)
  - [Switching Fundamentals](#switching-fundamentals)
    - [General](#general)
    - [VLANs, Trunks, and LACP](#vlans-trunks-and-lacp)
  - [Routing Fundamentals](#routing-fundamentals)
    - [Overview](#overview)
    - [BGP Overview](#bgp-overview)
    - [Autonomous Systems](#autonomous-systems)
    - [iBGP and eBGP](#ibgp-and-ebgp)

# Switching and Routing Fundamentals
Notes are taken from Linux Academy's Advanced Networking Specialty course

## Switching Fundamentals

### General
* Layer 2 - Data Link Layer
* Primarily used to break up collision domains
  * Each port on a switch is a collision domain, which avoids the problem of multiple hosts trying to communicate at once
* Switches use MAC addresses to send frames between devices
* Devices send ARP requests to learn the MAC address of other devices
* Switches store device MAC addresses in a MAC address table
* All devices connected to the same switch are within the same broadcast domain (unless VLANs are in use)

### VLANs, Trunks, and LACP
* **Virtual LANs (VLANs)** allow us to isolate certain devices so they can only communicate with one another
  * Tags are added to VLAN traffic to signal to the switch which VLAN traffic belongs to
* For separate VLANs to communicate with another, we would need either inter-VLAN routing (L3 switch) or a traditional router
* **Trunk ports** allow switches to pass traffic for more than one VLAN
  * Multiple IP addresses need to be configured on these ports so each separate VLAN can communicate with them
* **Link Aggregation Control Protocol (LACP)** allows you to bundle multiple network connections into a single logical connection
  * Often called a **port channel**
  * This allows for HA/failover should a connection fail on the bundle
  * Aggregates the bandwidth

## Routing Fundamentals

### Overview
* Routers and IP addresses live at layer 3
* Routers forward IP packets between different IP networks based on best path
* A NAT device is required to communicate between a private IPv4 network and the Internet
* All IPv6 addresses are public (no NAT)
* Sending traffic between different VLANs is routing
  * L3 switches are often used for this function
* When traffic leaves a network, it is forwarded to a *next hop* configured in the routing table
* The *next hop* router is the router the gives the shortest path to the destination
* Routing protocols are standards for how routers communicate with other routers
  * Examples: RIP, IGRP, EIGRP, OSPF, BGP

### BGP Overview
* Path vector protocol. Looks at available paths and calculates best path based on various metrics
  * BGP uses several rules, paths, weights, and priority to determine next hop
  * AS_PATH prepending is an example of how to influence path preference 
* **Autonomous Systems (ASes)** are a collection of routers with prefixes/policies under the control of a single entity
  * Typically operated by ISPs, large companies, universities, government agencies, etc. 
* A registered ASN is required to exhange information via BGP
* **Multi-Exit Descriminators** can be used to tell neighboring ASes which path is preferred for receiving traffic. Lower MED wins. 

### Autonomous Systems
* Each AS has a globally-unique identifier assigned
* Private ASNs also exist and must not be exposed to the Internet
* Public ASNs are assigned by IANA. Requirements:
  * Multi-homed network (two or more connections to the same or different networks)
  * BGPv4 capable router
  * Fully-functioning IGP on your network
* BGP uses *keep-alives* and a timer to determine a peer's liveliness. 
  * Dead peers are removed from the route table and advertisements 
* Bad ASNs can "break" the Internet
* BGP hijacking 
* Networks within an AS use an IGP such as OSPF, IS-IS, RIP, or EIGRP

### iBGP and eBGP
* iBGP - interior
  * IGP version of BGP -- only communicates within an AS. Usually used on border routers
  * Full mesh topology is necessary, as routes are not re-advertised to neighbors if they are learned via iBGP
* eBGP - exterior
  * Used to exchange routes with other ASes