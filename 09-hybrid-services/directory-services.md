# AWS Directory Service

## Microsoft AD

- AWS managed AD service is built using a native Microsoft Active Directory implementation (2012 R2)
- Runs within a VPC/subnets. It can be managed by using Standard Active Directory tools
- Supports Group Policy and single sign-on (SSO)
- It has a standard AD schema, it can support extensions to it, examples being: Sharepoint, SQL, Distributed File System (DFS)
- It comes in 2 sizes:
    - Standard: up to 30_000 objects
    - Enterprise: up to 500_000 objects
- The product is used for AD Authentication/Authorization of products and services within AWS
- It is HA by default, it deploys in at least 2 AZs with one domain controller (DC) in each
- Includes monitoring, recovery, replication, snapshots and maintenance windows which are all configurable and managed by AWS
- Running a directory service in MS AD mode means it supports trusts between it an on-premises directories, both one-way and two-way external trusts
- If the network fails between the AD directory and on-premises service, the product will function as expected (contrary to AD Connector)
- Supports RADIUS-based MFA
- Best choice for more than 5000 users and if we need trust relationship between AWS and on-premises directories
- The domain controllers are HA by design and they replicate data between each other
- AWS managed AD mode means we have a full directory within AWS running a native MS Active Directory

## AD Connector

- AD Connector provides a pair of directory endpoints running in AWS (ENIs in a VPC)
- Supports directory-aware AWS products, such as Workspaces, Workdocs, Workmail, etc.
- AD Connector redirects all the requests to an existing directory server running on-premises
- No directory data is stored in AWS
- AD Connector mode allows us to use AWS services which require a directory with an on-premises directory infrastructure
- AD Connector supports architectures like proof of concepts running on AWS
- There are 2 sizes of sizes for AD Connector: small and large. They do not impose any number for the user limit, but they control the amount of compute allocated to the connector instances
- We can use multiple connectors to distribute the load
- AD connector is placed into 2 subnets in a VPC, in different AZs
- The connector requires 1 or more directory servers to be configured
- It requires a working networking connection between AWS VPCs and the on-premises AD. The network connectivity is private, either Direct Connect or VPN