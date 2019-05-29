## Network Protocol Basics

### Internet Protocol (IP)
* Layer 3 protocol
* Carries upper-layer protocols such as UDP and TCP
	* Encapsulates these protocols and sends them across a network
* A router can make forwarding decisions based on layer 3 addresses

### Internet Control Message Protocol (ICMP)
* Common usage is ping (echo request/reply)
	* Determines if a network device can reach another network device
	* If you can't reach the Internet, can you reach your next hop router?

### User Datagram Protocol (UDP)
* Considered an unreliable, connectionless layer 4 protocol
* Does not receive ACKs when it transmits segments
* Userful for applications such as VoIP and live streams

### Transmission Control Protocol (TCP)
* Considered a reliable, connection-oriented layer 4 protocol
* Recieves ACKs for the segements it transmits
* Many applications need a reliable transport (HTTP/SSH)
* Uses a three-way handshake to set up a connection (SYN/SYN-ACK/ACK)
* After a session is set up, you can send varying amounts of data before receiving an ACK 
	* This is called **TCP Windowing** or **TCP Sliding Window**
	* On a reliable network, it's more efficient to send more data before expecting an ACK
	* TCP senders will scale the window size (number of unACKed segments) after each ACK received from the receiver
	* If an ACK is missed, the sender will drop the window back down and grow more cautiously

### Important Protocols

#### Domain Name System (DNS)
* Primary goal is to covert fully-qualifed domain names (FQDNs) to IP addresses
* Users can remember FQDNs well, but not IP addresses
* DNS Hierarchy:
	* Root - this is usually invisible to the user, but comes as a dot (.) at the end of an FQDN
	* Top-level domain (TLD) - (.com, .net, .gov, etc.)
	* Second-level domains - (amazon, google, facebook, etc.)
	* Sub-domain - (sub-domain of a parent, for example, **mail**.google.com)
	* Host - the host where the name resides
* Common DNS Records:

Record Type | Description
--- | ---
A | An **address record** is used to map a hostname to an IPv4 address
AAAA | An **IPv6 address record** is used to map a hostname to an IPv6 address
CNAME | A **canonical name** record is an alias of an existing record. This allows multiple DNS records to map to the same IP address
MX | A **mail exchange record** maps a domain name to an e-mail server for that domain
PTR | A **pointer record** points to a CNAME. It is commonly used when performing a reverse DNS lookup, which is a process used to determine what domain name is associated with a known IP address
SOA | a **start of authority** record provides authoritative information about a DNS zone. Ex. e-mail contact for a zone's admin, zone's primary name server, and refresh timers