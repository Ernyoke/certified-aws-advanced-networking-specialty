# AWS Shield and Web Application Firewall (WAF)

## AWS Shield

- Provides protection against DDoS attacks
- Provides a custom designed set of protection against DDoS attacks
- Comes in 2 versions:
    - **Shield Standard**:
        - Offers protection against all known infrastructure Layer 3 and Layer 4 DDoS attacks
        - It is free of charge
        - If offers comprehensive L3 and L4 protection when used with Route53 and CloudFront
    - **Shield Advanced**:
        - Costs $3000 per month per organization
        - Expands the range of products which can be protected: EC2, ELB, CloudFront, Global Accelerator and Route53
        - Shield Advanced provides access to 24/7 advanced response team
        - Provides financial insurance for any increase of payments in case of DDoS attacks

## WAF - Web Application Firewall

- It is Layer 7 Firewall (understands HTTP/S)
- Normally firewall operate at Layer 3, 4, 5
- WAF protects against complex Layer 7 attacks/exploits such as SQL Injection, Cross-Site Scripting
- It can filter based on location (Geo Blocks), and provides rate awareness
- Web Access Control List (WEBACL) integrated with ALB, API Gateway and CloudFront
- WEBACL has rules and they are evaluated when traffic arrives
- (WAF) Rules:
    - Structure: WEBACL => RULE GROUPS => RULES
    - WEBACL Capacity Units (WCU): how the products accounts for the complexity of rules, there is a limit of how many WCU can be on a single ACL
    - We are charged a monthly fee for each ACL, a monthly fee for each rule and charge based on the number of requests
    - WEBACL have a default action which can be `ALLOW` or `BLOCK`, based on that each rule can allow or block traffic
    - Another action is `COUNT`, can be used for testing. Allows to track how many times a rule allow after we can add another action such as block
    - Rules can be 2 type:
        - Regular: take a defined action based on the statement of the rule
        - Rate based: how frequently a single IP can go through an ACL