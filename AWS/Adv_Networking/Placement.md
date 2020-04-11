- [Placement Groups and Enhanced Networking](#placement-groups-and-enhanced-networking)
  - [Placement Groups](#placement-groups)
    - [Cluster](#cluster)
    - [Partition](#partition)
    - [Spread](#spread)
  - [Enhanced Networking](#enhanced-networking)
    - [Intel Interface/Drivers](#intel-interfacedrivers)
    - [Jumbo Frames](#jumbo-frames)

# Placement Groups and Enhanced Networking
Notes are taken from Linux Academy's Advanced Networking Specialty course

## Placement Groups
* Logical grouping of instances to provide optimized network throughput/performance
* No additional cost
* Three types:
  * Cluster - low latency instance group in a single AZ. Best for performance
  * Partition - spreads instances across logical partitions, ensuring the logical partitions do not share underlying hardware
    * Multiple instances can run on the same underlying hardware, but this is designed so a host can be lost and the application can still run
  * Spread - spreads instances across underlying hardware. Designed for HA
    * One instance per underlying host

### Cluster
* Recommended for apps that require low latency and high throughput
* Cannot span multiple AZs
* As a best practice, instances should all be deployed together; adding one later could cause undesired placement
* It's best to stop/start all instances in a cluster to ensure optimal placement
* Instances can get up to 10 Gbps with an aggregate of 25 Gbps
* Use instances with enhanced networking for best latency and PPS
* Mixed instance types are supported, but it's recommended the same type is chosen
* Network traffic to the Internet/DX is limited to 5 Gbps

### Partition
* Spread across logical partitions (groups of instances on hardware)
* Instances in one partition do not share the same underlying hardware as instances in other partitions
* Spread distributed workloads across distinct hardware
* EC2 attempts to distribute instances equally across the number of specified partitions
* Seven partitions per AZ; can span multiple AZs
* Only available via API

### Spread
* Each EC2 instance has its own underlying hardware
* Recommended for apps with a small number of critical instances
* Reduce the risk of simultaneous failure
* Not supported on dedicated instances
* Up to 7 running instances per AZ

## Enhanced Networking
* Enhanced Networking Adapter drivers provide fast packet processing 
* Cannot be managed from the AWS console; must be configured using AWS CLI or Powershell Tools
* Available bandwidth is based on instance type
* Amazon Linux 2 has enhanced networking enabled by default on supported instance types
* Supported on HVM, but not PV instances
* Up to 25 Gbps of bandwidth

### Intel Interface/Drivers
* Intel 82599 Virtual Function interface
* Use `aws ec2 describe-instance-attribute` to verify if SR-IOV support is enabled
* **Single-Root Input/Output Virtualization (SR-IOV)** offers higher I/O performance and lower CPU util. than traditional virtualized interfaces
  * The PCI adapter is shared, but looks like multiple virtual network interfaces to the hypervisor

### Jumbo Frames
* Ethernet frames with payloads larger than 1500 bytes
* 9001 MTU is the standard jumbo frame size
* Can be used within a VPC, but is restricted to 1500 when leaving the VPC
* Supported between VPC peers