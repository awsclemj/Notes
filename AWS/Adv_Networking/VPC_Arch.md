- [Designing/Architecting an AWS Network](#designingarchitecting-an-aws-network)
  - [VPC Reserved IP Addresses](#vpc-reserved-ip-addresses)
  - [DHCP Options Sets](#dhcp-options-sets)
    - [DHCP facts](#dhcp-facts)
    - [AWS specifics](#aws-specifics)
  - [VPC and Subnet Facts](#vpc-and-subnet-facts)
    - [VPC Resizing Facts](#vpc-resizing-facts)
    - [IPv6 Subnet Facts](#ipv6-subnet-facts)
  - [Route Tables](#route-tables)
  - [NAT Options](#nat-options)
  
# Designing/Architecting an AWS Network
Notes are taken from Linux Academy's Advanced Networking Specialty course

## VPC Reserved IP Addresses
* AWS reserves the first four IP addresses and the last IP address of every subnet
* Example using subnet 192.168.0.0/24:
  * Subnet address 192.168.0.0: this is the network address and cannot be used
  * Broadcast address (192.168.0.255) is not allowed within AWS and is reserved
  * .1 address is reserved for the subnet's router
  * .2 address is reserved for the DNS server
  * .3 is reserved for future use by AWS

## DHCP Options Sets
* Used to automatically assign IP addresses and other configuration settings to hosts
* DHCP assigns:
  * IP addresses
  * Subnet masks
  * Gateways
  * NTP servers
  * Domain names/DNS servers
  * AWS also assigned NetBIOS settings, which aren't generally used anymore

### DHCP facts
* IP addresses are leased by a client, when released go into a pool of unassigned IPs
* EC2 instances are assigned at least one private IP and can optionally be assigned public IPs as well
  * Elastic IPs server as static IPs in AWS
  * If an elastic IP is assigned to an instance, the DHCP-assigned address is released to the pool

### AWS specifics
* DHCP option sets cannot be changed, but new ones can be created and associated to a VPC
* DHCP options are applied every few hours based on lease renewal
* Default DNS option is AmazonProvidedDNS (.2 resolver)
* Custom DNS servers and names can be specified
  * Up to four DNS servers
* NTP and NetBIOS servers can optionally be specified

## VPC and Subnet Facts
* IPv6 is optional; AWS uses a dual-stack method for VPC resources
* Private subnets with a VGW are called VPN-only subnets
* VPCs are allowed blocks between /16 and /28 (RFC 1918 addresses recommended but not required)
* Publically-routable IPs are optional
* Subnet CIDRs may be the same size as the VPC or smaller

### VPC Resizing Facts
* Additional network masks must be between /16 and /28
* Cannot overlap with existing CIDRs of the VPC, VPN, or Direct Connect
* May be detached from a VPC
* CIDR block size is fixed
* VPCs can support 4 additional CIDRs (5 total)
* CIDR must be smaller than the CIDR range of the primary VPC CIDR

### IPv6 Subnet Facts
* Use a /56 CIDR which cannot be changed
* You can assign a /64 to a subnet
* Five addresses are also reserved per IPv6 subnet
* IPv6 does not use NAT; all addresses are public

## Route Tables
* All subnets must be associated with a route table
  * Either implicitly with main RTB or explicitly through association
* Main RTB cannot be deleted, but custom RTBs can be created
* Default routes
  * IPv4: 0.0.0.0/0
  * IPv6: ::/0
* Local routes allow traffic to flow between subnets
* After adding any gateway, route tables must be updated to point traffic to them
* Longest prefix match applies

## NAT Options
* NAT facilitates private IP to public IP mapping, allowing private hosts with non-routable IPs to communicate with the Internet
* There are two NAT options in AWS: **NAT Gateway** and **NAT instance**
  * EC2 instances in private subnets have no access to the Internet by default, so one of these are chosen
  * NAT instance:
    * Based on Amazon Linux
    * Customer-maintained; limited scalability and HA
  * NAT Gateway:
    * AWS-managed
    * Can be configured for HA and scalability
      * NAT Gateways can be deployed in multiple subnets for HA
      * Can scale bandwidth up to 45 Gbps
    * SGs are not supported; must use NACLs
    * Must use an Elastic IP