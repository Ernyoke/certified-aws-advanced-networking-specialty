# TGW - AWS Transit Gateway

- It is a network transit hub which connects VPCs to each other and to on-premise networks using Site-to-Site VPNs and Direct Connects
- It is designed to reduce the network architecture complexity in AWS
- It is a network gateway object, it is HA and scalable
- Attachments: we create attachments for the TGW to connect to VPCs and on-premise networks. Valid attachments are:
    - VPC attachments
    - Site-to-Site VPN attachments
    - Direct Connect Gateway attachments
- Attachments are configured in each subnet of the connected VPCs
- We can also peer transit gateways across regions and/or across accounts
- We can also attach transit gateways to the DX connections
- Transit Gateway Considerations:
    - Supports **transitive routing**: single transit gateway with multiple attachments using route tables
    - Can be used to create global networks with peering
    - We can share transit gateways using *AWS RAM*
    - Transit Gateways offer less complex architectures compared to VPC peering solutions

## Transit Gateway - Deep Dive

- Transit gateway is a *hub-and-spoke architecture*, it can connect to various types of networking objects within AWS
- Integration with Direct Connect:
    - A **Transit VIF** is required which goes through a DX Gateway
    - The DX Gateway can be attached to the Transit Gateway with a Transit Gateway Attachment
    - 1 DX Gateway can be attached to 3 Transit Gateways
- Transit Gateway has a default route table which is populated from the attachments:
    - For the VPCs we have the CIDR ranges of these VPCs
    - For VPNs we have the routes learned via BGP
    - For DX Gateways with the Transit Gateway Attachment we define the networks within the attachment configured at the DX Gateway side
- The default route table has all the routes which TGW learns about. By default, the route table is used by all of the attachments of the TGW
- We can peer TGWs with other TGWs between regions. We can peer a TGW with up to 50 other TGWs, and these TGWs can also peer with other TGWs
- A TGW by default has one route table
- All attachments use this RT for routing decisions, by default all attachments propagate routes to this route table, exception peering attachments
- All attachments can route to all other attachments by default

## Transit Gateway Peering

- In case of peering attachments routes are not shared, we need to use static routes, similar to VPC peering (when configuring a TGW AWS recommends using unique ASNs for future enhancements for route advertisements)
- Resolution of public DNS to private addressing is not supported over inter-region peers
- Data transfer over peering connection is encrypted and is sent over AWS network
- We can peer up to 50 peering attachments per TGW

## Transit Gateway Isolated Routing

- By default:
    - All attachments are associated with the same route table
    - All attachments propagate to the same route table, all attachments are aware of any other attachments
- Attachments can only be associated with 1 route table, route tables can be associated to many attachments
- Attachments can propagate to many RTs, event to those they are not associated with
- If we would want to isolate networks:
    - We create a route table and we configure all attachments to propagate to the route table
    - We associate the route table with only the attachments we would want to communicate with each other
    - We create another route and associate it to the attachment we don't want to communicate with each other. We configure other attachments to propagate to this route table

## AZ Affinity and Appliance Mode

- AZ Affinity:
    - Transit Gateway tries to keep the traffic in the same originating AZ until it reaches its destination
    - Benefits:
        - No cross-AZ data charges
        - In case something fails, only one AZ is affected
- Appliance mode:
    - AZ Affinity may result in Asymmetric Routing (traffic comes back using a different route compared to how it went initially)
    - This can be an issue if we are using network appliances that are stateful
    - To solve this issue, we can enable **Appliance Mode**
    - TGW uses the flow hash algorithm to select a the correct target for the return traffic

## Transit Gateway Connect Attachment

- We create a TGW Connect Attachment to establish a connection between a TGW and a third party virtual appliance (such as SD-WAN appliances) running in a VPC
- A Connect attachment uses an existing VPC or AWS Direct Connect attachment as the underlying transport mechanism
- Supports Generic Routing Encapsulation (GRE) tunnel protocol for high performance and BGP for dynamic routing (for Connect Attachments it is a must to have BGP peering)
- Connect attachments do not support static routes. BGP is a minimum requirement for Transit Gateway Connect
- TGW Connect supports a maximum bandwidth of 5 Gbps per GRE tunnel. Bandwith above 5 Gbps is achieved by advertising the same prefixes across multiple Connect peers (GRE tunnels) for the same Connect attachments
- A maximum of 4 Connect peers are supported for each Connect attachments

## Multicast

- Multicast is a communication protocol used to deliver a single stream of data to multiple receiving computers simultaneously
- Destination is a multicast group address:
    - Class D: 224.0.0.0 - 239.555.555.555
- The multicast protocol is connectionless, based on UDP and one way communication
- Multicast components:
    - Multicast Domain
    - Multicast Group and member
    - Multicast source and receivers
    - Internet Group Management Protocol (IGMP) used for group membership management
- Multicast has to be enabled on the TGW at creation, afterwards it cannot be enabled
- Supports both IPv4 and IPv6 addressing
- Supports hybrid integration with external applications
- Supports both static (API based) and Dynamic group membership through IGMP (supports IGMPv2)
- Multicast traffic in a VPC:
    - We create a multicast domain and add the participating subnets
    - We create a multicast group and associate group membership IP addresses (example: 224.0.0.100)
    - We configure the group membership statically using CLI/SDK or dynamically using IGMPv2
    - Now we can send traffic from source to multicast group IP
