- [Elastic Load Balancing](#elastic-load-balancing)
  - [Overview](#overview)
    - [Key facts](#key-facts)
  - [ELB Sandwich](#elb-sandwich)

# Elastic Load Balancing
Notes are taken from Linux Academy's Advanced Networking Specialty course

## Overview
* Distribute application or network traffic across multiple backend targets. Targets can be **EC2 instances, containers, or IP addresses**.
* ELB can improve availability and fault tolerance by using a combination of scaling, health checking, and distributing load across AZs.
* ELBs can be used to offlload encryption/decryption operations
* Types:
  * Application - layer 7 (HTTP(S))
  * Network - layer 4 (TCP/UDP)
  * Classic - layer 4 or 7

### Key facts
* Minimum of /27 subnet must be used with a minimum of 8 free IP addresses
* SGs/NACLs apply to ELBs
* Connection draining ensures client connections are closed before an instance is taken out of service. Default: 5 minutes
* SSL security policies can be applied to your ELB to restrict what ciphers are used in an SSL transaction
* Access logs are publishd every 60 mins, but the interval can be either 5 or 60 mins
* X-Forwarded headers are supported with CLB and ALB
* Sticky sessions are supported on CLB and ALB
  * CLB supports both application cookie session duration and time duration
  * ALB only supports time-based duration cookies
* ALB supports host and path-based routing
* ALB supports IP-based targets, which allow for targets outside of the VPC
* NLB and CLB support Proxy Protocol, which adds human-readable information to be passed in a request header
  * Includes source IP, dest IP, and ports

## ELB Sandwich
* A common network design is the ELB sandwich. ELBs terminate web traffic and then forward the traffic to a set of virtual firewalls or load balancers.
  * These virtual devices typically have some feature ELBs don't, such as DPI and/or IPS/IDS. 
* Traffic is then forwarded to internal ELBs, which forward requests to the backend web servers that serve the application frontend.
* Tip: if the virtual devices cannot resolve the FQDN of the ELBs in the second ELB tier, use NLB since it has a static IP address. 