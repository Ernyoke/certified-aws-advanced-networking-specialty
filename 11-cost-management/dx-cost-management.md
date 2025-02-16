# DX Cost Management

## Transit Gateway Billing

- For a single region:
    - TGWs are billed per an hourly charge per attachment. The account, who owns the entity which is attached to the gateway, will be charged
    - TGWs are also billed on per a GB processed data. The account who owns the TGW is charged
    - For VPN attachments there are additional VPN charges. These are same charges which would apply to a VPN connected to Virtual Private Gateway
- For TGW peering cross region:
    - Peering attachment is also billed as any other attachment
    - When crossing the region boundary, there is an extra charge for data sent from one region to another. Billing only happens at the sending side

## Direct Connect Billing

- There is an hourly fee for the DX port allocation based on the speed of the DX port and location
- Any data received from the VPCs form AWS has a per GB fee associate to it
- The cost of data center depends on the cost region and destination region
- The data transfer rate is often cheaper than using the public internet (and obviously faster)
- DX Gateway is a free service, no additional costs will apply if there is a DX Gateway provisioned
