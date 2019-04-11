# IPv6 Overview
* History
    * Born in 1995 (RFC 1883), later formalized in 1998 (RFC 2460)
    * 2^128 address space vs 2^32
    * Not designed to be interoperable with IPv4
        * Simplified header
        * 8 vs. 12 fields
        * Fixed at 40 bytes instead of variable 20-60 bytes
            * Easier for devices to locate certain parameters within headers
    * IPv6 implementation is no longer optional. IPv4 address space is getting depleted quickly
    * Network devices will not fragment packets with IPv6
        * This caused issues in IPv4
* Benefits
    * Larger address space [340 undecillion (trillion x3) vs. 4 billion in IPv4]
    * Better lookups due to aggregated spaces and fixed header
    * Customizable via complimentary extension fields
    * Better built-in security
    * No need for NAT -- all public IP addresses
        * From our perspective, we have egress-only IGWs to block incoming traffic over IPv6
    * Better L2 to L3 mappings using neighbor discovery
        * Multicast solution -- better than using ARP
    * Enhanced auto-addressing 
        * SLACC - Stateful/Stateless DHCP Addressing
        * Addresses itself based on a prefix and its own MAC address
* Why?
    * Customers looking to deploy v6
    * We now have v6 in VPC, EC2, DX and other services
    * World is adopting -- Verizon at 70%, T-Mobile at 58%, etc.
    * In the next three years IPv4 will be legacy
* Allocation
    * Unspecified :: (default is ::/0)
    * Loopback: ::1/128
    * Unique local address: fc00::/17
    * Link-local: fe80::/10
    * Global unicast: 2000::/3
        * AFRINIC, APNIC, ARIN (2600::/12)
    * Multicast: ff00::/8

