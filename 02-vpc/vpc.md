# VPC

## Public vs Private Services

- Public service: a service which is accessed by using public endpoints
- Private service: a service which runs inside a VPC
- Either private or public, every service can have permissions in order to be accessible
- VPC: private network isolated from the internet. Can't communicate to the network unless we are allowing it. Nothing from the internet can reach the services from a VPC as long as we do not configure it otherwise
- Internet Gateway: we can connect it to a VPC, this will allow the services in the VPC to communicate with the public internet

## Custom VPCs

- VPCs are regional isolated services
- A VPCs is created in a region and it operates in all availability zones of the selected region
- VPCs are isolated networks, we can have multiple isolated networks
- No traffic is allowed in or out of a VPC without explicit configuration
- Custom VPCs support hybrid networking, which lets us connect our VPCs to other cloud providers or on-premises networks
- When a VPC is created, we can chose between default and dedicated tenancy. This controls if the resources in the VPC run on a dedicated hardware or shared hardware
- Dedicated tenancy allows resources running only on dedicated HW, recommended to chose default tenancy if there is no good reason for dedicated
- A VPC can use IPv4 public and private IPs
- By default every resource uses private IPs, public IPs are used when we want to allow other people to access our resources from the internet
- A VPC should have 1 primary private IPv4 CIDR block. This has to be specified when the VPC is created
- This primary CIDR block has the following restrictions:
    - Min /28 (16 IPs, some of the them cannot be used)
    - Max /16 (65536 IPs)
- Optionally we can add secondary IPv4 blocks, max 5 (can be increased with a ticket to the support)
- Optional single assigned IPv6 /56 CIDR block can be assigned to the VPC
- IPv6 block can be picket by AWS or we can use addresses we own
- IPv6 addresses are all public by default, but resources still need to be given explicit access to be reachable

## DNS in VPC

- AWS VPCs have fully featured DNS
- This is provided nby Route 53
- The DNS inside of a VPC is available at the Base IP + 2 address (example: 10.0.0.0 -> DNS ip is 10.0.0.2)
- DNS options for VPCs:
    - `enableDnsHostNames`: gives public instances a DNS name by default
    - `enableDnsSupport`: DNS resolution is enabled/disabled in the VPC

## VPC Subnets

- A subnet is a subnetwork of a VPC, a port of a VPC which is inside of a specific availability zone
- It is created within one AZ and this can not be changed
- Subnets provide AZ resiliency and high availability by putting resources in physically different locations
- Relationship of subnet and AZ: 1 Subnet is in 1 AZ, 1 AZ can have 0 or more subnets
- A subnet by default is using IPv4 which is a subset of the VPC CIDR range
- A CIDR from a subnet can not overlap with other subnets
- Optional IPv6 CIDR can be assigned to a subnet (/64 subset of the /56 VPC - space for 256 addresses)
- Subnets can communicate with other subnets in a VPC by default
- Not all of them can be assigned to resources
- Reserved IP addresses (5 in total) for a 10.16.16.0/20 subnet:
    - Network address (10.16.16.0)
    - Network + 1 (10.16.16.1) - VPC Router (network interface)
    - Network + 2 (10.16.16.2) - Reserved for every subnet (DNS*)
    - Network + 3 (10.16.16.3) - Reserved for future use
    - Broadcast address (10.16.31.255) - Last IP of a subnet
- VPCs have a configuration object applied to them, called DHCP Option Set
- Option sets can be configured manually on a VPC but they cannot be changed. Some option sets are:
    - Auto Assign IPv4, IPv6

## DHCP in a VPC

- DHCP - Dynamic Host Configuration Protocol: offers auto configuration for network resources
- Every device has a hard-coded MAC address (Layer 2 address)
- DHCP begins with a L2 broadcast to discover a DHCP server on the local network
- Once discovered a DHCP server and a DHCP clients communicate, meaning that the client will get in the end an IP address, a Subnet Mask and Default Gateway address (L3 configuration)
- DHCP also configures which DNS server should a resource use in a VPC
- Also configures NTP servers, NetBios Name Servers and Node types
- For DNS server we can explicitly provide values or we can use `AmazonProvidedDNS`
- We also get allocated 1 or 2 DNS names for the services in the VPC. One can be public if the instance has a public IP address allocated
- Custom DNS names: we can give custom DNS names to EC2 instances if we use our own custom DNS servers
- DHCP options sets:
    - Once created option sets can not be changed
    - Can be associated with 0 or more VPCs
    - Each VPC can have a max of 1 option set associated
    - We we change a DHCP option set associated to the VPC, the change is immediate, but any new setting will only affect anything once a DHC renew occurs
    - What we can configure in an option set:
        - DNS server (Route 53 resolver) what we can use in the VPC
        - NTP server

