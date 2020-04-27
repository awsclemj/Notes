- [DNS](#dns)
  - [DNS Fundamentals](#dns-fundamentals)
    - [Route 53 Overview](#route-53-overview)
  - [DNS Records](#dns-records)
  - [Routing Policies](#routing-policies)
  - [Hybrid DNS](#hybrid-dns)
    - [AD as DNS Forwarder](#ad-as-dns-forwarder)

# DNS
Notes are taken from Linux Academy's Advanced Networking Specialty course

## DNS Fundamentals
* Recursive DNS lookup (no cache)
  * Client wants to go to www.example.com
  * Asks local resolver for IP address
  * Local resolver does a recursive lookup to several servers
  * DNS root server returns IP for TLD (.com/.net/etc) server
  * TLD server returns IP for www.example.com name server
  * www.example.com name server returns IP for www.example.com
  * Local resolver returns IP address for www.example.com to client

### Route 53 Overview
* AWS's DNS service; used as the default local resolver for VPCs. 
* VPC resolver is the .2 address in your VPC. 
* Performs the following main functions:
  * Domain registration
  * DNS routing
  * Health checking
* **Zones** hold DNS record sets and information like the authoritative name server for a domain
* R53 provides **hosted zones**, which act as containers for your DNS records
* Types of hosted zones:
  * **Public** - for Internet-based traffic
  * **Private** - for VPC-based traffic
* Delegating a subdomain to R53:
  * You can create a hosted zone for subdomains for a parent domains that are hosted elsewhere
  * Create a public hosted zone and update the parent DNS records to point at the R53 NS records
  * Ensure the parent does not add an SOA record for the subdomain

## DNS Records
* There are many types of DNS records, and each record has other data such as **time-to-live (TTL)**
  * Records are cached by the requesting server for the time defined by the TTL
* **NS (Name Server)** - authoritative name server for the domain
* **A (IPv4 address)** - resolves a domain/subdomain name to IPv4 addresses
* **AAAA (IPv6 address)** - resolves hostnames to IPv6 addresses
* **CNAME (canonical name)** - point domain/subdomain names to other domain names. (ex. www.example.com -> example.com)
* **MX (mail exchanger)** - directs email to a server. Always point to a domain name, not IP
* **Alias** - similar to a CNAME, but can be used to resolve alternate names to the zone apex (root)
  * Ex.: www.example.com can use a CNAME to point to example.com
  * Alias can be used to point at an IP address or a service like a load balancer
* **CAA (Certificate Authority Authorization)** - allow domain owners to declare which CAs are allowed to issue certificates for a domain
* **NAPTR (Name Authority Pointer Record)** - typically used in Internet telephony to map servers to user addresses
* **PTR** - used for reverse DNS lookup (IP address to DNS name)
* **SOA (start of authority)** - admin info about the zone
* **SPF (sender policy framework)** - specify a list of hostnames/IPs that mail can originate from
* **SRV (service record)** - allows for specifying the port in addition to the IP address
* **TXT** - text about the server, network, etc.

## Routing Policies
* **Simple** - default policy. Used when a single resource performs a certain function for a domain
* **Weighted** - multiple resources are associated. The resources perform the same function. R53 responds to queries based on weights assigned to the resources.
  * Lower weight = lower amount of traffic
* **Latency-based** - based on lowest network latency for the end user; works best when resources are spread across AZs or regions
* **Failover** - Active/passive failover based on health checks
* **Gelocation** - chooses the route based on geographic location of the user. Uses a database that maps IPs to locations
* **Multivalue** - multiple values are returned in a response to a query; round-robin method
* **Health checks** - R53 checks health of resources; route based on the targets' health

## Hybrid DNS
* The AWS DNS resolver can only respond to queries originating from its VPC. It cannot resolve requests that originate from on-prem locations. 
* **Route 53 Resolver** endpoints use ENIs for inbound and outbound DNS resolver queries between on-prem and VPCs. It can also resolve requests cross-account using **Resource Access Manager** if the resources are in the same region.
* VPCs use the .2 resolver, which then forwards requests to the Route 53 Resolver outbound endpoint ENI in a hub VPC
* On-prem clients have a forwarding rule in their local resolver that points to the R53 Resolver inbound endpoint ENIs
* Prior to R53 Resolver, an EC2-based DNS forwarder would need to be stood up in the VPC to forward requests to the .2 resolver and vice versa to on-prem
  * Disadvantages:
    * Management overhead due to a separate instance needing to be stood up and configured
    * Potential point of failure
    * VPC DHCP options set would need to be updated with address(es) of DNS forwarders

### AD as DNS Forwarder
* AWS supports AD as a non-authoritative DNS forwarder
* Simple AD and Managed Microsoft AD are supported. You can also use your own custom AD instances that you manage. 
  * Major difference here is that actual domain controllers are spun up for Managed MS AD. Simple AD uses Samba. 
    * Managed MS AD enables a larger feature set like trust relationships, Sharepoint, SQL server, etc. 