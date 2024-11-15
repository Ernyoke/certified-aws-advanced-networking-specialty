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
    - **Static VPN**:
        - Uses static network configuration: static routes are added to the route tables AWS side, static networks has to be identified on the VPN connection on-premise side
        - It is simple, it just uses IPSec, works anywhere, having limitation on terms of HA
        - No load balancing and multi connection failover
    - **Dynamic VPN**:
        - Uses BGP protocol, if customer router does not support BGP, we can not use dynamic VPNs
        - BGP allows routing on the fly, allows multiple links to be used at once between the same locations. Allows using HA available architectures
        - With dynamic VPN we still can add static routes to the route tables and/or we can enable *route propagation*
        - *Route propagation*: if enabled means that routes are added ro the Route Table automatically
- Considerations for VPN:
    - Speed Limitation for VPN: *1.25 Gbps*, AWS limitation + there is an encryption overhead
    - There is a speed limitation for all the VPN connection connecting to the same VGW. A VGW can allow at most 1.25 Gbps
    - Latency considerations: inconsistent, traffic goes through the public internet
    - Cost: hourly cost for outgoing traffic
    - Speed of setup: can be setup in hours, VPN is a the quickest to be setup compared to other private connection technologies
    - VPN can be used for Direct Connect backup or they can be used over the Direct Connect for adding a layer of encryption

## Dead Peer Detection (DPD)

- It is a mechanism through which we can detect liveliness of IPSec VPN connection
- DPD is optional setting
- VPN peers can decide during the IKE Phase 1 if they want to use DPD
- If DPD is enabled AWS continuously (every 10 seconds) sends DPD (R-U-THERE) messages to customer gateway (CGW) and waits for the R-U-THERE-ACK
- If there is no response for 3 consecutive requests, then the DPD times out
- By default when DPD occurs the gateways will delete the security associations. During this process the alternate IPSec tunnel can be used (if there is one)
- Default DPD timeout value is 30 sec which can be set higher
- DPD uses UDP port 500 or UDP port 4500 to send DPD messages
- When DPD timeout occurs, the following actions can happen:
    - *Clear* (default action): end the IPSec IKE session, stop the tunnel and clear the routes
    - *Restart*: AWS restarts the IKE phase 1
    - *None*: take no action
- Customer router must support DPD when using dynamic routing with BGP
- VPN tunnel can still be terminated due to inactivity or idle connection. We can set up appropriate idle timeout or setup hosts which sends ICMP ping requests to each other every 5-10 seconds

## Accelerated Site-to-Site VPN

- Performance enhancement for AWS Site-to-Site VPN that uses the AWS global network, the same network used for Global Accelerator and CloudFront
- Using a classic Site-to-Site VPN, the traffic goes through the public internet. In order to avoid this, some companies use a Site-to-Site VPN over Direct Connect. Direct Connect offers more better performance, but at a higher cost. Since DX is not an option for everybody, accelerated Site-to-Site VPN was created to improve performance compared to classic Site-to-Site VPNs
- Accelerated Site-to-Site VPN architecture:

![Accelerated Site-to-Site VPN](images/AcceleratedS2SVPN1.png)

- Acceleration can be enabled when creating a Transit Gateway attachment only! Not compatible with VPNs using virtual gateways (VGW)
- Accelerated Site-to-Site VPN has a fixed accelerator cost fee and a transfer fee

## VPN NAT Traversal (NAT-T)

- AWS VPN supports the VPN termination behind NAT on customer side
- With NAT Traversal we encapsulate our IPSec ESP header with an UDP header. With this we avoid breaking the NAT connectivity in case our customer gateway is behind a NAT
- We must open UDP port 4500 on customer side firewall for NAT-T

## VPN Transitive Routing Scenarios

- We can reach the internet from on-premises if:
    - There is a **NAT Instance** (self managed NAT) in the VPC to which the VGW is connected
- VPC Peering:
    - When using S2S VPN to connect to one VPC which is peered with another VPC, the traffic from on-prem to the peered VPC is not originating from the initial VPC. This means, we cannot reach resources from the peered VPC via S2S VPN
- With Gateway Endpoints:
    - Similarly to peering scenario, since traffic is not originating from the VPC, it cannot flow through the VPC Gateway Endpoint
- VPC Interface Endpoints:
    - VPC Interface endpoint creates an ENI inside the VPC
    - Through this ENI we can reach the AWS services
- Summary:
    - From on-premises via VGW:
        - We cannot access the internet through VPC attached Internet Gateway
        - We cannot access the internet through NAT Gateway in a public subnet
        - We cannot access peered VPC resources through VPC peering connection via AWS VGW
        - We cannot access S3, DynamoDB via VPG Gateway endpoints
        - We can access AWS services via interface endpoints
        - We can access the internat with a managed NAT (NAT instance)
    - From AWS to on-premises via customer gateway:
        - We can access the internet and other network endpoints based on routing rules we setup on the VGW in on-premises network