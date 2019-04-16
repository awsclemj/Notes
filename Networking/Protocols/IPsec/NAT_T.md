# NAT-Traversal

Source: How Does NAT-T work with IPSec? | VPN | Cisco Support Community](https://supportforums.cisco.com/document/64281/how-does-nat-t-work-ipsec)

## Overview
* ESP
    * IP protocol, therefore does not have TCP/UDP port numbers in its header
* Port Address Translation (PAT)
    * Used to provide many hosts a single publicly-routable IP address. 
    * Builds database
        * Binds local IP to publicly-routable IP using a port number
    * Packets sourced from inside hosts have IP header modified
        * RFC 1918 address changed to public IP and a new port
        * Return traffic is untranslated in the same manner

## Problems
* ESP cannot pass through PAT devices because it does not use port numbers
    * There is no port to change in the ESP packet
        * Database binding can’t be completed
    * Return traffic will not be able to be untranslated

## How does NAT-T help?
* Performs two tasks:
    * Detects if both endpoints support NAT-T
        * ISAKMP MM message 1 and 2
    * NAT-Discovery (detects NAT devices along the path)
        * ISAKMP MM packets 3 and 4
        * Payload hashes original IP address and port (both source and dest.)
        * Receiver recalculates hash and compares it to the one it receives
            * If the hashes differ, there is a NAT device
* If the NAT device exists:
    * MM messages 5 and 6 change from UDP 500 to UDP 4500
    * All further transmissions, including QM and then ESP, are encapsulated inside UDP 4500
* Both the source and dest. ports are assigned as 4500
    * This gives the PAT database enough information to uniquely bind these packets
    * Successful bidirectional translation
    * Source port will be changed to a random high number, dest. remains 4500
