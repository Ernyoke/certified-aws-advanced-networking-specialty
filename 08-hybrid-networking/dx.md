# Direct Connect

## Concepts

- It is a physical connection into an AWS region
- It can be 1, 10 or 100 Gbps connection
- The connection is between the business premisses => DX Location => AWS Region
- When we order a DX connection, actually we order a port allocation at a DX location. It does not provide a connection of any kind by itself, it is just a physical port. It is up to us to connect to this directly or arrange a connection to be extended via a third party comms provider
- The cost for DX includes a hourly cost + cost for the outbound data transfer. Inbound data transfer is free of charge
- To be taken in consideration:
    - Provisioning time: AWS will take time to allocate a port + arrange connection to the premises (weeks/months)
    - No builtin resilience by default
    - Provides low and consistent latency + high speeds
    - DX can be used to access AWS Private Services (running in VPCs) and AWS Public Services

## Physical Connection Architecture

- A direct connect is a physical port allocated at DX location, this physical port provides either 1, 10 or 100 Gbps connection speed
- Port allocated at the DX location requires the use of single-mode fibre, we can't connect using copper connection
- Physical layer connection standards:
    - 1Gbps connection: 1000BASE-LX (1310nm) Transceiver
    - 10Gbps connection: 10GBASE-LR (1310nm) Transceiver
    - 100Gbps connection: 100GBASE-LR4 Transceiver
- In terms of the configuration for the DX ports:
    - We have to make sure that Auto-Negotiation is disabled
    - We configure the port speed
    - Full-duplex has to be manually set on the network connection
- The router on the DX location should support the BGP protocol and BGP MD5 Authentication
- Optional configurations:
    - MACsec
    - Bidirectional Forwarding Detection (BFD)

## Direct Connect MACsec

- Partially improves the long standing problem with DX: lack of built-in encryption
- MACsec is standard, allows Layer 2 frames to be encrypted. It extends standard Ethernet
- MACsec provides a hop by hop encryption architecture. 2 devices need to be next to each other
- MACsec provides the following higher level features:
    - Confidentiality (strong encryption at L2)
    - Data Integrity (data can not be modified in transit without detection)
    - Data origin authenticity
    - Replay protection
- MACsec does not replace IPSec over DX, MACSec is not end-to-end
- MACsec is designed to allow super high speeds
- Key components:
    - **Secure Channel**: base component, each MACsec participant creates a secure channel used to send traffic to other participant (uni-directional traffic)
    - Secure Channels are assigned an identifier (SCI)
    - **Secure Associations**: communication that occurs at each secure channel, takes places as a series of transient sessions. Each secure channel generally has 1 secure association, exception when this associations are being replaced
    - MACsec modified Ethernet frames by inserting a **16bytes MACsec** tag and also adds a 16 bytes **Integrity Check Value (ICV)**
    - **MACsec Key Agreement** protocol: manages peer discovery, authentication and the generation of encryption keys
    - **Cipher Suite**: how data is encrypted, controls the algorithm, packets per key, key rotation, etc.

    ![NO MACsec](images/DXMACSec.png)

    ![MACsec 101](images/DXMacSec2.png)

- MACsec architecture:

    ![MACsec Architecture](images/DXMACsec3.png)

    ![MACsec Architecture Extended](images/DXMACSec4.png)

- MACsec is not a substitute of IPSEC encryption!