## VPC Router Deep Dive

- Is at the core of any network which involves AWS
- Is a virtual router in a VPC
- It is HA across al AZs in a region, no management overhead is required
- It is scalable, no management overhead required
- VPC routes routes traffic between subnets in a VPC
- Routes traffic from external network into the and vice-versa
- VPC router has an interface in every subnet in a VPC: `subnet+1` address (Default Gateway), the first IP address in each subnet after the network address itself
- We control how the VPC routes traffic using Route Tables

## VPC Route Tables

- Every VPC is created with a main Route Table (RT), which is the default for every VPC
- Custom route tables can be created for each subnet
- Subnets can be associated with only one RT which can be the main one or custom
- If we disassociate a custom RT form a subnet, the main RT will be attached to it
- Main RT should not be changed, custom RT should be used for any routing changes
- RT have routes, routes have an order, the most specific route wins
- Edge Association: a RT tables is associated with network gateway
- All RTs have at least one route: the local route which matches the VPC cidr range. These routes are un-editable

## NACL - Network Access Control Lists

- A NACL can be considered to be a traditional firewall in an AWS VPC
- NACLs are associated with subnets, every subnet has a NACL associated to it
- Connection inside a subnet are not affected by NACLs
- NACls can be considered stateless firewalls, so we can talk about the following type of rules:
    - **Inbound rules**: affect data coming into the subnet
    - **Outbound rules**: affects data leaving from the subnet
- Rules can explicitly **ALLOW** and explicitly **DENY** traffic
- Rules are processed in the following order:
    1. A NACL determines if a the inbound or outbound rules apply
    2. It starts from the lower rule number, evaluates traffic against each rule until is a match (based on IP range, port, protocol)
    3. Traffic is allowed/denied based on the rule
- Last rule is an implicit deny in every NACL, if no rule before that applies, traffic will be denied
- Default NACL: when a VPC is created, a default NACL is attached to it. The default NACL is allowing all traffic
- Custom NACL: we can create them and attach them to subnets. The default NACL denies by default all traffic. Can be associated with many different subnet, however each subnet can have only one NACL associated to it at any time
- NACL are not aware af any logical resources within a VPC, they are aware of IPs, CIDRs and protocols

## SG - Security Groups

- Security Groups are stateful firewalls, meaning they detect response traffic to a request and they automatically allow it
- SGs do not have explicit **DENY** rules, they can not be used to block bad actors (use NACLs for this)
- SGs support IP/CIDR rules and also allow to reference logical resources (including other SGs and event itself)
- SGs are attached to Elastic Network Interfaces (ENI), when we attach a SG to an EC2, the SG will be attached to the primary ENI
- SGs are capable to reference logical resources, ex. other security groups or self referencing

## VPC Flow Logs

- Provide details of traffic flow inside of a VPC
- VPC Flow Logs only capture metadata, they do not capture the content of packets. For capturing content we would need to have a packet sniffer deployed in our VPC
- Flow Logs works by attaching actual monitors in a VPC. These monitors can be attached at 3 different levels:
    - VPC level: monitors every network interface in every subnet
    - Subnet level: monitors every network interface in the subnet
    - ENIs directly: monitor a specific interface only
- Flow Logs are not realtime, we can't rely on Flow Logs to offer real time telemetry
- Flow Logs can be configured to save data in different destinations. Currently S3 and CloudWatch Logs are supported
- We can use Athena if we want to query Flow Logs in an S3 bucket using a SQL-like query language
- FLow Logs can be configured to capture metadata on only `ACCEPT`ed connections, only `REJECT`ed connections or capture ALL metadata
- Flow Logs product capture Flog Log Records, which contain the following fields:
    - Version
    - Account ID
    - Interface ID
    - Srcaddr: source IP address
    - Dstaddr: destination IP address
    - Srcport: source port
    - Dstort: destination port
    - Protocol: 
        - 1 - `ICMP`
        - 6 - `TCP`
        - 17 - `UDP`
    - Packets
    - Bytes
    - Start
    - End
    - Action: `ACCEPT` or `REJECT`. Traffic is accepted or rejected
    - Log-status
- VPC Flow logs exclude the following traffic from logging:
    - Requests to `169.254.169.254`, `169.254.169.123`
    - DHCP traffic
    - Requests to Amazon DNS server
    - Requests to Amazon Windows License server

## IPv6 Capability in VPCs