- We can integrate multicast with external services (such as service from on-prem). TGW does not support multicast over DX! What we can do, is to connect our service to VPC using VPN or DX. After that we configure a virtual router inside the VPC and have a GRE tunnel with the on-prem service
- Further considerations:
    - A subnet can be part of only one multicast domain
    - Hosts (ENIs) in the subnet can be part of one or more multicast groups within the multicast domain
    - TGW issues an IGMPv2 QUERY message to all members every two minutes. Members renew their membership by responding with a IGMPv2 JOIN message
    - Members that do not support IGMP must be added or removed from the group using the AWS console or CLI
    - `Igmpv2Support` attribute determines how group members join or leave a multicast group. When the attribute is enabled, members can send JOIN or LEAVE messages
    - `StaticSourceSupport` multicast domain attributes determine wether there are static multicast sources for the group
    - A non-Nitro instance cannot be a multicast sender. If we use a non-Nitro instance as a receiver, we must disable Source/Destination check
    - Multicast routing is not supported over DX, S2S VPN, TGW peering attachments or TGW Connect attachments
    - Security group configuration on the IGMP host and any ACLs configuration on the host subnets must allow these IGMP protocol messages:
        - NACL Inbound:
            - IGMP(2) from `0.0.0.0/32` (IGMP QUERY sent to 224.0.0.1/32)
            - UDP traffic
        - NACL Outbound:
            - IGMP(2) from `224.0.0.2/32` (for the IGMP LEAVE)
            - IGMP(2) from the multicast group IP address (For the IGMP JOIN)
            - UDP traffic
        - SG Inbound:
            - IGMP(2) from `0.0.0.0/32` (IGMP QUERY)
            - UDP traffic traffic to the multicast IP address
        - SG Outbound:
            - IGMP(2) from `224.0.0.2/32` (for the IGMP LEAVE)
            - IGMP(2) from the multicast group IP address (For the IGMP JOIN)
            - UDP traffic to the multicast IP address
- Multicast domains can be shared between AWS accounts with AWS RAM

## Centralized Egress to Internet With NAT Gateway

- We can uses a NAT gateway in each AZ for high availability and for saving inter AZ data transfer cost
- If one AZ fails, all the traffic can be sent via the TGW and NAT gateway endpoints in another AZ
- A NAT gateway can support up to 55_000 simultaneous connections to each unique destination
- NAT gateways can scale from 5 Gbps to 100 Gbps
- We can use Blackhole routes in the TGW route tables to restring inter-VPC traffic
- This architecture does not offer a lot of cost savings compared to placing a NAT gateway in each VPC

## Centralized Traffic Inspection with Gateway Load Balancer and Network Appliances

- Using AWS PrivateLink, GWLB Endpoint routes traffic to the GWLB. Traffic is routed securely over Amazon network without any additional configuration
- GWLB encapsulates the original IP traffic using GENEVE protocol and forwards it to the network appliance over UDP port 6081
- GWLB uses 5-tuples or 3-tuples of an IP packet to pick an appliance for the life of the flow. This creates session stickiness to an appliance for the life of a flow required for stateful appliances like firewalls
- This combined with TGW Appliance Mode provides session stickiness irrespective of source and destination AZ

## Centralized VPC Interface Endpoints

- VPC Interface Endpoints provide a regional level DNS entry and an AZ level DNS entry
- The regional DNS will return the IP address for all the AZ level endpoints
- In order to save the inter-AZ data transfer costs in case of a spoke VPC to the hub VPC, we can use the AZ specific DNS endpoint
- Instead of using TGW, we can use VPC peering between the spoke VPCs and the centralized VPC. This can reduce costs, although we might have to pay attention to the limits of VPC peering (as of today VPC peering supports up to 125 peering connections)

## VPC Peering vs Transit Gateway

|                                    | VPC Peering                                                                                           | Transit Gateway                                                    |
|------------------------------------|-------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|
| Architecture                       | One-to-one connection - Full mesh                                                                     | Hub and Spoke with multiple attachments                            |
| Hybrid Connectivity                | Not supported                                                                                         | Supported hybrid connectivity via VPN and DX                       |
| Complexity                         | Simple for fewer VPCs, complex as the number of VPCs increase                                         | Simple for any number of VPCs and hybrid network connectivity      |
| Scale                              | 125 peering/VPC                                                                                       | 5000 attachments per TGW                                           |
| Latency                            | Lowers                                                                                                | Additional Hop                                                     |
| Bandwidth                          | No limit                                                                                              | 50 Gbps /  attachment                                              |
| Ref Security Group                 | Supported                                                                                             | Not supported                                                      |
| Subnet Connectivity                | For all subnets across AZs                                                                            | Only subnets within the same AZ in which TGW attachment is created |
| Transitive Routing (ex IGW access) | Not supported                                                                                         | Supported (there is an ENI involved)                               |
| TCO (cost)                         | Lowest - only data transfer cost (free within the same AZ, charged for cross AZ/cross region traffic) | Pay per attachment + we pay for data transfer cost                 |

## Sharing Transit Gateways with AWS RAM

- We can share a TGW with other AWS accounts form our organization or with accounts outside of our organization
- When an TGW is shared, AWS S2S VPN attachments must be created in the same AWS account that owns the TGW
- If we are using DX Gateway to connect TGW to DX, we can use DX Gateways from other accounts
- If a TGW is shared with an account, that account cannot create, modify or delete TGW route tables, or route table propagations and associations