# URL Filtering in a VPC

- Goal is to filter URLs in a VPC => we need a L7 device, so NAT instance and NAT gateway wont work, because they don't have L7 awareness
- Application Load Balancer are designed for incoming traffic, we can not use them to filter outgoing traffic. We cannot use them
- We cannot use Security Groups, they are L5 devices, no URL awareness
- We cannot use NACLs, they are L4
- We can use a self-managed proxy server (L7) which can act as a man-in-the-middle
- Architecturally we can create a SG for our instance fleet and we can also create an SG for the proxy server. We can configure the SG for the proxy to allow all outbound for TPC 80/443 traffic. We also allow inbound communication for the proxy on TCP 3128 port, we allow this port for outbound traffic as well for the SG of the instances