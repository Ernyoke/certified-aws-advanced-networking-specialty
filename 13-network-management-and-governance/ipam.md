# IPAM - IP Address Management

- Used to track and manage IP addresses in use and available IP addresses
- IPAM components:
    - Scope: 
        - Can be public or private. Public for public IP addresses, private for private IP ranges
        - When we enable IPAM, the public and private scopes are created for us
    - Pools:
        - Collection of CIDR ranges
        - We can layered pools: top level pools with sub-pools
        - When we create VPCs we can allocate VPC CIDRs from the sub-pools
- IPAM can be used as part of an AWS organization to manage IP addresses for all the member accounts
- When used with AWS Organizations, we can create a Service Control Policy (SCP) to enforce CIDR allocation through IPAM while creating VPCs
- We can also have rules that would enforce how used would use IPAM to create VPCs. We can enforce things such as VPC size, VPC tag, IP range for specific region, etc.

## Tracking and Monitoring IP Address Usage

- With IPAM we can track the following:
    - Monitor CIDR usage with IPAM Dashboard
    - Monitor CIDR usage by resource
    - Monitor IPAM with Amazon CloudWatch
    - View IP address history
    - View public IP insights

## Other

- IPAM enables cross-region and cross-account sharing of our BYOIP addresses
- Pricing options:
    - Free tier: we can manage public IP pools only
    - Advanced tier: we can manage private IP pools as well