# AWS Cloud WAN

- It is a managed wide-are networking (WAN) service to build, manage and monitor a global network across AWS and on-premises
- It is policy driven, provides a way to configure simple network policies to configure and automate network management
- Provides a central dashboard (through AWS Network Manager)

## AWS Cloud WAN Components

- Global Network: consists of Core Network + Transit Gateway Network
- Core Network consisting of:
    - Core Network Edge
    - Core Network Policy
- Network Segments: routing domains, example: dev network, product network. etc.
    - Segment Actions
- Attachments: represent attachment to components such as VPCs, SD-WAN, S2S VPNs, TGWs
- Peering
- CloudWAN is a global service. All of this configuration data for components is stored in our home region, which is **us-west-2** + our region of choice for CloudWAN

## Core Network Policies

- Everything the we do in CloudWAN is defined by the Core Network Policy (JSON documents)
- Elements of the Core Network Policy:
    - Network Configurations:
        - ASN ranges
        - Regions (Core Network Edge)
    - Segments:
        - Name
        - Regions
        - Require acceptance
        - Sharing between segments
        - Static routes
    - Attachment policies:
        - Rules
        - Association of the attachments to the segments
        - Require acceptance
- Example:
    ```json
    {
        "version: "2021.12",
        "core-network-configuration": {
            "asn-ranges": ["64512-65534"],
            "inside-cidr-blocks": ["100.65.0.0/16"],
            "edge-locations": [
                {
                    "location": "eu-central-1"
                },
                {
                    "location": "eu-west-1"
                }
            ]
        },
        "segments: [
            {
                "name": "sales"
            },
            {
                "name": "testing"
            },
            {
                "name": "iot",
                "isolate-attachments": true"
            },
        ]
    }
    ```
- Network configurations:
    - ASN Ranges (16-32 bit)
    - `edge-locations`: similar to a managed TGW in a region
    - `inside-cidr-blocks`:
        - Required if we connect Connect attachments to the core network
        - Used to setup the GRE tunnels for TGW Connect Attachments
    - ECMP support: optionally can be enabled
- Segments:
    - For each segments we have to provide a name
    - Optionally we can provide an edge location:
        - If we don't provide an edge location to the segment, the segment will be available in all edge locations from the core network
    -`isolate-attachments`: 
        - If set to true, attachment routes are shared automatically
        - Routes can shared through Share segment action or by adding static routes
    - Require attachment acceptance (true/false, by default true):
        - Require acceptance for attachments created by other accounts and try to attach to the WAN
    - Allow/Deny list of the segments:
        - Used to allow or deny segments to share routes
        - We cannot provide both allow and deny lists at the same time
    - Segment Actions:
        - Used to specify for a particular segment with which other segments should be allowed to be shared
        - Used to create static routes

## Attachments

- Attachments types are the following:
    - VPC
    - VPN
    - Connect and Connect Peer:
        - Used to connect to 3rd party virtual appliances hosted inside VPCs
        - Supports both GRE and Tunnel-less protocols
        - Uses BGP for Dynamic routing (two BGP sessions for redundancy)
    - Transit Gateway Route table:
        - After wee peer our TGW with the Core Network, after we can create a route table attachment for the particular network segments
- Attachment policy example:
    ```json
    {
        "rule-number": 200,
        "conidition-logic": "or",
        "conditions": [],
        "actions": {
            "associate-method": "constant",
            "segment": "TestingSegment",
            "require-acceptance": true
        }
    }
    ...
    ```
- Rule number:
    - 1 - 65535
    - Lower number takes priority
    - Rule evaluation stops after fist match
- Require acceptance:
    - Only takes effect when Segment require acceptance is disabled
- Rules have logical conditions: AND/OR
    - AWS Accounts
    - Region
    - TagKey, TagValue
    - ResourceID (example: VPC ID, VPN ID)
- Association method:
    - Constant/Tag
    - If Constant, then we have to provide the segment name

## AWS Cloud WAN and Transit Gateway

- To attach a Transit Gateway to AWS CloudWAN we need to do the following steps:
    1. Create a peering connection between the TGW and the Core Network
        - This peering connection does the peering at switch level
        - When we do the peering, it will also set up the dynamic routing between the TGW and the Core Network
        - It also create a Policy Table for the TGW
    2. Create the Transit Gateway route table attachments with the particular segment
        - With this we connect the VPCs from the TGW to the CloudWAN segment

## AWS Cloud WAN and Direct Connect
- The new capability enables you to directly attach your Direct Connect gateways to Cloud WAN without the need for an intermediate AWS Transit Gateway.

### Limitations for the Direct Connect gateway attachments
- Can't configure static routes, they must be dynamically advertised.
- BGP Communities are not supported.
- Can't configure a list of allowed prefixes to be advertised over the DGW attachment from Cloud WAN to an on-premise network.
- The ASN of a DGW must be outside the ASN range configured for the core network.

### Route propagation
#### Inbound routes
- Via BGP in the segment route-tables.
- Can be routed across all AWS Regions for that segment.
- Cloud WAN follows the route evaluation order for the same prefixes learned over multiple attachments.
#### Outbound routes
- From segment route table to the DGW via BGP.
- Each core network edge association advertises only its local routes.
- The AS_PATH is retained when advertised to on-premise.