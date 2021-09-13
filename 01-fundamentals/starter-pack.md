# Networking Started Pack

## OSI 7-Layer Model

![OSI 7-Layer Model](images/OSI7LayerModel.png)

## Layer 1 - Physical

- Physical mediums can be copper (electrical), Fibre (light) or Wifi (Rf)
- Layer 1 specifications define the transmission and reception of raw bit streams between devices and shared physical mediums
- It defines things like voltage levels, timing, rates, distances, modulation and connectors
- A higher level device can understand the specifications imposed by lower layers (example: a layer 3 device can understand the specifications of layers 1, 2 and 3)
- **Hub**: anything received on one port is retransmitted on every other port (including errors and collisions)
- At layer 1 there are no individual device addresses, all data is processed by all devices
- 2 devices might transmit at once, a collision might occur
- Layer 1 has no media access control
- With layer 1 architectures collisions are very likely to occur, the chances being higher as more devices are added to the network
- Layer 1 is not able to detect when collisions do occur

## Fibre Optic Cables

- Alternative way to transmit data vs copper cables
- Fibre optic cable use a thin glass/plastic core surrounded by protection, the core is the diameter of a human hair
- Fibre optic cable transmit light over the glass/plastic medium
- Fibre optic cables can achieve higher speeds (region of TB/s) and larger distances compared to copper cables
- They are resistant to EMI and less prone to water ingress
- Offers a more consistent experience compared to copper cables
- Fibre diameter is expressed as X/Y: X is the diameter of the core, Y is the diameter fo the cladding (both in microns - 1000 microns in a mm)
- Fibre optic cables: [https://www.rfindustries.com/pdfs/articles/Fiber-Optic-Cable-Types-V2.pdf](https://www.rfindustries.com/pdfs/articles/Fiber-Optic-Cable-Types-V2.pdf)
- Fibre optic cable modes:
    - SINGLEMODE: generally has a yellow jacket and uses lasers, great for long distances and generally high speeds (10GB and above)
    - MULTIMODE: orange/aqua jackets. Has a bigger core, can be used with a wider range of waves of light at the same time, meaning it tends to be faster. It has more distortion and can be used for shorter distances
- Transceivers:
    - SFP Transceiver Module (SFP or MINI GBIC) single form-factor pluggable into networking equipments
    - They generate and send/receive light to/from fibre
    - They are either single or multimode, they are optimized for a single type of cable
    - Specifications: **1000BASE-LX**, **10GBASE-LR**, **100GBASE-LR4**

## Layer 2 - Data Link Layer

- Higher layers build on lower layers, a layer 2 network requires a working layer 1 network to operate on
- Frames: format of sending information in a layer 2 network
- Devices at layer 2 have an uniq address (MAC). MAC addresses are 48 bit addresses, in hex
- A MAC address is not software assigned, it is uniquely assigned by the manufacturer
- A MAC address has 2 parts: the OUI (Organizationally Unique Identifier) and the network interface controller specific part
- Layer 2 frame components:
    - Preamble: indicates the start of the frame
    - Destination/source MAC addresses
        - If the destination address is ALL F's, the frame will be broadcasted to all devices on the network
    - Ether type: which layer 3 protocol puts its data to the frame
    - Payload (46 - 1500 bytes)
    - Frame check sequence: allows the check if the frame was corrupted

![Layer 2 frame](images/Layer2DataLink.png)

- CSMA/CD (Carrier-sense multiple access with Collision Detection): layer 2 checks from any carrier and waits for it until it stops transmitting. When the carrier is not detected any more, layer 2 sends the frame down to layer 1 which will transmit it across the physical medium
    - If a collision is detected, a jam signal is sent by all the devices and a random backup occurs
    - After this backup period the transmitting is retried
- For anything communicating at layer 2 is abstracted from the complexity for layer 1
- **Switch**: it is layer 2 device, works similar to a hub
    - It maintains a MAC address table
    - It can interpret frames and pass the data to the necessary node
    - In case a data frame arrives to a switch:
        - If the switch does not know to which node to transmit according to its MAC address table, the data is transmitted to everyone
        - If it knows where to transmit, the data will be sent to the only node
    - Switches do not forward collisions


