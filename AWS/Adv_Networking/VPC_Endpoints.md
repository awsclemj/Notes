- [VPC Endpoints and PrivateLink](#vpc-endpoints-and-privatelink)
  - [Overview](#overview)
  - [Gateway Endpoints](#gateway-endpoints)
  - [Interface Endpoints](#interface-endpoints)

# VPC Endpoints and PrivateLink
Notes are taken from Linux Academy's Advanced Networking Specialty course

## Overview
Many AWS services are public by default. VPC Endpoints allow you to privately connect your VPC resources to some public AWS resources. 

* **Interface Endpoints** use a service called PrivateLink, which creates a private ENI in your VPC. Access can be controlled by policies and/or security groups. 
* **Gateway Endpoints** are a target for a specific route in your route table. S3 and DynamoDB use Gateway Endpoints.

## Gateway Endpoints
* Route with a target of a prefix list is added to your route table(s)
* Routes can only be changed by modifying the gateway
* Only support IPv4
* Cross-region is not supported
* VPN/DX/VPC peers cannot access the gateway
* DNS resolution is required

## Interface Endpoints
* Many AWS services and also SaaS solutions on marketplace take advantage of PrivateLink
* Endpoint-specific DNS names are created in a private hosted zone. Private DNS can optionally be enabled for AWS services. 
* Interface endpoints can be accessed from VPN, DX, and VPC peers
* One subnet in an AZ may be used at a time
* Support up to 10 Gbps per AZ; additional capacity can be added based on use
* Only support TCP IPv4 traffic
* Regional construct; no cross-region