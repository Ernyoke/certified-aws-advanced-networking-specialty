# Rout Priority and Selection within AWS

- IPv4 and IPv6 routes are threated separately in AWS
- Route Tables can contain many routes, some routes may apply some don't
- General rule: **Longest Prefix** is always preferred, example: `/32` over `/24`
- If we have equal prefix routes, we have the following priority order:
    1. Static routes (explicitly added routes)
    2. Prefix Lists (randomly chosen if multiple exist. **For single flow of traffic the same route is always used!**)
    3. Propagated routes (routes learned by the route tables):
        - Propagation can be enabled for a route table in a VPC if we use a VGW
        - Routes learned by the VGW (from VPN or DX) are added to the route table
- In case of propagated routes we have to following priority order:
    1. BGP propagated routes - learned from Direct Connect
    2. Static routes added for a Site-to-Site VPN connections
    3. BGP learned routes from Site-to-Site VPN connections
    4. At this point we could have equal dynamic routes learned from BGP, the priority order for these is the following:
        1. Shortest AS_PATH
        2. Lowest multi-exit discriminator (MEDs): this is set at the remote side, it is an indicator for preferred entry/exit point to/from a network
