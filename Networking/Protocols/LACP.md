# LACP
## What is LAG?
* What is Link Aggregation?
    * Group 1 GbE or 10 GbE interfaces into a "bundle"
    * Overcomes bandwidth limitations
    * Load balances
        * Uses deterministic hash algorithm
    * Automatic failover using LACP
    * NIC bonding, PAgP, 802.3ad MLFR, MLPPP
    * LACP - defined in 802.1AX and IEEE 802.1aq or previous IEEE 802.3ad
        * Supports dynamic bundling by sending LACPDU packets via multicast
        * Can be active/passive
        * Uses 1-8 integer value to identify link/LAG
        * When PDUs are sent out, they also include the system ID so LACP will know which device the PDU is coming from

## Other LAG Notes
### Overview
* What is LAG?
* How it works
* Merits of deploying LAG
* LAG product
* API calls

### What is LAG?
* Bundling several physical interfaces into one logical interface
* Operates at layer 2
    * Specifically the link aggregation sublayer
    * EtherType 0x8809 (Slow protocols)
    * Slow protocols subtype 0x01 (LACP)
    * Uses a standard multicast MAC address for exchanging frames between devices
* Actor/partner
    * An actor is the local interface, and the partner is the remote
* Modes of operation
    * Active: node is initiating LACP PDUs to be exchanged
    * Passive: node will not initiate an LACP PDU exchange
        * They will just receive the packet
    * You can have an active/active, active/passive, but not passive/passive configuration
    * LACP PDU interval
        * Short: sends hello message every 1 second. Times out after 3 seconds
        * Long: 30 seconds — times out after 90 seconds

### Merits of deploying a LAG
* More scalable method of upscaling bandwidth
* Load balanced using flow-based hashing
* Automatic failover/backup using LACP
* LACP: defined in IEEE 802.1AX and IEEE 802.1aq or the previous IEEE 802.3ad
* LACP supports dynamic bundling by sending LACP PDUs via multicast address

### How it’s implemented in AWS
* AWS DX LAG is strictly LACP based
* AWS side is configured as LACP actor
* AWS side would not allow static 802.1ad LAGs without LACP
* All member connections are configured Active

#### Product details
* Available for 1G and 10G connections
* Max. 4 connections per LAG (soft limit)
    * Will increase based on use case
* All DXcons in the LAG should use the same link speed
* DXcons in LAG should terminate on the same endpoint at AWS side
* Can be used for both private/public VIFs
* Existing DX connection can be used to create LAG
* Does not come with complete failover options
    * Customer would have to provision another DXcon or DX LAG for redundancy which would land on a different AWS endpoint
* Entire LAG will be down if designated minimum connections are not in available state

#### API calls
* create-lag: creates new link aggregation group (LAG) with specified number of bundled physical connections
* update-lag: updates name of number of minimum links associated with lag

### LAG Guidelines
* All the ports must have same:
    * Speed
    * Full duplex
    * MTU
* Member links do not need to be on contiguous ports
* Existing DX ports can be converted to LAG bundles
* Equal link priorities
* Dynamic LACP, not static
