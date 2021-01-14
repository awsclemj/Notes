- [Networking](#networking)
  - [Concepts](#concepts)
    - [Unicast vs. Multicast](#unicast-vs-multicast)
    - [Ephemeral Ports](#ephemeral-ports)
    - [VPC Reserved IP Addresses](#vpc-reserved-ip-addresses)
  - [On-prem to VPC Connectivity](#on-prem-to-vpc-connectivity)
    - [AWS Site-to-Site VPN](#aws-site-to-site-vpn)
    - [Direct Connect](#direct-connect)
    - [CloudHub](#cloudhub)
    - [Software VPN](#software-vpn)
    - [Transit VPC](#transit-vpc)
  - [VPC to VPC Connectivity](#vpc-to-vpc-connectivity)
    - [VPC Peering](#vpc-peering)
    - [PrivateLink](#privatelink)
  - [Internet Gateways](#internet-gateways)
  - [VPC Routing](#vpc-routing)
    - [Route Tables](#route-tables)
    - [Border Gateway Protocol](#border-gateway-protocol)
  - [Enhanced Networking](#enhanced-networking)
    - [Network Adapters](#network-adapters)
    - [Placement Groups](#placement-groups)
  - [Route 53](#route-53)
    - [Routing Policies](#routing-policies)
  - [CloudFront](#cloudfront)
    - [SSL/TLS and SNI](#ssltls-and-sni)
  - [Elastic Load Balancing](#elastic-load-balancing)

# Networking
Notes taken from Linux Academy/ACG SA Pro 2020 course.

## Concepts

### Unicast vs. Multicast
* **Unicast** - 1:1 transaction
* **Multicast** - 1:many transaction

### Ephemeral Ports
* Short-lived transport protocol ports above well-known ports (above 1024)
* Suggested range is 49152 to 65535
  * Linux kernels generally use 32568 - 6100
  * Windows generally begin from 1025
* Remember NACL vs. SG implications

### VPC Reserved IP Addresses
* Example subnet 10.0.0.0/24
  * 10.0.0.0 - network address
  * 10.0.0.1 - VPC router
  * 10.0.0.2 - Amazon-provided DNS server
  * 10.0.0.3 - Amazon future use
  * 10.0.0.255 - VPCs don't support broadcast

## On-prem to VPC Connectivity

### AWS Site-to-Site VPN
* IPsec VPN used to connect to your VPC
* Quick and simple way to establish a secure tunnel connection; redundant link for Direct Connect or another VPN

### Direct Connect
* Dedicated network connection over private links connected to AWS backbone
* Used when a company requires a "big pipe" into AWS
* More predictable network performance vs. public Internet
* You can use VPN over DX for encryption

### CloudHub
* Connect locations in a hub-and-spoke manner using AWS's VGW
* Uses BGP to advertise routes to remote offices connected to the same VGW
* Must use separate BGP ASNs

### Software VPN
* "Do it yourself" VPN installed on EC2 instance (e.g. OpenVPN, OpenSwan)
* Must design your own redundancy
* Appliances available in AWS Marketplace

### Transit VPC
* Common strategy for connecting geographically diverse VPCs/locations to create a global transit network
* Hybrid software and managed solution; uses hub and spoke strategy
* Providers like Cisco, Juniper, etc., have offerings to support transit VPC

## VPC to VPC Connectivity

### VPC Peering
* AWS-provided network connectivity between VPCs
* Uses AWS backbone without touching public Internet
* No transitive peering

### PrivateLink
* AWS-provided network connectivity between VPCs and/or AWS services using an **interface endpoint**
* Use AWS backbone to reach services rather than public Internet

## Internet Gateways
* **Internet Gateway** - horizontally-scalled, highly-redundant. No bandwidth constraints. Performs NAT for instances with Public IPs attached.
* **Egress-Only IGW** - IPv6 address "NAT" GW. Provides outbound Internet access and prevents inbound access. Stateful.
* **NAT instance** - AWS-provided AMI for EC2. Translates traffic from private instances to a single public IP. Doesn't allow inbound connections. 
* **NAT Gateway** - Fully-managed NAT service that replaces NAT instance. Must be in public subnet. Uses an elastic IP. Bandwidth 5 - 45 Gbps.

## VPC Routing

### Route Tables
* VPCs have an implicit router with a main route table
* You can modify the main table or create new ones
* Each route table contains a local CIDR block representing the VPC CIDR
* Highest prefix match

### Border Gateway Protocol
* Popular routing protocol for the Internet
* Propagates information about the network to allow dynamic routing
* Required for DX, optional for VPN
* DX support BGP community tagging to control route scope and preference

## Enhanced Networking

### Network Adapters
* Generally for HPC workloads
* Single root I/O virtualization (SR-IOV)
* May need to install a driver if not using Amazon Linux
* Intel 82599 Interface
  * 10 Gbps
* Elastic Network Adapter
  * 25 Gbps

### Placement Groups
**Clustered**
* Single-AZ low-latency group
* Get the most out of enhanced networking instances
* Finite capacity; launch all-at-once for best results

**Spread**
* Instances spread across underlying hardware
* Reduce risk of failure
* Can span AZs
* Max 7 instances per group per AZ

**Partition**
* Instances grouped into partitions spread across racks
* Reduce risk of correlated hardware failure
* Better for large distributed workloads than spread
* No dedicated host support

## Route 53

### Routing Policies
* **Simple** - destination for the name
* **Failover** - as name implies, performs health checks on a primary backend and fails over if the primary fails health checks
* **Geolocation** - routes clients to an endpoint closest to their continent
* **Geoproximity** - routes clients to the closet region based on location
* **Latency** - routes clients to the endpoint with the lowest latency for them
* **Multivalue answer** - return several IP addresses for a name/basic round robin load balancing 
* **Weighted** - Set up resources with traffic percentage weights; routes based on these percentages

## CloudFront

### SSL/TLS and SNI
* CloudFront support either a default cert or custom cert if you'd like to use a custom domain name
  * Integrates with ACM, supports wildcard certs
* CloudFront must either distribute your certs to a dedicated IP at each edge location or use SNI
* **Server Name Indication (SNI)** - allows CloudFront to serve your HTTPS content anywhere without a fee; clients must suppor this feature
* Dedicated IPs at each edge location costs $600/month per distro
* Security Policies
  * Adjust which SSL/TLS versions are supported by your distro

## Elastic Load Balancing
* Know differences between the three different LBs
* NLB
  * Route based on port number only
  * Connections to backend are persisted for the duration of the TCP/UDP session
* ALB
  * Host and path-based rotuing
  * HTTP header and method-based routing
  * Query string-based routing
  * Source IP-based routing
* Sticky sessions
  * Allow session continuity between client and server with session ID cookie