- IPv6 addresses are all publicly routable
- NAT is not used for IPv6, IPv6 does not need network address translation simply because of the huge number of available IPv6 addresses
- IPv6 needs to be manually enabled on a VPC. We can either bring our own IP address in a VPC or utilize an AWS provided range
- In case of AWS provided IPv6 addresses, AWS will allocate an uniq /56 range to the VPC. This range will be entirely uniq and all addresses will be publicly routable
- If we chose to allocate an IP range for a VPC, AWS will use a hex pair to uniquely allocate IP addresses to the subnets
- Routing is handled separately for the IPv6 addresses, we will have IPv4 routes and IPv6 routes
- Egress only internet gateway: similar to NAT gateway, allows outbound traffic denying inbound traffic in case of IPv6 addressing. NAT gateways or instances do not support IPv6!
- We can have both internet gateway and egress only interne t gateway associated to the same subnet
![IPv6 Architecture](IPv6EOIGW.png)
- IPv6 can be set up while creating a VPC/subnet or we can migrate an existing VPC to IPv6
- We can enable IPv6 on specific subnets only
- We can point IPv6 traffic to internet gateway and egress only internet gateways as well
- Not every service in AWS supports IPv6!

## IGW - Internet Gateway

- IGWs can be created and attached to AWS VPCs, but they are not mandatory, essentially making the VPC private
- IGWs have a 1 to 1 relationship with VPCs, 1 IGW can be attached to only 1 VPC, VPCs can have 0 or 1 IGWs
- Architecturally internet gateways are at the border of the VPC and the AWS public zone
- IGWs are used to access AWS public services and the public internet
- IGWs are highly available by default and the do scale by default. They are managed by AWS
- IGW works with IPv4 and IPv6 for both inbound and outbound traffic
- With IPv4 the IGW performs 1:1 static NAT
- For both IPv4 and IPv6 the IGW routes traffic in and out of the VPC
- Fop IPv6 the internat gateway routes packets in and out of the VPC with no modifications
- Traffic routed for public AWS services through the IGW does not live the AWS network
- Making a subnet public:
    1. Create an IGW and attach it to the VPC
    2. Create a custom route table and associate it to the subnet we want to make public
    3. Create some default routes
    4. Make sure that public instances are launched with a public IP (or make sure the subnet attaches a a public IP automatically to the instances)
- For IPv6 the 4. step does not apply


## Egress-Only Internet Gateway

- It is an internet gateway only allowing traffic from the inside of a VPC to the outside world
- With IPv4 addresses are either private or public, private addresses can not directly communicate to the public internet while public instances can communicate to the public internet and be communicated with from the public internet
- NAT gateway allows private IPs to access the public internet, but without allowing externally initiated connections
- NAT exists because of limitations of IPv4, it does not work for IPv6
- With IPv6 all IPs are publicly routable, meaning an IGW will allow all IPs to communicate out and to handle incoming requests
- Egress-Only Internet Gateways allow IPv6 traffic to be initiated out from the VPC, but they are not allow inbound connections
- With normal IGW, instances can connect out and external instances can connect in. With Egress-Only IGW, instances from the VPC can initiate connections to the outside, but they can not be connected to from the outside
- IGWs are stateful devices, they allow responses to outgoing traffic

## Bring your own IP Address (BYOIP)

- AWS owns IPv4 and IPv6 addresses we can use
- For IPv4 we can use them as elastic IP addresses (on EC2, NATGW, etc.)
- For IPv6 we can assign them directly to the VPC
- IP addresses which are allocated to our resources are owned by AWS. They are not portable, we can't use them on-premises
- Because AWS own these IP addresses, they are advertising them via BGP and they are authorized to do so
- As an organization, we can control IPv4 and IPv6 ranges, meaning that we allocated these ranges via a regional internet registry (RIR)
- If we control these addresses, we can port them into AWS and we can authorize AWS to BGP advertise them from ASN 16509 and 14618
- The internet will see these addresses as within AWS
- BYOIP steps:
    1. and 2. Key Pair and Cert
        - Create a an RSA (AES256) private key and a corresponding public key
        - Generate a x509 certificate signed with the private key
            - The only mandatory field is the common name
        - Register with the Resource Public Key Infrastructure (RPKI) - requests made are signed with the private key 
    3. Route Origin Authorization (ROA)
        - We have a Regional Internet Registry (RIR) allocated address
        - Route Origin Authorization: we can think of as a document which contains the IP range together with who is authorized the advertise it
        - Sign the ROA and provide it to the registry (RIR)
    4. RDAP Record
        - Core router: perform origin validation to make sure advertisements are valid
        - RDAP record is updated with the address space within the RIR, self-signed certificate is added to the RIR record
        - RDAP provides a protocol for anybody to be able to identify who is allowed to advertise an IP range
    5. and 6. Provision and Advertise
        - Create an authorization message for AWS and sign it with the private key
        - AWS uses RDAP to verify the control of the IP range
