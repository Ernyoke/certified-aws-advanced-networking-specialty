# NAT - Network Address Translation

- It is a process to designed the growing shortage of IPv4 addresses
- IPv4 address are either publicly routable or private addresses
- Public addresses have to be unique in order to function correctly
- Private addresses can be used in multiple places but can't be routed over the internet
- In order to give internet access to devices from a private network, we can use the NAT process
- Types of NAT:
    - Static NAT: 1 private address to 1 (fixed) public address (IGW in AWS)
    - Dynamic NAT: 1 private address to first available public address
    - Port Address Translation (PAT): many private addresses to 1 public (NAT GW in AWS)
- NAT only makes sense for IPv4, for IPv6 we don't need any form of translation

## Static NAT

- The router maintains a 1:1 NAT table where a private IP address is mapped to a public IP
- Packets are generated as normal with a private source IP and an external destination IP
- The router in the local network is the default gateway
- As the packet passes through the NAT device, the source address is translated to the associated public IP address from the NAT table
- For incoming traffic the destination IP address is updated with the corresponding private IP address a forwarded to the device on the network

![Static NAT](images/NAT1.png)