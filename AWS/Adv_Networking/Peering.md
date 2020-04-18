- [VPC Peering](#vpc-peering)
  - [Overview](#overview)
  - [Multiple VPC Peers](#multiple-vpc-peers)

# VPC Peering
Notes are taken from Linux Academy's Advanced Networking Specialty course

## Overview
* Route IPv4/6 traffic between VPCs as if they belonged to the same network
* Intra-region, inter-region, and inter-account supported
* No gateway required
* One-to-one relationship
* Limits:
  * Active peering connections limited to 50
  * Routes are required with a target of pcx-xxxxx
  * Longest prefix match applies
  * Inter-region limitations:
    * SG rules cannot reference peered VPC SGs
    * IPv6 not supported
    * No jumbo frames
* Features:
  * Enabling DNS resolution across peered VPC allows public DNS to resolve to private IP addresses
  * Inter-region peering traffic is encrypted and sent across the AWS backbone network
  * For same region peers, SGs can be referenced as sources across VPCs
  * You can access PrivateLink endpoints over VPC peering connections

## Multiple VPC Peers
* Transitive routing is not supported in VPC peering. For example, if VPC A had a PCX with both VPC B and VPC C, VPC B couldn't access resources in VPC C and vice versa
* You must have a full mesh topology for all VPCs to be able to communicate with one another
* Hub-and-spoke topologies are a common use case for management or shared-services VPCs

