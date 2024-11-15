# VGW - Virtual Private Gateway

- Is a gateway object between AWS VPCs and non AWS networks (on-premises, other providers such az Azure, GCP)
- **A VGW can be attached at maximum one VPC at a time**
- They can be detached and reattached to new VPCs, any connection managed by the VGW will be maintained
- VGW can be used with DX (*Private VIF*) directly, with *DX Gateway* and with *Site-2-Site VPN (public IPs)*
- They can also be used as termination points to Site-2-Site VPNs (public IPs)
- VGWs work with BGP, they are given with a private ASN which defaults to `64512`
- VGWs are highly available by default, they operate in multiple AZs in a region
- Used with VPNs we are allocated public IP addresses, these IP addresses are in different AZs => we can create HA architectures
- VGW architecture:

    ![VGW architecture](images/VGWDeepDive.png)

- When we have DX connection connected to VGW, BGP is used to advertise prefixes in both directions
- Using a VGW we also enable route propagation in route tables. Route tables will be dynamically updates with any routes learned by the VGW (**this only works with VGW and not with Transit Gateway(TGW)**)
- Architecturally it is very common to have a VGW with a Site-2-Site VPN as a backup to DX
- A VGW can only be assigned to one VPC, but it can be detached and re-assigned to another VPC. When re-attaching, the configurations of the connections are maintained

## VPN CloudHub

    ![VPN CloudHub](images/VPNCloudHub.png)

- Each of the business sites can communicate with each other using the VGW, creating a virtual hub and spoke network using the VGW
- Requirements to work:
    - Each customer site needs to have an uniq ASN
    - Each VPN needs to end on the same VGW
    - Sites must not have overlapping IP ranges
- We can connect up to 10 CGW
- Can serve as a failover connection between on-premises locations