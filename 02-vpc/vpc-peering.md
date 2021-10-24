# VPC Peering

- VPC peering lets us create a private and encrypted network link between two VPCs
- One peering connections link two and only two VPCs
- VPC peering can be created in the same region or cross-region. Peered VPCs can be in the same account or cross-account
- When a VPC peer is created, we can enable the option to have public hostnames being resolved to private IPs inside the VPCs. Same DNS can be used to locate public services and resolve their hostname to their private IP
- If the VPCs are in the same region, they can reference each other by using security group IDs. In different regions we have to reference IP addresses and IP ranges
- VPC peering does not support transitive peering: if VPC_A and VPC_B is peering, but also VPC_B and VPC_C are peering, this does not mean that VPC_A is peered with VPC_C
- When a peering is created, a logical gateway is created in both VPC. This means we have to configure routing
- The IP ranges of the VPC CIDRs can not overlap!
- Any data between VPC peers is encrypted. If the peered VPCs are in different regions, the data between peers will go over AWS global transit network

## VPC Peering Considerations

- Security Groups are tied to a VPC in a region. They are attached to ENIs and the can reference each other and themselves
- When we create a peer between VPCs in the same region, SGs from one VPC can reference SGs from other the VPC. If the VPCs are in different account, but same region, we can reference a SG by `accountID/SG ID`
- In case of cross-region peering we can not reference SGs from the other peered VPC
- DNS in peered VPCs:
    - If we are resolving public IPv4 DNS inside a VPC, it will resolve to the private IPv4
    - Outside of the VPC, the public EIP v4 address will resolve to a public or elastic IPv4 address
    - DNS resolution supports:
        - Requester DNS resolution
        - Accepter DNS resolution