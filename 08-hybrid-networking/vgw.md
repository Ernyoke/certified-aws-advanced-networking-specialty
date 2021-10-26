# VGW - Virtual Private Gateway

- Is a gateway object between AWS VPCs and non AWS networks (on-premises, other providers such az Azure, GCP)
- A VGW can be attached at maximum one VPC at a time
- They can be detached and reattached to new VPCs, any connection managed by the VGW will be maintained
- VGW can be used with DX (Private VIF) directly and with DX Gateways
- They can also be used as termination points to Site-2-Site VPNs (public IPs)
- VGWs work with BGP, they are given with a private ASN which defaults to 64512
- VGWs are highly available by default, they operate in multiple AZs in a region
- Used with VPNs we get public IP addresses
- VGW architecture:

    ![VGW architecture](images/VGWDeepDive.png)

## VPN CloudHub

    ![VPN CloudHub](images/VPNCloudHub.png)

- Each of the business sites can communicate with each other using the VGW, creating a virtual hub and spoke network using the VGW
- Requirements to work:
    - Each customer site needs to have an uniq ASN
    - Each VPN needs to end on the same VGW