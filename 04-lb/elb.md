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
- Listener configuration: controls what the LB is listening to (what protocols, ports will be accepted at the listener side of the load balancer)
- An internat facing load balancer can connect to both public and private instances
- Minimum subnet size for a LB is /28 - 8+ free addresses per subnet (AWS suggests a minimum of /27)

## Cross-Zone Load Balancing

- Initially each LB node could distribute traffic to instances in the same AZ
- Cross-Zone Load Balancing allows any LB node to distribute connections equally across all registered instances in all AZs
- For ALB cross-zone load balancing is enabled by default

## Other Architectural Considerations

- When an ELB is provisioned, we see it as one device which runs in 2 or more AZs
- What actually is created is one ELB node per subnet in each AZ that the LB is configured in
- We are also creating a DNS record for that LB which spreads the incoming requests through all the active nodes for the LB
- Nodes in one subnet per AZ can scale
- Load balancers comes in 2 types:
    - Internet facing: means the nodes have a public IPv4 address
    - Internal: nodes have only private IP addresses
- An internet facing load balancer can communicate to private instances within a VPC. These instances don't need public IP addresses
- A load balancers requires 8 or more free IP addresses per subnet, and a /27 subnet to allow scaling

## Application and Network Load Balancers

- Consolidation of load balancers:
    - Classic load balancers do not scale, they do not support multiple SSL certificates (no SNI support) => for every application a new load balancer is required
    - V2 load balancers support rules and target groups
    - V2 load balancers can have host based rules using SNI
- **Application Load Balancer (ALB)**:
    - ALB is a true layer 7 load balancer, configured to listen either HTTP or HTTPS protocols
    - ALB can not understand any other layer 7 protocols (such as SMTP, SSH, etc.)
    - ALB requires HTTP and HTTPS listeners, it cannot be configured with TPC/UDP/TLS listeners
    - It can understand layer 7 content, such as cookies, custom headers, user location, app behavior, etc.
    - Any incoming connection (HTTP, HTTPS) is always terminated on the ALB - no unbroken SSL
    - All ALBs using HTTPS must have SSL certificates installed
    - ALBs are slower than NLBs because they require more levels of networking stack to process
    - ALB offer health checks evaluation at application layer
    - Application Load Balancer Rules:
        - Rules direct connection which arrive at a listener
        - Rules are processed in a priority order, default rule being a catch all
        - Rule conditions: `host-header`, `http-header`, `http-request-method`, `path-pattern`, `query-string` and `source-ip`
        - Rule actions: `forward`, `redirect`, `fixed-response`, `authenticate-oidc` and `authenticate-cognito`
    - The connection from the ALB and the instance is a separate connection
- **Network Load Balancer (NLB)**:
    - NLBs are layer 4 load balancers, meaning they support TCP, TLS, UDP, TCP_UDP connections
    - They have no understanding of HTTP or HTTPS => no concept of headers, cookies or session stickiness
    - They are really fast, they can handle millions of request per second having 25% latency of ALBs
    - Recommended for SMTP, SSH, game servers, financial apps (not HTTP(S))
    - Health checks can only check ICMP/TCP handshake
    - They can be allocated with static IP addresses, useful for whitelisting
    - They can forward TCP straight through the instances => unbroken encryption
    - NLBs can be used for PrivateLink

## Connection Draining and Deregistration Delay

- Connection draining is a setting which controls what happens when instances become unhealthy or deregistered
- Normally all connections are closed and the instance receives not new connections
- Connection draining allows in-flight requests to complete before the instance being terminated, while no new connection is sent to that instance
- It is a way of gracefully removing connections in order to reduce disruptions
- **Connection draining is supported in Classic Load Balancer only!**
- Connection draining is a timeout value between 1 and 3600 seconds (default: 300)
- If an instance is taken out of service, it is listed in "InService: Instance deregistration in progress"
- If we are using an ASG, it will wait for all connections to complete or for the timeout, whichever occurs first
- **Deregistration Delay: same feature as connection draining supported on NLB, ALB and Gateway Load Balancers (GWLB) with subtle differences**
- Deregistration Delay is defined on target groups, not on load balancers themselves
- It is enabled by default, default value being 300 and having a range between 0 and 3600 seconds

## `X-Forwarded-For` and `PROXY` Protocol

- Clients connects to load balancers and load balancers connect to backend services
- Load balancers tend to mask the IP address of clients who are talking with our backend services
- `X-Forwarded-For`:
    - In order to pass the IP address to the backend service, we can use `X-Forwarded-For` header
    - `X-Forwarded-For` is a HTTP header, which is appended by proxies/load-balancers to the request
    - `X-Forwarded-For` can contain a list of IP addresses, containing all the addresses from which the requests are redirected
    - This header is supported by CLB and ALB
- `PROXY` protocol:
    - For non HTTP/S communication we can use the `PROXY` protocol for the same purpose
    - `PROXY` protocol is an additional layer 4 TCP header
    - There are two versions of `PROXY` protocol: v1(human readable, supported by CLB) and v2 (binary encoded, supported by NLB)
    - Common usage for `PROXY` protocol: unbroken encryption (HTTPS)

## Load Balancer Security Policies

- A security policy is a set of ciphers and protocols which we configure on the listener of the LB
- Defines which ciphers and protocols the LB can use
- Protocol ensures secure client-server communication. A protocol can use many ciphers
- Cipher is an algorithm to encrypt/decrypt data
- Client and server present ciphers/protocols which they support. From the overlap between those 2 the best supported one is picked
- In case of the LB we control the policy between the client and LB. An AWS chosen one is used between the LB and the targets (example: `ELBSecurityPolicy-2016-08`)
- Newer policies are more secure, but they are usually less compatible, especially with older browsers
- If we must ensure that we have forward secrecy, we can chose an alternate: `ELBSecurityPolicy-FS`
