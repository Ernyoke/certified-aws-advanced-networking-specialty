# VPC Cost Management

- Cost of data transfers from/in a VPC:
    - If a public instance from a public VPC accesses a public service (S3, DynamoDB, etc.) in the same region, the transfer between the instance and the IGW and the transfer between the IGW and the service are both free
    - Situation is similar from a private VPC to a service is also free, but there is a data transfer charge for the NAT gateway and additionally is a hourly charge for a NAT gateway for running
    - In case of data transfer from a VPC to the public internet AWS imposes a charge
    - Any data transferred in from the public internet to AWS is free of charge
    - There is also a charge for data transfer between a VPC from a region to another region. The price changes based on the source and destination regions
- Cost of data transfer inside of a VPC:
    - In most cases the data transfer between subnets if free of charge even if the data does not crosses an or more AZs
    - If the data transfer is cross AZ, there is charge for it:
        - If there is a transfer between instances in AZ A and AZ B, there is a cost for both inbound and outbound transfer
    - Some services allow cross AZ data transfers for free (ex: RDS replication)
- To keep in mind:
    - If 2 instances in the same AZ, same subnet, communicate with each other using their private IPs, the communication is free
    - If 2 instances in the same AZ, same subnet, communicate with each other using their public IPv4 addresses, the communication is charged. The data goes to the IGW and back again
    - If we resolve an instance's public DNS name from a VPC, it will resolve to its private IP, it is safe to use it without additional charging form communication

## VPC Peering Cost Management

- Any communication between to VPCs in the same AZ using VPC peering is free of charge
- In case of cross AZ communication using VPC peering, there is cross AZ charge (inbound and outbound)
- When peering regions globally, inter region data charges are applied:
    - Data is charged when it exits from the region
    - The ingress part is free in the destination region