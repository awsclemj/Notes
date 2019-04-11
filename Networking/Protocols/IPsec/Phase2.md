# Phase 2 
## ISAKMP Header
**RFC 2408**

## Packet Exchange

## Message 1/2 Explosion (excludes KE payload)
**RFC 2409**

* Message 1
    * Payload messages are encrypted using SKEYID_e from Phase 1
    * Message ID in ISAKMP header identifies a Quick Mode in progress for a particular ISAKMP SA
        * The ISAKMP SA is identified by the cookies in the ISAKMP header
        * Each Quick Mode instance uses a unique IV, so it is possible to have multiple Quick Modes in progress simultaneously. 
    * Hash payload is used to authenticate peers and provide check for liveliness
    * Proposal payload has ESP/AH, and an SPI value
    * Transform payload contains tunnel or transport mode, DH group, enc/hash, and lifetime value
    * An additional KE payload for PFS (optional)
    * Proxy identities (ID_s) and (ID_d) are the networks to be encrypted between peers

* Message 2
    * Fields are mostly the same, but the responder will only agree on one transform pair sent by initiator
    * The proposal payload contains the SPI of the outgoing SA of the responder rather than the SPI sent by the initiator

* Prior to message 3, two keys are generated on the initiator and responder
    * One is generated for the outbound SA. This value matches the inbound SA on the opposite peer
    * One is also generated for the inbound SA. This value matches the outbound SA on the opposite peer
    * KEYMAT calculation
        * prf (SKEYID_d, g(qm)^xy | protocol | SPI | Ni_b | Nr_b)
        * The different SPIs generated guarantee a different key for each direction

## QM Message 3 (Hash Payload)
**RFC 2409**

* Message 3
    * Verifies the liveliness of the initiator
    * Responder needs a means to know the initiator received its only QM message and processed it
    * This also avoids a limited denial of service

