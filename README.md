# certified-aws-advanced-networking-specialty

## Table of Contents

1. Networking and Technical Fundamentals
    - [OSI 7-Layer Model](01-fundamentals/osi.md)
    - [NAT - Network Address Translation](01-fundamentals/nat.md)
    - [DDoS - Distributed Denial of Service Attacks](01-fundamentals/ddos.md)
    - [SSL - Secure Sockets Layer and TLS - Transport Layer Security](01-fundamentals/ssl.md)
    - [DNS 101](01-fundamentals/dns.md)
    - [Route 53 Fundamentals](01-fundamentals/route53.md)
    - [VLANs, Trunks and Q-in-Q](01-fundamentals/vlan.md)
    - [Jumbo Frames and MTU](01-fundamentals/jumbo.md)
    - [Reserved IPv4 Ranges](01-fundamentals/reserved-ipv4-ranges.md)
2. Virtual Private Cloud
    - [VPC](02-vpc/vpc.md)
    - [VPC Endpoints](02-vpc/vpc-endpoints.md)
    - [Advanced EC2 Networking](02-vpc/ec2-networking.md)
    - [VPC Peering](02-vpc/vpc-peering.md)
3. Networking Automation
    - [CloudFormation](03-networking-automation/cloudformation.md)
4. Elastic Load Balancing - Deep Dive
    - [ELB - Elastic Load Balancer](04-lb/elb.md)
    - [GWLB - Gateway Load Balancers](04-lb/gwlb.md)
5. Route 53 - Deep Dive
    - [Route 53](05-r53/r53.md)
6. Network Content Delivery (CDN) in AWS
    - [CloudFront](06-cdn/cloudfront.md)
    - [ACM - AWS Certificate Manager](06-cdn/acm.md)
7. Network Security, Risk and Compliance
    - [S3 Access Points](07-security/s3-access-points.md)
    - [CloudTrail](07-security/cloudtrail.md)
    - [`ip-ranges.json`](07-security/ip-ranges.md)
    - [Application Layer (L7) Firewalls](07-security/application-layer-firewalls.md)
    - [AWS Shield](07-security/shield.md)
    - [URL Filtering in a VPC](07-security/url-filtering.md)
    - [AWS Config](07-security/config.md)
    - [CloudHSM](07-security/cloudhsm.md)
    - [Amazon Macie](07-security/macie.md)
    - [AWS Inspector](07-security/inspector.md)
    - [AWS Control Tower](07-security/control-tower.md)
    - [AWS Firewall Manager](07-security/firewall-manager.md)
8. Hybrid Networking
    - [IPSEC VPN Fundamentals](08-hybrid-networking/ipsec.md)
    - [VGW - Virtual Private Gateway](08-hybrid-networking/vgw.md)
    - [AWS Site-to-Site VPN](08-hybrid-networking/s2s-vpn.md)
    - [Client VPN](08-hybrid-networking/client-vpn.md)
    - [BGP - Border Gateway Protocol](08-hybrid-networking/bgp.md)
    - [TGW - AWS Transit Gateway](08-hybrid-networking/tgw.md)
    - [AWS Global Accelerator](08-hybrid-networking/global-accelerator.md)
    - [Rout Priority and Selection within AWS](08-hybrid-networking/route-priority.md)
    - [DX - Direct Connect](08-hybrid-networking/dx.md)
    - [AWS Cloud WAN](08-hybrid-networking/cloud-wan.md)
9. Hybrid Services
    - [AWS Directory Service](09-hybrid-services/directory-services.md)
    - [Amazon Workspaces](09-hybrid-services/workspaces.md)
    - [FSx](09-hybrid-services/fsx.md)
    - [Storage Gateway](09-hybrid-services/storage-gateway.md)
10. EKS Networking
    - [Kubernetes and EKS Networking](10-eks-networking/kubernetes.md)
11. Network Billing and Cost Management
    - [VPC Cost Management](11-cost-management/vpc-cost-management.md)
    - [DX Cost Management](11-cost-management/dx-cost-management.md)
12. Disaster Recovery
    - [DR/BC Architecture](12-dr/dr.md)
13. Network Management and Governance
    - [AWS Network Manager](13-network-management-and-governance/aws-network-manager.md)
    - [IPAM - IP Address Management](13-network-management-and-governance/ipam.md)
    
## Exam Description

The AWS Certified Advanced Networking - Specialty (ANS-C01) exam is intended for individuals who perform an AWS networking specialist’s role. The exam validates a candidate's ability to design, implement, manage, and secure AWS and hybrid network architectures at scale.

The exam also validates a candidate’s ability to complete the following tasks:
- Design and develop hybrid and cloud-based networking solutions by using AWS
- Implement core AWS networking services according to AWS best practices
- Operate and maintain hybrid and cloud-based network architecture for all AWS services
- Use tools to deploy and automate hybrid and cloud-based AWS networking tasks
- Implement secure AWS networks using AWS native networking constructs and services

## Preparation

- Official Exam Guide: [https://d1.awsstatic.com/training-and-certification/docs-advnetworking-spec/AWS-Certified-Advanced-Networking-Specialty_Exam-Guide.pdf](https://d1.awsstatic.com/training-and-certification/docs-advnetworking-spec/AWS-Certified-Advanced-Networking-Specialty_Exam-Guide.pdf)

- There are two types of questions on the examination:

- Multiple choice: Has one correct response and three incorrect responses (distractors).
- Multiple response: Has two or more correct responses out of five or more options.

- Minimum passing score: 750/1000

- Number of questions: 65

- Time: 170 minutes to complete the exam (with a possibility to request 30 minutes extra for non-native english speakers)

## Exam Content

| **Domain**                                                              | **% of Examination** |
|-------------------------------------------------------------------------|----------------------|
| Domain 1: Network Design                                                | 30%                  |
| Domain 2: Network Implementation                                        | 26%                  |
| Domain 3: Network Management and Operation                              | 20%                  |
| Domain 4: Network Security, Compliance, and Governance                  | 24%                  |
| **Total**                                                               | **100%**             |