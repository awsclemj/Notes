# Phase 1 In-depth
**Sources:**
* [ISAKMP - RFC 2408](https://tools.ietf.org/html/rfc2408)
* [IKEv1 - RFC 2409](https://tools.ietf.org/html/rfc2409)
## ISAKMP Header
```
                         1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    !                          Initiator                            !
    !                            Cookie                             !
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    !                          Responder                            !
    !                            Cookie                             !
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    !  Next Payload ! MjVer ! MnVer ! Exchange Type !     Flags     !
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    !                          Message ID                           !
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    !                            Length                             !
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
**RFC 2408**

## Packet Exchange
```
   When doing a pre-shared key authentication, Main Mode is defined as
   follows:

              Initiator                        Responder
             ----------                       -----------
              HDR, SA             -->
                                  <--    HDR, SA
              HDR, KE, Ni         -->
                                  <--    HDR, KE, Nr
              HDR*, IDii, HASH_I  -->
                                  <--    HDR*, IDir, HASH_R
```
**RFC 2409**

## SA Payload
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
**RFC 2409**
 
* Message 1
    * An initiator cookie (CKY-I) is calculated and inserted
    * The responder cookie (CKY-R) is zero
    * SA payload, which contains three properties
        * Domain of Interpretation (DOI) states that the message exchange is for IPsec
        * Proposal Payload (PP) contains a proposal number, protocol ID, number of transforms, and the SPI is set to zero
        * Transform payload (TP) has the encryption algorithm(s), hash algorithm(s), DH group(s), lifetime, and authentication method
 
* Message 2
    * Responder calculates the responder cookie (CKY-R) and is inserted
    * Most of the fields are the same, but the responder agrees only to a single proposal/transform pair sent by the initiator

## KE/Nonce Payload
```
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~             ISAKMP Header with XCHG of Main Mode,             ~
      ~                  and Next Payload of ISA_KE                   ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !    ISA_NONCE  !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~   D-H Public Value  (g^xi from initiator g^xr from responder) ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      !       0       !    RESERVED   !        Payload Length         !
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      ~         Ni (from initiator) or  Nr (from responder)           ~
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
**RFC 2409**

* Message 3
    * Key Exchange (KE) payload is used to carry the DH public value 
(Xa) and initiator nonce payload (Ni). 
        * The nonce payload contains a very large random number 
        generated using mathematical techniques

* Message 4
    * Similar message to message 3, but contains the corresponding DH public value (Xb) and nonce payload (Nr) for the responder. 

* Prior to message 5 and 6, there are three session key IDs calculated
    * Notation
        * prf(key, msg) is the keyed pseudo-random function-- often a keyed hash function-- used to generate a deterministic output that
             appears pseudo-random.
        * SKEYID is a string derived from secret material known only to the active players in the exchange.
        * SAi_b is the entire body of the SA payload
        * g^xi and g^xr are the Diffie-Hellman ([DH]) public values
        * g^xy is the Diffie-Hellman shared secret
    * Calculations performed
        * SKEYID = prf (pre-shared-key, Ni | Nr)
        * SKEYID_d is calculated for subsequent IPsec keying material
            * prf (SKEYID, g^xy | CKY-I | CKY-R | 0)
        * SKEYID_a is used for data integrity and authentication
            * prf (SKEYID, SKEYID_d | g^xy | CKY-I | CKY-R | 1)
        * SKEYID_e is used to encrypt the subsequent IKE messages (confidentiality)
            * prf (SKEYID, SKEYID_a | g^xy | CKY-I | CKY-R | 2)

* Message 5
    * All packets are encrypted with SKEYID_e
    * Identity payload (ID_I) contains information about the identity of initiator
    * Hash payload (Hash_I) is used for authentication purposes
        * HASH_I = prf(SKEYID, g^xi | g^xr | CKY-I | CKY-R | SAi_b | IDii_b )

* Message 6
    * Similar to message 5. Has ID payload (ID_R) and hash payload (Hash_R) for the responder
        * HASH_R = prf(SKEYID, g^xr | g^xi | CKY-R | CKY-I | SAi_b | IDir_b )

