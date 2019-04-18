# BGP
Sources:
* [RFC 4271](https://tools.ietf.org/html/rfc4271)
* [BGP Packet Capture](https://www.cloudshark.org/captures/005a2e2381cd)

## Basics
* Basic BGP
    * Exterior gateway protocol 
        * Exchange routing/reachability info via autonomous systems (AS) on the Internet
        * Often classified as a path vector protocol but sometimes also called a distance-vector routing protocol
        * Routing decisions based on:
            * Paths
            * Rule sets
            * Network policies
            * BGP can be used to route within an AS
            * Called Interior Border Gateway Protocol, Internal BGP, or iBGP
        * Also used to route across the Internet
            * Called Exterior Border Gateway Protocol, External BGP, or EBGP

## Traffic classification
* Egress/Ingress Traffic
    * Egress - packets leaving your AS
            * Route availability (what others send you)
            * Route acceptance (what you accept from others)
            * Policy and tuning (What you do with routes from others)
            * Peering and transit guidelines
    * Ingress - packets entering your AS
            * What information your send and whom
            * Based on others policy (what they accept from you and what they do with it)

## Operation
* BGP Operation
    * BGP neighbors are called peers
            * Established by manual configuration between routers; utilizes TCP port 179
            * Keep-alives are sent every 60 seconds by a BGP speaker
            * iBGP - routing within same AS
        * eBGP - routing between different ASes
        * Routers on the boundary of one AS exchanging info with another AS are called border or edge routers
            * Typically connected directly
        * iBGP peers can be interconnected through other intermediate routers
        * eBGP also allows for VPN tunneling
            * Allows for two remote sites to exchange information securely
        * Differences between eBGP and iBGP
            * New routes learned by eBGP peers are propagated to all other eBGP and iBGP peers (if transit mode is enabled)
            * If new routes are learned on iBGP peering, it is only re-advertised to eBGP peers

## Messages/Tables
* BGP Packets and Tables
    * Packets
        * OPEN: starts session
            * Contains BGP version number, AS number, Hold Down Time value, and BGP identifier
            * Keepalive: makes sure the neighbor is still alive
                * Send r_u_there and if r_u_ack is received the hold time is reset
            * Update: network reachability exchanges (if a network goes down or up)
            * Notifications: something bad happened (closes session)
                * Error on connection
                * Packet loss
                * Wrong ASN
            * Route Refresh: request for a new copy of the peer's full route table
        * Tables
            * Neighbor table: connected BGP neighbors
            * BGP table: A list of all BGP routes received
            * Routing table: Preferred routes or best path value

## Routing Information Base (RIB)
* Adj-RIBs-In: stores routing information learned from inbound UPDATE messages. Represents routes that are available as input to Decision Process
* Loc-RIB: contains local routing information the BGP speaker selected by applying its local policies to the routing information contained in Adj-RIBs-In.
    * Will be used by local BGP speaker
        * Next hop must be resolvable via local BGP speaker’s routing table
* Adj-RIBs-Out: stores information local BGP speaker selected for advertisement to its peers.
    * Carried in local BGP speaker’s UPDATE messages

## Messages In-Depth
* Header
    * Marker - 16-byte field included for compatibility. Must be set to all ones.
    * Length: 2-byte field indicates total length of message, including header
        * at least 19, no greater than 4096
    * Type: 1-byte type code of message
        1. OPEN
        2. UPDATE
        3. NOTIFICATION
        4. KEEPALIVE

* OPEN message
```
       0                   1                   2                   3
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+
       |    Version    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |     My Autonomous System      |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |           Hold Time           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                         BGP Identifier                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       | Opt Parm Len  |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                                                               |
       |             Optional Parameters (variable)                    |
       |                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
**Source: RFC 4271**
    * Version: 1-byte field indicates protocol version number. Current version is 4.
    * My Autonomous System: 2-byte field indicates ASN of sender
    * Hold Time: 2-byte field indicates number of seconds sender proposes for value of Hold Timer. 
        * Upon receiving this, the BGP speaker must calculate the value of hold timer by using the smaller of configured Hold Time and Hold Time received in the OPEN message.
        * Must be either zero or at least three seconds. 
    * BGP identifier: 4-byte field indicating the BGP identifier of the sender
        * BGP speaker sets this value to an IP address that is assigned to the BGP speaker. Determined on startup and the same for all local interfaces.
    * Optional Parameters Length: 1-byte field indicating length of optional parameters field.
    * Optional Parameters: List of optional parameters supported by sender

* UPDATE Message
```
      +-----------------------------------------------------+
      |   Withdrawn Routes Length (2 octets)                |
      +-----------------------------------------------------+
      |   Withdrawn Routes (variable)                       |
      +-----------------------------------------------------+
      |   Total Path Attribute Length (2 octets)            |
      +-----------------------------------------------------+
      |   Path Attributes (variable)                        |
      +-----------------------------------------------------+
      |   Network Layer Reachability Information (variable) |
      +-----------------------------------------------------+
```
**Source: RFC 4271**
    * Withdrawn Routes Length: 2-byte field indicating total length of Withdrawn routes field. Allows length of NLRI field to be determined
    * Withdrawn Routes: Variable-length field that contains list of IP address prefixes being withdrawn from service
    * Total Path Attribute Length: 2-byte field indicates the total length of the Path Attributes field in bytes. 
        * Value of zero indicates NLRI and Path Attribute field will be present in UPDATE message
    * Path Attributes: Variable-length sequence of path attributes
