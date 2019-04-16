# Phase 2 
**Sources:**
* [ISAKMP - RFC 2408](https://tools.ietf.org/html/rfc2408)
* [IKEv1 - RFC 2409](https://tools.ietf.org/html/rfc2409)

## ISAKMP Header
```
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~             ISAKMP Header with XCHG of Main Mode,             ~
      ~                  and Next Payload of ISA_SA                   ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !       0       !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !                  Domain of Interpretation                     !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !                          Situation                            !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !       0       !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !  Proposal #1  ! PROTO_ISAKMP  ! SPI size = 0  | # Transforms  !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !    ISA_TRANS  !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !  Transform #1 !  KEY_OAKLEY   |          RESERVED2            !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~                   prefered SA attributes                      ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !       0       !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !  Transform #2 !  KEY_OAKLEY   |          RESERVED2            !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~                   alternate SA attributes                     ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
**RFC 2408**

## Packet Exchange
```
     Initiator                        Responder
       -----------                      -----------
        HDR*, HASH(1), SA, Ni
          [, KE ] [, IDci, IDcr ] -->
                                  <--    HDR*, HASH(2), SA, Nr
                                               [, KE ] [, IDci, IDcr ]
        HDR*, HASH(3)             -->
```

## Message 1/2 Explosion (excludes KE payload)
```
!  Proposal #1  ! PROTO_IPSEC_AH! SPI size = 4  | # Transforms  !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~                        SPI (4 octets)                         ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !    ISA_TRANS  !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !  Transform #1 !     AH_SHA    |          RESERVED2            !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !                       other SA attributes                     !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !       0       !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !  Transform #2 !     AH_MD5    |          RESERVED2            !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !                       other SA attributes                     !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !    ISA_ID     !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~                            nonce                              ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !    ISA_ID     !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~              ID of source for which ISAKMP is a client        ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !      0        !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~           ID of destination for which ISAKMP is a client      ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
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
```
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~          ISAKMP Header with XCHG of Quick Mode,               ~
      ~   Next Payload of ISA_HASH and the encryption bit set         ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !       0       !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~                         hash data                             ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
**RFC 2409**

* Message 3
    * Verifies the liveliness of the initiator
    * Responder needs a means to know the initiator received its only QM message and processed it
    * This also avoids a limited denial of service

