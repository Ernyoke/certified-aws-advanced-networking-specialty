# Route 53 Fundamentals

- Route 53 provides 2 main services:
    - We can register domains
    - It can host zone files on managed nameservers
- Route 53 a global service with a single database
- Data contained in Route 53 is globally distributed
- It provides global resilience, it can tolerate the failure of one or more regions

## Register Domains

- Route 53 has relationship with all of the major domain registries
- When a domain is registered a few things happen:
    1. Route 53 checks with the registry for the top-level domain if the domain is available
    2. Route 53 creates a zone file for the domain
    3. Route 53 allocates nameservers with the zone (4 managed nameservers for a hosted zone)
    5. It communicates with the top-level registry and adds the nameserver records into the zone file for top-level domain

## Hosted Zones

- DNS as a service
- Lets us create and manage zone files in AWS
- Zone files are hosted in AWS managed nameservers (4 nameservers)
- Hosted zone can be public available for the public internet
- Hosted zone can also be private, linked for one or more VPCs
- A hosted zone hosts DNS records (recordset)
