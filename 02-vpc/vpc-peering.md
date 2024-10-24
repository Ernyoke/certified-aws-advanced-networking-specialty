# VPC Peering

- VPC peering lets us create a private and encrypted network link between two VPCs
- One peering connections link two and only two VPCs
- VPC peering can be created in the same region or cross-region. Peered VPCs can be in the same account or cross-account
- When a VPC peer is created, we can enable the option to have public host names being resolved to private IPs inside the VPCs. Same DNS can be used to locate public services and resolve their hostname to their private IP
- If the VPCs are in the same region, they can reference each other's security group IDs. In different regions we have to reference IP addresses and IP ranges
- VPC peering does not support transitive peering: if VPC_A and VPC_B is peering, but also VPC_B and VPC_C are peering, this does not mean that VPC_A is peered with VPC_C
- When a peering is created, a logical gateway is created in both VPC. This means we have to configure routing
- The IP ranges of the VPC CIDRs can not overlap!
- Any data between VPC peers is encrypted. If the peered VPCs are in different regions, the data between peers will go over AWS global transit network

## VPC Peering Considerations

- Security Groups are tied to a VPC in a region. They are attached to ENIs and the can reference each other and themselves
- When we create a peer between VPCs in the same region, SGs from one VPC can reference SGs from other the VPC. If the VPCs are in different account, but same region, we can reference a SG by `accountID/SG ID`
- In case of cross-region peering we can not reference SGs from the other peered VPC
- DNS in peered VPCs:
    - Reminder, in case of a VPC (no VPC peering!):
        - If we are resolving public IPv4 DNS from inside a VPC, it will resolve to the corresponding private IPv4 address of that DNS
        - Outside of the VPC, the public EIP v4 address will resolve to a public or Elastic IPv4 address
    - In case of VPC peering, we can have both options for DNS resolution depending on what setting we choose:
        - **Requester DNS resolution**
        - **Accepter DNS resolution**
    - These settings are independent and they define if one VPC can resolve public DNS to private IP addresses for the other VPC
    - Both VPCs must enable **DNS hostnames** and **DNS resolution** for these settings to work

## VPC Peering Unsupported Configurations

- The following configurations are not supported by VPC peering:
    - Overlapping CIDR blocks
    - Transitive peering: if A is peered with B and B is peered with C, this does not mean that A and C are also peered
    - Edge routing: 
        - Edge routing from through a peered VPC over Site2Site VPN or DX is not supported
        - Edge routing from through a peered VPC with an IGW is also not supported. If VPC_B has an Internet Gateway and the peered VPC_A try to reach the internet using VPC_B, the connectivity wont work
- Handle overlapping CIDR blocks:
    - Split routing in VPC per subnets with different route tables
    - Use more specific routes (longest prefix wins) in case of overlaps