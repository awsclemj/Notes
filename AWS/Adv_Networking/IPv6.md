- [Design an IPv6 Network](#design-an-ipv6-network)
  - [Enabling IPv6 on VPC](#enabling-ipv6-on-vpc)
  - ["Private" IPv6 Subnets](#%22private%22-ipv6-subnets)

# Design an IPv6 Network
Notes are taken from Linux Academy's Advanced Networking Specialty course

## Enabling IPv6 on VPC
* IPv6 CIDR can optionally be assigned during VPC creation or added on later
* Instances launched with IPv6 associate the IPv6 address with eth0
* IPv6 addresses are removed when instances are terminated
* SGs/NACLs can be used for access control
* IPv4 addresses cannot be removed from VPC/instances
* Things to keep in mind:
  * Update your route tables with destinations for IPv6 traffic
  * Update your SGs/NACLS for IPv6 access control
  * After assigning IPv6 addresses to any existing instances, it may be necessary to configure the instance to use DHCPv6

## "Private" IPv6 Subnets
* To keep an instance private, you can do the following:
  * Restrict access with SGs/NALCs
  * Use an egress-only IGW and route traffic from private subnets there
* EIGW facts:
  * Stateful
  * SGs cannot be assigned