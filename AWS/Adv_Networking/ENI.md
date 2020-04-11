- [Networking with ENIs](#networking-with-enis)
  - [Elastic IP Addresses](#elastic-ip-addresses)
  - [Elastic Network Interfaces](#elastic-network-interfaces)
    - [Use cases for multiple ENIs per instance](#use-cases-for-multiple-enis-per-instance)
    - [Elastic Network Adapters (ENAs)](#elastic-network-adapters-enas)
  - [Best Practices](#best-practices)

# Networking with ENIs
Notes are taken from Linux Academy's Advanced Networking Specialty course

## Elastic IP Addresses
* When an instance is launched, IP addresses can be assigned statically, via DHCP, or via an Elastic IP.
* Elastic IPs replace the DHCP-assigned IP address when attached to an instance
* When removed from a primary interface, a new DHCP lease will be requested within a few minutes
  * Does not apply to secondary interfaces
* Essentially static public IPv4 addresses
* A small fee is incurred if EIPs are left unusued
* If deleted, EIPs can be recovered if not already assigned to another AWS account

## Elastic Network Interfaces
* Attributes:
  * Private IPv4 address assigned by DHCP
  * One or more secondary private IPv4 addresses
  * One Elastic IP per private IP
  * Public IPv4/IPv6 addresses
  * Security group assignments
  * MAC address
  * Source/dest. check flag
  * Description

* ENIs can be attached and detached from an instance, except for eth0 (primary)
* Number of IPs per interface varies based on instance type

### Use cases for multiple ENIs per instance
* Network/security appliances
* Dual-homed instances with workloads that span subnets
* Failover; move an ENI to a new instance from a broken instance
* Create a management network

### Elastic Network Adapters (ENAs)
* Custom interface used to optimize network performance on some instance types

## Best Practices
* ENI can be attached to an instance when launching, stopped, or running
  * Secondary ENIs can be detached when running or stopped
* Amazon Linux/Windows servers will automatically configure the private IPv4 address and gateway on the OS when dual-homed
* Warm/hot attachments of additional interfaces may require manual configuration of the interface depending on OS
* Adding additional ENIs will **not** aggregate bandwidth
* Dual-homed ENIs attached in the same subnet may cause asymmetric routing; use a secondary private IP on eth0 instead