- [Amazon VPC Networking](#amazon-vpc-networking)
  - [VPC Essentials](#vpc-essentials)
  - [VPC Networking Basics/Components](#vpc-networking-basicscomponents)
  - [VPC Security Basics](#vpc-security-basics)
    - [Bastion Hosts and NAT Gateways](#bastion-hosts-and-nat-gateways)
  - [HA/Fault Tolerance](#hafault-tolerance)
  - [Load Balancers](#load-balancers)

## Amazon VPC Networking
* Networking in the cloud. Notes taken from AWS Developer Associate course on Linux Academy.

### VPC Essentials
* Enables you to launch AWS resources into a virtual network that you've defined
* Designed to resemble:
    * Private on-prem data centers
    * Private corporate networks
* Important facts:
    * VPCs are housed within a chosen AWS region
    * Can span multiple AZs within a region
        * This in turn allows redundancy, high availability, and fault tolerance
    * AWS Provides a DNS server for your VPC so each instance can have a hostname
        * You can choose to run your own custom DNS server by changing the VPC's DHCP options set
* Benefits:
    * Launch instances in subnets
    * Define custom CIDR ranges
    * Configure separate routes between subnets via route tables
    * Extend to your on-prem with VPN/DX
    * Layered security:
        * Instance-level security groups
        * Subnet-level network ACLs

### VPC Networking Basics/Components
* Internet Gateway
    * Required if you would like to access any VPC resources over the public Internet (attach to VPC)
    * Allows communication between instances in the VPC and the Internet
    * Horizontally-scaled, redundant, with no availability or bandwidth constraints
    * Provides NAT translation for instances with public IP addresses assigned
* Route tables
    * Contains a set of rules (routes) that are used to determine where network traffic is destined
    * Comprised of two components:
        1. Destination: CIDR block range of target
        2. Target: a name identifier of where the data will be routed to
    * Local route there by default; cannot modify this route
* Subnet
    * After creating a VPC, you can create one or more subnets in each AZ. Subnets can't span zones.
    * Must be associated with a route table (main route table by default)
    * Public subnets have a route to the IGW
    * Private subnets don't have a route to the Internet or a route to a NAT instance/gateway

### VPC Security Basics
* NACLs:
    * Firewall on the subnet-level of the VPC
    * Support ALLOW and DENY rules
    * Stateless; return traffic must be allowed through an outbound rule
    * Process rule numbers in order
    * Optional layer of security controlling traffic in/out of your subnets
* Security groups:
    * Similar to ACLs in that they allow/deny traffic
    * However, they operate at the instance level
    * The way these rules work are different than NACLs:
        1. Only support allow rules
        2. Stateful; return traffic requests are allowed regardless of rules
        3. All rules are evaluated before deciding to allow traffic
    
#### Bastion Hosts and NAT Gateways
* Bastion host:
    * Lives in a public subnet, is used as a sort of "gateway" for instances in private subnets
    * Considered a "critical strong point" of the network; all traffic must pass through it first
    * Should have very tight security
* NAT Gateway:
    * Provides instances in a private subnet access to the Internet
    * Prevents hosts located outside of the VPC from initiating connections with instance in private subnets
    * Only allows inbound connections from instances in private subnets

### HA/Fault Tolerance
* This is a main benefit of AWS. It is very easy to create highly-available, fault tolerant environment.
* ELB:
    * EC2 service that automates the process of distributing inbound traffic evenly to instances associated with the ELB
    * Can load balance traffic to multiple EC2 instances across multiple AZs
    * Should be paired with Auto Scaling to enhance HA/fault-tolerance, as well as allow for scalability/elasticity.
    * Can use ELB internally or have it Internet-facing
    * Automatically stops service traffic to unhealthy instances
    * Can help offload processing SSL from backend instances
* Auto Scaling:
    * Service provided by AWS to automate the process of increasing/decreasing the number of instances available for your application
    * Scales based on chosen CloudWatch metrics
    * This enables your application to scale up/down with demand (elasticity)

### Load Balancers
* ALB:
    * Routing decisions are made at layer 7 (HTTP/HTTPS)
    * Supports path-based routing (forward rules based on the URL in the request)
    * Supports host-based routing. Can configure rules for your listener that forwards requests based on the Host field in the HTTP header
    * Support for routing requests to multiple applications on a single EC2 instance
        * You can register the same instance/IP address to the same target group with multiple ports
* NLB:
    * Routing decision made at layer 4 (TCP/UDP)
    * Highly performant; can handle millions of requests per second
    * Forwards requests without modifying headers
    * Dynamic host port mapping
    * Static IP addresses for the load balancer
* Dynamic Port mapping:
    * CLB doesn't support this, but both the ALB and NLB do
    * Consider a single EC2 instance could be running multiple containers with a randomly assigned port
    * These ports are not static; when containers are shutdown/restarted, they could receive new port numbers
    * These are facilitated by Target Groups
        * The TG tracks lists of ports that are accepting traffic on each instance and provides the LB a means to distribute traffic across the ports
    * This makes ALB/NLB ideal for running Docker/ECS workloads