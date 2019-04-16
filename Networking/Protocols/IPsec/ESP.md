# Encapsulating Security Payload
[RFC 4303](https://tools.ietf.org/html/rfc4303)
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ----
|               Security Parameters Index (SPI)                 | ^Int.
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ |Cov-
|                      Sequence Number                          | |ered
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ | ----
|                    Payload Data* (variable)                   | |   ^
~                                                               ~ |   |
|                                                               | |Conf.
+               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ |Cov-
|               |     Padding (0-255 bytes)                     | |ered*
+-+-+-+-+-+-+-+-+               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ |   |
|                               |  Pad Length   | Next Header   | v   v
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ------
|         Integrity Check Value-ICV   (variable)                |
~                                                               ~
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
Source: RFC 4303

**Security Parameters Index (SPI)**
* Uniquely identifies security association for ESP datagram
    * Usually chosen by destination system upon establishment

**Sequence Number**
* Field contains monotonically increasing counter value
    * Mandatory
    * Present in sender’s packet even if receiver does not have anti-replay functions enabled
    * Transmitted sequence number may never cycle; new SA is formed prior to the 2^32nd packet on the SA

**Payload Data**
* Data described by the next header field
    * Mandatory; variable length
    * Encryption algorithm IV will be present here if necessary
        * If algorithm requires explicit per-packet sync, it must indicate length, structure, and location of this data

**Padding**
* May be required to ensure cipher text terminates on a 4-byte boundary
    * Pad length and next header field must be right-aligned within a 4-byte word

**Pad length**
* 8 bits; specifies size of the padding field in bytes

**Next header**
* IPv4/v6 number describing format of payload data

**Authentication data***
* Contains Integrity Check Value (ICV) computed over ESP packet.
    * Length specified by authentication function selected
    * Optional; included only if authentication service is selected
    * Authentication algorithm must specify length of ICV and comparison rules

