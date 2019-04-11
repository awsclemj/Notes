# IPsec
* Basic IPsec
    * IPsec communications work by authenticating and encrypting each IP packet of a session
    * Protocols for establishing mutual authentication/negotiation of cryptographic keys
    * Operates at the Internet Layer of the IP suite
        * Other security systems use transport (TLS) and application (SSH)
        * All traffic communicated over the network is therefore encrypted
    * Terms
        * Authentication Header (AH) - connectionless data integrity and authentication of origin for IP datagrams; protection against replay attacks
        * Encapsulating Security Payloads (ESP) - confidentiality, authentication of origin, connectionless integrity, anti-replay, and limited traffic-flow confidentiality
        * Security Associations (SA) - provide the bundle of algorithms and data to provide the parameters for AH and/or ESP
            * Internet Security Association and Key Management Protocol (ISAKMP) - provides framework for authentication and key exchange by using pre-shared keys, Internet Key Exchange (IKE), Kerberized Internet Negotiation of Keys (KINK), or IPSECKEY DNS records
    * Modes of Operation
        * Transport - Only the IP payload is encrypted/authenticated. If AH is used, the IP address cannot be modified by NAT, as it invalidates the hash. 
        * Tunnel - Entire IP packet is encrypted and authenticated. Encapsulated into a new IP packet with a new IP header; used to create VPNs for network-to-network (linking sites), network-to-host (remote user access), and host-to-host (private chat) communications. 
    * Algorithms
        * HMAC-SHA1/SHA2 for integrity and authenticity
        * 3DES-CBC for confidentiality
        * AES-CBC for confidentiality
        * AES-GCM for confidentiality and authentication

