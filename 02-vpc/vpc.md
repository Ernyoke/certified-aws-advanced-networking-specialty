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