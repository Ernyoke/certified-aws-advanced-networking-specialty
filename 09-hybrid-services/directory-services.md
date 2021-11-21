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