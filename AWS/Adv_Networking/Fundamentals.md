- [General Networking Fundamentals](#general-networking-fundamentals)
  - [Need to Know](#need-to-know)
  - [OSI Model](#osi-model)
  - [IPv4](#ipv4)
    - [Fundamentals](#fundamentals)
    - [Decimal to Binary Conversion](#decimal-to-binary-conversion)
    - [IP Addresses and Subnet Masks](#ip-addresses-and-subnet-masks)
    - [Classful vs. Classless](#classful-vs-classless)
    - [RFC 1918 (Private IP Addresses)](#rfc-1918-private-ip-addresses)
    - [Subnetting](#subnetting)
  - [IPv6](#ipv6)
    - [Overview](#overview)
    - [IPv6 on AWS](#ipv6-on-aws)

# General Networking Fundamentals
Notes are taken from Linux Academy's Advanced Networking Specialty course

## Need to Know
* General networking concepts
* OSI model
* IPv4
  * IP addressing
  * Subnetting
* IPv6 basics

## OSI Model
1. Physical - cables, hubs, repeaters
2. Data Link - L2 switches, MAC addresses, bridges
3. Network - routers, IP, ICMP
4. Transport - TCP/UDP
5. Session - L2TP, NetBIOS, AppleTalk
6. Presentation - JPEG, GIF, MPEG
7. Application - HTTP, SMTP, DNS, etc.

## IPv4

### Fundamentals
* Used in conjunction with routing for communication between devices on different networks
* Layer 3 protocol
* 32-bit numbers
* Split into 4 bytes and separated by a period (.)
  * Ex. 192.168.16.212
* Made up of **network ID** and **host ID**
* **Subnet masks** determine the amount of hosts a subnet can have
* Subnets have reserved addresses that cannot be assigned: the **network/subnet address** and the **broadcast address**

### Decimal to Binary Conversion
* 1 byte = 8 bits
* Each bit in an octet is a power of two
  * Ex. 192 = 1 1 0 0 0 0 0 0
* Powers of two: 128  64  32  16  8  4  2  1

### IP Addresses and Subnet Masks
* Subnet masks determine the portion of the IP address that is the network ID
  * Ex. 11111111.11111111.11111111.00000000 = 255.255.255.0
  * With this example, the first three octets are the network ID. The last octet is reserved for host IDs.
  * Network bits are represented by ones and host bits by zeroes
* 192.168.112.0 with a subnet mask of 255.255.255.0 would have a usable host range of 192.168.112.1 - 192.168.112.254
* Using subnet masks, we can select how many subnets we can create and/or how many hosts we can have per subnet
* Number of subnets can be determined by 2^n where n=the number of borrowed bits
* Number of hosts = 2^(32-n)-2 where n=the number of bits in the subnet mask

### Classful vs. Classless
* Classful addressing defined network adresses in 8-bit groups
  * A: 8 network ID bits, 128 networks, 16,777,216 addresses per network. 0.0.0.0-127.255.255.255
  * B: 16 network ID bits, 16,382 networks, 65,535 addresses per network. 128.0.0.0-191.255.255.255
  * C: 24 network ID bits, 2,097,152 networks, 256 addresses per network. 192.0.0.0-223.255.255.255
  * D: Multicast range 224.0.0.0-239.255.255.255
  * E: Reserved range 240.0.0.0-255.255.255.255

* Classless networks are also referred to as **CIDR (Classless Inter-Domain Routing)**
* CIDR notation uses a network address (AKA routing prefix) with a suffix that indicates the number of bits for the prefix
  * ex. 192.168.1.0/24
* **Variable-length subnet masking** identifies the number of bits representing the network

### RFC 1918 (Private IP Addresses)
* Non-routable; use NAT to facilitate Internet access
* A: 10.0.0.0/8; 10.0.0.0-10.255.255.255
* B: 172.16.0.0/12; 172.16.0.0-172.31.255.255
* C: 192.168.0.0/16; 192.168.0.0-192.168.255.255
* 169.254.0.0/16 reserved for link-local IP addresses

### Subnetting
* The ability to take a larger network and divide into smaller networks
* Example: You are assigned a network address of 192.168.5.0/24. Subnet this address to create networks capable of a capacity of 20 hosts.
  * /27 mask = 3 borrowed bits = 16 networks with 30 hosts per network

## IPv6

### Overview
* Developed to deal with IPv4 exhaustion
* All addresses are public (no NAT)
* 128-bit addresses, or 2^128 addresses total. 
* Not compatible with IPv4, but you can use dual-stacking
* Hexadecimal addressing:
  * Eight groups of for hexadecimal numbers separated by colons
  * Can be abbreviated by remove one or more leading zeroes from hexadecimal groups
  * Consecutive sections of zeroes are replaced with ::
  * :: can only be used once in an address
  * ::1 is the IPv6 loopback address
  * Link-local:
    * fe80::/10
  * Example a full address and valid abbreviations
    * 2101:abcd:0000:0000:0000:0000:3257:9615
    * 2101:abcd:0:0:0:0:3257:9615
    * 2101:abcd::3257:9615

### IPv6 on AWS
* IPv6 must be added to your VPC; it's not enabled by default
* AWS assigns a /56 IPv6 CIDR when enabled
* Instances must be dual-stacked
* Services that support IPv6:
  * VPC and related services 
  * Route 53
  * CloudFront
  * S3
  * DX
  * WAF
  * ELB