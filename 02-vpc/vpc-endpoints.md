# VPC Endpoints

## AWS PrivateLink

- Tech stack which supports VPC interface endpoints
- Enables us to connects to services in a secure way which are hosted by other AWS accounts. This are called AWS Endpoint Services
- We can connect to these directly or we can utilise AWS partner services from the AWS Marketplace. In both cases these services are presented in our VPC with a private IP address and with ENIs
- AWS PrivateLink architecture:

    ![AWS PrivateLink Architecture](images/PrivateLink.png)

- From a consumer VPC we can access the provided by injecting Interface Endpoints into our subnets
- Interface Endpoints can be secured with SGs and NACLs
- By using AWS PrivateLink, the data does not go through the public internet
- AWS PrivateLink key facts:
    - For HA we should deploy multiple Interface Endpoints, one for each AZ inside the VPC
    - PrivateLink supports IPv4 and TCP only (no IPv6)
    - Private DNS is supported. This can override the public DNS (if public DNS is provided)
    - PrivateLink services can be access via DX, Site-to-Site VPN and VPC Peering

## Gateway Endpoints

- Gateway endpoints provide private access to supported services: **S3** and **DynamoDB**
- They allow any resource in a private only VPC to access S3/DynamoDB
- We create a gateway endpoint per service per region and associate it to one or more subnets in a VPC
- We allocate a gateway endpoint to a subnet, a *Prefix List* is added to the route table for the subnet
- Any traffic targeted to S3/DynamoDB will go through the gateway endpoint and not through the internet gateway
- Gateway endpoints are highly available across all AZs in a region, they are not directly inside a VPC/subnet
- **Endpoint policy**: allows what things can be connected to the by the endpoint (example: a particular subset of S3 buckets)
- Gateway endpoints can be used to access services in the same region only
- Gateway endpoints allow private only S3 buckets: S3 buckets can be set to private allowing only access from the gateway endpoint. This will help prevent *Leaky Buckets*
- Gateway endpoints are logical gateway objects, they can be only accessed from inside the assigned VPC
- Gateway endpoints architecture:

    ![Gateway Endpoints Architecture](images/GatewayEndpoints.png)

## Interface Endpoints

- Interface endpoints provide private access to AWS public services similar to Gateway Endpoints
- Historically they have been used to provide access to services other than S3 and DynamoDB, recently AWS allowed interface endpoints to provide access to S3 as well
- Difference between gateway endpoints and interface endpoints is that interface endpoints are not HA. Interface endpoints are added to subnets as an ENI
- In order to have HA, we have to add an interface endpoint to every subnet per AZ inside of a VPC
- Interface endpoints are able to have security groups assigned to them (gateway endpoints do not allow SGs)
- We can also use endpoints policies, similar to gateway endpoints
- Interface endpoints support TCP only over IPv4
- Interface endpoints use PrivateLink behind the scene
- Gateway endpoints use prefix lists, interface endpoints use DNS. Interface endpoints provide a new DNS name for every service they are meant communicate with
- Interface endpoints are given a number of DNS names:
    - Endpoint Region DNS
    - Endpoint Zonal DNS
    - PrivateDNS overrides the default service DNS with a new version pointing to interface endpoint
- Interface endpoint architecture:

    ![Interface Endpoints Architecture](images/InterfaceEndpoints.png)