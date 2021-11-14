# Advanced VPC Routing

- Subnets are associated with 1 route table only, which is either the main RT from the VPC or a customer RT
- In case an explicitly associated RT is removed from the subnet, the main RT is associated again with it
- RTs can be associated with an Internet Gateway or Virtual Private Gateway. This allows them to be used to control traffic flow entering into the VPC
- IPv4 and IPv6 are handled separately within an RT
- Routes send traffic based on destination to a target
- A RT has a default limit of 50 static routes and 100 propagated routes if enabled
- Whenever traffic arrives to a VPC interface, IGW or VGW, it is matched against in the routes in the relevant route table
- All routes are evaluated, but the highest-priority matching one is used