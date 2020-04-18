- [Hybrid Networking - VPN](#hybrid-networking---vpn)
  - [Overview](#overview)
  - [Client VPN](#client-vpn)
  - [Site-to-Site VPN](#site-to-site-vpn)
    - [VGW Routing Rules](#vgw-routing-rules)
    - [Customer Gateway](#customer-gateway)
    - [VPN High-Availability](#vpn-high-availability)
    - [VPN CloudHub](#vpn-cloudhub)
    - [Transit VPCs](#transit-vpcs)
  - [Transit Gateway Overview](#transit-gateway-overview)

# Hybrid Networking - VPN
Notes are taken from Linux Academy's Advanced Networking Specialty course

## Overview
* Types of VPN:
  * **Site-to-site** - uses IPsec to secure communications between VPC the remote network. AWS isde uses VGW and provides two tunnels for redundancy
    * Features:
      * IKEv1/v2 supported
      * NAT traversal
      * 2 and 4-byte ASNs
      * CW metrics
      * Several encryption options based on need, inclusing AES128/256, SHA2 hashing, and DH groups
      * Custom tunnel options
  * **Client VPN** - client-based VPN service that uses TLS-based OpenVPN to secure communcations between client(s) and the VPC
  * **VPN CloudHub** - If there are multiple remote networks, VGW can function as a hub for site-to-site communication between the networks
  * **Software VPN** - a third-party VPN device running on EC2. Can be found in Marketplace or be FOSS. 
* VPN can be used to foster communications between a VPC and remote onn-prem sites or between two VPCs in the same or different regions
* VPN termination endpoints are required at both sites. The endpoint is responsible for protocols, encapsulation, and encryption/decryption

## Client VPN
* OpenVPN-based. Users must have an OpenVPN-compatible client installed on their workstation.
* Users connect to a CVPN endpoint, which can be associated with one or more **target networks** (subnets) in a VPC
* Users can access peered VPCs, public Internet/resources via IGW, and on-prem resources via VGW/TGW
* Server requires a TLS certificate. This can be a private or self-signed certificate but must be uploaded to ACM
* Supports mutual authentication (client certificate auth) and/or Active Directory auth

## Site-to-Site VPN

### VGW Routing Rules
* Longest prefix match applies
* If propagated routes overlap with a VPC local route, the VPC route is preferred
* Static routes are preferred over propagated routes
* If longest prefix match can't be applied, prioritization is as follows:
  * DX propagated routes
  * Static routes
  * VPN propagated routes

### Customer Gateway
* Configured with static or BGP routing
* Configuration of common devices provided by AWS
* Components of IPsec configuration:
  * IKE version
  * Encryption algorithm
  * Hash algorithm
  * DH group
  * PSK or certificate authentication
  * Tunnel interface IPs: /30 address assigned to each side for inside tunnel IPs
* Traffic must be generated from the CGW for the VPN to establish (AWS configured as responder-only)
* DPD enabled; VPN tunnel will go down if no traffic is detected

### VPN High-Availability
* The VGW is redundant be default. You receive two tunnel endpoints in two separate AZs per VPN connection. However, this does not ensure resiliency on the customer's side
* Redundancy can be achieved by adding a secondary CGW on the customer's side. Multiple CGWs can be paired with the same VGW
* BGP routing is reocmmended in this case because BGP metrics influence route preference and handles failover seamlessly. This can't be easily achieved with static routing.

### VPN CloudHub
* Multiple distinct data centers with different route prefixes and BGP ASNs can have access to the same VGW via VPN. This creates a hub-and-spoke topology with the VGW and customer on-prem sites
* The VGW re-advertises routes learned from one ASN to other unique connected ASNs

### Transit VPCs
* Transit VPCs are created using software VPN appliances and allow for transitive routing between VPCs
* The VPN appliance connects to the customer's VPCs and also to the customer's on-prem
* Customer's on-prem recieves routes to all VPCs and VPCs receive routes to other VPCs and customer's on-prem

## Transit Gateway Overview
* AWS network did not support transitive networking until TGW was introduced. This led to complex solutions such as Transit VPC for edge-to-edge routing
* TGW similifies this by allowing for a centralized managment structure
* Traditional hub-spoke model
  * Attachments for VPCs, VPNs, and DX
* TGW controls how traffic is routed between these elements
* ECMP is supported between on-prem connections, enabling load balancing and aggregate bandwidth
* Limitations:
  * 5 TGWs per account
  * 5 TGW attachments per VPC
  * 5000 total attachments per TGW
  * 1.25 Gbps per VPN connection
  * 50 Gbps bandwidth per VPC
  * 10,000 routes
  * Overlapping CIDRs not supported