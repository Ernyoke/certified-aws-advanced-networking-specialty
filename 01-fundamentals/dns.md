# DNS 101

- DNS is a discovery service
- It translates information needed by machines to information required by humans and vice-versa, example: `www.amazon.com` => `104.98.34.131`
- DNS is a huge database and has to be distributed and resilient

## DNS Zone

- For computers to communicate with `www.amazon.com`, the DNS name has to be translated to an IP address. This is entirely abstracted for the users
- DNS is a huge scale database, distributed and global
- In all of this distributed database there must exist a database which has the information we need in order to convert the DNS address into IP address
- This database is called Zone, they way that this zone is stored is often referred as zonefile
- Zones files contain records in them, a DNS records which links the name to an IP address
- The zonefile is hosted on a Nameserver (NS)
- One of the core piece of functionality that DNS provides, is that allows the a DNS resolver server to find the zone. Once located it, query it and use the results to access the website

## Terminology

- DNS Client: a piece of software running on a device, talks with the DNS Resolver
- DNS Resolver: a piece of software running also on a device, or a separate server. This software queries the DNS on our behalf
- DNS Zone: a part of the DNS database
- Zonefile: how the data for a zone is physically stored
- Nameserver: server which hosts the zonefiles

## DNS Root

- DNS is structured like a tree, the DNS root is at the top of the tree
- DNS root occupies the top most point of DNS
- The DNS root is hosted on 13 special nameservers, operated by 12 different organizations
- Each root server has a letter from `a` to `m`
- Our devices communicate initially with the DNS resolved which supplies the Root Hints file
- This Root Hints file is the pointer to the DNS routes
- Using the Root Hints file, the client communicates with one or more DNS roots accessing the Root Zone
- The Root Zone content is managed by IANA

## DNS Hierarchy

- DNS is based on trust
- When something is trusted in DNS, it is know as authoritative
- At the beginning of DNS resolving the root zone is the only authoritative entity
- Delegation: the root zone can delegate a part of itself to another entity/another zone. This entity becomes authoritative as well
- The root zone and thus the zone file is just a database for top level domains (example: .com, .org, .uk)
- IANA don't run anything on the root zone besides the top level domains, they delegate management to other organizations named registries

## DNS Record Types

- Nameserver records (NS): allow delegation to occur in DNS, point at other authoritative servers
- A and AAAA records: map host names to IP addresses. A records maps the DNS to an IPv4 address, AAAA maps it to a IPv6 addresses
- CNAME (Canonical Name) records: lets us create host to host records. Used to reduce admin overhead. Example: CNAME ftp, mail and www can point to a single name
- MX records: used for email server. 
    - MX records have two parts, a priority and a value
    - The value can be the host mail server It can can point to a server inside the zone or a host outside of the zone (by appending a dot in the end becoming a FQDN)
    -  The priority records indicate which host should be prioritized and picket first
- TXT records: allows to add arbitrary text to a domain
    - Common usage: prove domain ownership by adding a specific text to the domain
    - Other uses are spam fighting, etc.
- TTL: a numeric value in seconds. We can indicate how long records can be cached for at the resolver server