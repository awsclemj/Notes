- [Direct Connect](#direct-connect)
  - [Overview](#overview)
  - [Public/Private VIFs](#publicprivate-vifs)
  - [Link Aggregation Groups (LAGs)](#link-aggregation-groups-lags)
  - [Routing](#routing)
    - [BGP Communities](#bgp-communities)
  - [DX Gateway](#dx-gateway)
  - [HA and Redundancy](#ha-and-redundancy)

# Direct Connect
Notes are taken from Linux Academy's Advanced Networking Specialty course

## Overview
* Connection Types:
  * Dedicated - 1/10 Gbps connections that can support up to 50 VIFs
  * Hosted - Sub-1Gbps (some partners support 1/2/5/10 Gbps) connections that are provisioned by Direct connect partners. Supports a single VIF.
  * Hosted VIFs - Providers provision a hosted VIF instead of a hosted connection. Has the potential for oversubscription
* Network requirements:
  * Single-mode fiber, 1000BASE-LX transceiver for 1 Gbps or 10GBASE-LR for 10 Gbps
  * Auto-neg disabled, port speed and full-duplex manually configured
  * 802.1q VLAN encapsulation is required across the entire connection, including intermediary devices
  * The router used must support BGP and BGP MD5 auth
  * Bidirectional Forwarding Detection (BFD) is supported, but optional
* Other requirements:
  * An LOA-CFA is required for authorizing the connection to the AWS network (provided by AWS when you create a connection)
  * LOA-CFA expires after 90 days; port-hour charges begin after 90 days or when the connection is established
  * Hosted connections are fully provisioned by an AWS Direct Connect partner. You must accept this connection type before it can be used

## Public/Private VIFs
* A VIF is a logical connection between your router and the AWS router facilitated through 802.1Q VLAN tags
* You can have up to 50 VIFs per dedicated connection. You may have public and private VIFs that share this physical connection
* **Public** VIFs are used to communicate with services that reside outside of the VPC (S3, SQS, DynamoDB, etc.)
  * Requirements:
    * Public/private BGP ASN. Private ASNs are supported, but AS-path prepending will not be honored. Public ASNs must be registered with IANA
    * Unused VLAN ID
    * Public IP address for BGP peer
    * MD5 auth key (can be auto-generated)
    * Specify public prefixes you want to advertise to AWS (these must be whitelisted and belong to you)
* **Private** VIFs are used to communicate with resources in VPCs. Connect to either a VGW or DXGW
  * Requirements:
    * Configure VGW/DXGW and specify the Amazon-side ASN
    * Unused VLAN ID
    * VIF name and owner info
    * BGP peer IPs (option to auto-generate link-local IPs in the 169.254.0.0/16 range)
    * MD5 auth key (can be auto-generated)

## Link Aggregation Groups (LAGs)
* Uses **Link Aggregation Control Protocol (LACP)** to aggregate multiple connections at a single DX endpoint
* Multiple connections are treated as a single logical connection
* Allows for aggregation of bandwidth; links are treated as active-active. 
* All connections must share the same bandwidth and terminate on the same DX endpoint.

## Routing
* DX routes are always preferred over VPN routes
* Standard BGP path selection is used otherwise
  * Longest-prefix match
  * Local preference (you can use BGP communities to set local preference on the AWS end)
  * AS-path (not considered best practice anymore due to local pref in the AWS backbone)
  * Routes with equal cost will be load-balanced across connections (ECMP)
* BFD is automatically enabled for each DX VIF, but it must be enabled on your router for it to function
  * Faster route convergence time for path failures

### BGP Communities
* Metadata applied to a route prefix during advertisement. This can influence route selection in a neighbor's network
* The Following BGP communities can be used in AWS:
  * Public VIF scoping (advertised **to** AWS):
    * 7224:9100 - advertise to local AWS region only
    * 7224:9200 - advertise to the local continent
    * 7224:9300 - advertise globally
  * Public VIF scoping (advertised **from** AWS):
    * 7224:8100 - prefixes from local region
    * 7224:8200 - prefixes from local continent
    * No tag - global prefixes
  * Private VIF route manipulation (local preference):
    * 7224:7100 - low preference
    * 7224:7200 - medium preference
    * 7224:7300 - high preference 

## DX Gateway
* Global logical device that allows you to connect multiple VPCs from multiple regions and/or accounts to a single VIF. 
* Simplifies global connectivity by acting as a redundant, distributed peering point to aggregate both VIFs and VPCs
* Connects to a VGW or to TGW in the case of Transit VIF. 
* Limits:
  * A VGW can only be attached to a single DXGW
  * A DXGW can have a maximum of 10 VGWs attached
  * Up to 30 private VIFs per DXGW
  * DXGW and VPCs must be owned by the same payer account
* Routing limitations:
  * Overlapping VPC CIDRs not supported
  * Inter-VPC/Inter-VIF communication not supported
  * No route summarization or manipulation possible
  * Cannot exceed 100 route advertisements from on-prem

## HA and Redundancy
* Possible scenarios:
  * Single data center:
    * Active/active DX connections - multiple customer devices are used for multiple physical connections to a single DX location
    * DX with failover VPN - a single DX is used as the active connection with a VPN connection for failover (cost savings, but less reliable backup connection)
  * Multiple data centers:
    * Two DCs with two DX - one (or more) DX per DC, DX terminates at different locations for redundancy. More connections per DC = more redundancy
    * Two DCs, one with DX and one with VPN - one DC has DX and the other has VPN as a failover