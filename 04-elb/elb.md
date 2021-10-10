# ELB - Elastic Load Balancer

- Currently there are 3 different type of load balancers available in AWS
- They are split between v1 (deprecated) and v2 (preferable)
- Load balancer product started with the Classic Load Balancer (CLB), this a v1 product introduces in 2009
- CLBs can load balance HTTP/HTTPS and also lower layer protocols, but they are not considered layer 7 devices
- CLBs can not really understand HTTP, they lack many features and they have a limitation supporting 1 SSL per CLB
- Application Load Balancer (ALB) - v2 product, layer 7 load balancer supporting HTTP/HTTPS/WebSocket protocols
- Network Load Balancer (NLB) - v2 product, layer4 load balancer supporting TCP, TLS and UDP
- In general v2 load balancers are faster, cheaper and they support target groups and rules

## ELB Architecture

- It is the job of the load balancer to accept connection from an user base and distribute it to the underlying services
- ELBs support many different type of compute service
- LB architecture:

![LB Architecture](images/ELBArchitecture1.png)

- Initial configurations for ELB:
    - IPv4 or double stacking (IPv4 + IPv6)
    - We have to pick the AZ which the LB will use, specifically we are picking one subnet in 2 or more AZs
    - When we pick a subnet, AWS places one or more load balancer nodes in that subnet
    - When an LB is created, it has a DNS A record. The DNS name resolves all the nodes located in multiple AZs. The nodes are HA: if the node fails, a different one is created. If the load is to high, multiple nodes are created
    - We have to decide on creation if the LB is internal or internet facing (have public IP addresses or not)
- Listener configuration: what the LB is listening to (what protocols, ports etc.)
- An internat facing load balancer can connect to both public and private instances
- Minimum subnet size for a LB is /28 - 8+ fee addresses per subnet (AWS suggests a minimum of /27)