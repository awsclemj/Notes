- [Protocol Reference Models](#protocol-reference-models)
  - [OSI Model](#osi-model)
  - [TCP/IP Model (AKA DoD Model)](#tcpip-model-aka-dod-model)

## Protocol Reference Models
![Model Comparison](https://www.inetdaemon.com/img/network_models.png "Model Comparison")
* Source: Inetdaemon

### OSI Model
* Open Systems Interconnect Model defined by the ISO
* A generic, seven-layer representation of networks and their associated protocols
* The layers:
1. Physical - physical network medium
2. Data Link - physical addressing (MAC)
3. Network - logical addressing (IP)
4. Transport - transmission protocol (TCP/UDP)
5. Session - setting up, maintaining, tearing down sessions (SIP)
6. Presentation - how data is represented on the network (ASCII)
7. Application - a network service that allows end-user applications to interact with data (HTTP/DNS/DHCP)

### TCP/IP Model (AKA DoD Model)
* Originally defined by the U.S. Department of Defense
* Can be mapped directly to one or more layers of the OSI model
* Layers:
1. Network Access/Network Interface/Link - encompasses layers 1 and 2 of OSI
2. Internet - corresponds to layer 3 of OSI
3. Transport - corresponds to layer 4 of OSI
4. Application - comprehensive representation of layers 5-7 of OSI