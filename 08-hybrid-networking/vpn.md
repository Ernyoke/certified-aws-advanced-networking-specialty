# AWS Site-to-Site VPN

- A Site-to-Site VPN is a logical connections between a VPC and an on-premise network running over the public internet. The connection is encrypted using IPSec
- Can be fully HA if it is implemented correctly
- It is quick to provision, it can be provisioned in less than an hour (contrast to DX)
- Components involved in creating a VPN connection:
    - **VPC**
    - **Virtual Private Gateway (VGW)**: it is a gateway object which can be the target of one or more rules in a Route Tables. It can be associated to a single VPC
    - **Customer Gateway (CGW)**: can refer to 2 different things:
        - Often is referred to the logical configuration in AWS
        - Physical on-premises router which the VPN connects to
    - **VPN Connection** itself: the connection linking the VGW from the AWS to the CGW
- Static vs Dynamic VPN:
    - **Dynamic VPN** uses BGP protocol, if customer router does not support BGP, we can not use dynamic VPNs
    - **Static VPN** uses static network configuration: static routes are added to the route tables AWS side, static networks has to be identified on the VPN connection on-premise side. It is simple, it just uses IPSec, works anywhere, having limitation on terms of HA
    - Dynamic VPN uses BGP: allows routing on the fly, allows multiple links to be used at once between the same locations. Allows using HA available architectures.
        - Route propagation: if enabled means that routes are added ro the Route Table automatically
- Considerations for VPN:
    - Speed Limitation for VPN: *1.25 Gbps*, AWS limitation
    - Latency considerations: inconsistent, traffic goes through the public internet
    - Cost: hourly cost for outgoing traffic
    - VPN can be used for Direct Connect backup or they can be used over the Direct Connect for adding a layer of encryption

## Client VPN

- Is a managed implementation of OpenVPN
- Any client device which can use the OpenVPN software is supported
- Architecturally we connect to a Client VPN endpoint which can be associated with one VPC and with one ore more Target Networks
- Client VPN billing is based on network association and hourly fee for usage
- Client VPN setup:
    - We crate a Client VPN Endpoint and associate it with a VPC and one or more subnets from the VPC
    - This association places an ENI into the subnets associated
    - We can only pick one subnet per AZ
- Client VPN can use many different methods of authentication (Cognito User Pool, Federated Identities AWS Directory Service)
- We associate a route table to the Client VPN Endpoint in order to set up routing and connectivity (to internet via NAT Gateways, other VPCs with peering, etc.)
- This route table is pushed to any client which connects to the Client VPN Endpoint
- The default behavior is the Client VPN route table replaces any local routes on the client, meaning the client devices can not access anything locally on their local network without having communication going through the Client VPN Endpoint
- We can use split tunnel VPN, meaning that any routes from the Client VPN Endpoint are added to local client route tables. This solves the problem with the default behavior