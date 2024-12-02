# AWS Firewall Manager

- It is a security management tool which simplifies security administration and maintenance tasks across multiple AWS accounts and resources
- Manages the rules for AWS WAF, AWS Shield Advanced, Amazon VPC Security Groups, AWS Network Firewall and Amazon Route 53 Resolver DNS Firewall
- It works at AWS Organizations level, new accounts added to the org are automatically protected
- Provides the centralized monitoring of DDoS attacks across AWS organizations
- To use AWS Firewall Manager we need the following pre-requisites:
    - Enable AWS Organizations (full feature)
    - Enable AWS Config
    - Enable AWS Resource Access Manager (RAM)
- AWS Firewall can also manage third party Firewalls from the Marketplace, example: Palo Alto Cloud Next Generation Firewall (NGFW), Fortigate Cloud Native Firewall (CNF)