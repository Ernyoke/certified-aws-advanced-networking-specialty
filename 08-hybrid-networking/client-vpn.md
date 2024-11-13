# Client VPN

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
- Split tunnel is **NOT** enabled by default, it must be enabled by us, otherwise all the data will go through the tunnel