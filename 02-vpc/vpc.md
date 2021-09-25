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