- The most specific IPv4 address range we can bring into AWS is a /24, and for IPv6 is /48 for publicly routable ranges and /56 for CIDRs that are not publicly advertised
- BYOIP is one region at a time
- We can import 5 IPv4 and IPv6 ranges per region per account
- We can not share our IP address range with other accounts using AWS Resource Access Manager (RAM)

## Bastion Hosts

- Is a server which runs at a network edge
- It is hardened to withstand attacks from public networks
- Acts as a gatekeeper between two different zones (public => private)
- A **jumpbox** is a generic term used for a server between two or more networking zones. It allows to jump between networks using this server
- A bastion host is a subset o jumpboxes, it runs between public and private networks and it is hardened
- A bastion host is essentially used an an ingress control point, a single hole at the perimeter of a private network. It is extensively logged, it has some form of authentication (SSH, ID federation) and has tightly controlled network security

## NAT Instances

- Before the introduction of the NAT Gateways, this was the recommended way to give internet access to resources from private networks
- It is just a normal EC2 instances configured to provide NAT services
- It is no more recommended to use without any specific reason, it is running Amazon Linux 2018.03
- NAT instances are non-specialized, there are EC2 instances running  NAT software
- The speed we get from a NAT instance is based on size and type of EC2 instances
- With EC2 instances for NAT we have to disable traffic source/destination checks, otherwise the packets will be dropped
- With NAT instances we can have SG on the ENI and NACLs on the subnet
- NAT instances can save costs by being able to be configured as bastion instances
- NAT instances are self managed
- NAT instances do not have built-in resilience! We can use scripts to implement high availability!
- For logging/monitoring we can use Flow Logs
- NAT instances work across DX, VPN and Peering connections

## NAT Gateways

- AWS recommended way of doing NAT process in a VPC
- The NAT gateway itself does not have a public IP address, the communication with the public internet happens through the IGW
- The NAT gateway communicates with the IGW while all the public traffic is routed from the private VPC to the NAT GW
- NAT gateways needs to run from a public subnet because it requires to use an static IPv4 address to be able to do network address translation
- NAT GW are AZ resilient services (HA is provided in the AZ)
- For region resilience we need to deploy NAT gateways in each AZ we use and point the route table to redirect traffic to NAT gateways in the specific regions
- NAT gateway are not expensive but they can get costly if we have a lot of AZs
- NAT gateways are managed by AWS, they provide 5Gbps by default, they can scale up to 45Gbps. We can have multiple NAT GWs in a single AZ for traffic splitting
- With NAT gateways we are paying for each instance per duration of usage + there is data ingestion cost
- NAT gateways are using Elastic IP addresses, the IP address can not be changed. If we want to change the IP address, we need to provision a new NAT GW
- NAT gateways can not be used across VPC peers, Site-to-Site VPN connections or AWS Direct Connect (NAT instances can be used!!!)
- A NAT GW can handle 55000 simultaneous connections to each unique destination. If we reach this point, we may run into port allocation error, this can be monitored with `ErrorPortAllocation' metric
- NAT gateways inquire data transfer costs even when we are using them for accessing AWS services. Use gateway endpoints for accessing S3/DynamoDB, they inquire no additional cost
- **We can't use security groups on the NAT gateways!**

# Advanced VPC Routing

- Subnets are associated with 1 route table only, which is either the main RT from the VPC or a customer RT
- In case an explicitly associated RT is removed from the subnet, the main RT is associated again with it
- RTs can be associated with an Internet Gateway or Virtual Private Gateway. This allows them to be used to control traffic flow entering into the VPC
- IPv4 and IPv6 are handled separately within an RT
- Routes send traffic based on destination to a target
- A RT has a default limit of 50 static routes and 100 propagated routes if enabled
- Whenever traffic arrives to a VPC interface, IGW or VGW, it is matched against in the routes in the relevant route table
- All routes are evaluated, but the highest-priority matching one is used
- Route Table evaluation priority:
    - 1. Longest prefix wins
    - 2. Static routes
    - 3. Propagated routes
    - 4. For propagated routes the priority list is the following
        - 1. Routes learned via DX
        - 2. VPN Static
        - 3. VPN BGP
        - 4. AS_PATH (distance with 2 different autonomous systems)

##  Advanced VPC Routing - CIDR Overlap

- It is not recommended to avoid having VPCs with overlapping IP ranges
- If we really need to deal with this kind of networks, we can do the following:
    - In case of VPC with multiple subnets peering with 2 VPCs with overlapping IP ranges, we can create different route tables for each subnet a let them peer to different VPCs
    - Use a specific route (`/32` prefix) for certain services deployed in the overlapping VPCs