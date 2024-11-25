# BGP - Border Gateway Protocol

- BGP is a routing protocol
- Used to control how data flows from point A through point B and C and arrives to the destination point D
- BGP is made up from a lot of self managing networks know as Autonomous Systems (AS)
- AS could be a large network, collection of routes etc. and is viewed as a black box from BGP perspective
- Each AS is allocated a number by IANA, named ASN
- ASNs are 16 bit in length and range from 0 to 65535, the range from 64512 to 65534 is private
- ASNs are used by the BGP to identify different entities on the network
- BGP is designed to be reliable and distributed, and it operates of TCP/179
- It is not automatic, the communication between two different AS should be done manually
- AS do exchange network topology information between them
- BGP is a path-vector protocol: it exchanges the best path to a destination between peers, the path is called **AS_PATH** (Autonomous System Path)
- **iBGP** - internal BGP, routing within an AS
- **eBGP** - external BGP, routing between AS's
- BGP always choses the shortest path. There are ways to influence the path by artificially expending the path (prepending itself to the path)

## BGP Path Selection

- BGP is designed to pick the best path to a given destination that it knows about
- Always starts with the most specific route wins, often this is enough
- But, also, we often run into the situation where we have multiple different routes through to a destination with the same prefix size. If we are using BGP, it has a fairly extensive way of picking route priority
- BPG Path Selection decision points (from the MOST preferred, to the LEAST preferred):
    1. **Weight**: CISCO proprietary, it is a value local to the router. Default is 0. Highest weight is selected as the most preferred
    2. **Local Preference** (`LOCAL_PREF`): we can set this property on a specific route, this value will be distributed inside the AS. Default value is 100, highest value is preferred
    3. **Locally Originated**: prioritizes any path which originate locally
    4. **AS Path Length** (`AS_PATH`): list of AS to destination. Shortest is preferred. To deprioritize any route we can do path pre-pending
    5. **Origin code**: prefer lowest Origin Code (IGP, EGP, Incomplete)
    6. **Multi-Exit Discriminator** (`MED`): represents the external metric of a route. Preference is given to any route with a lower MED value
    7. **eBGP over iBPG**: prefer eBGP over iBGP paths
    8. **Shortest IBGP Path to next hop**: prefer the path with the shortest number of hops to the exit (within an AS)
    - At this point if the router supports multiple routes, we can inject both/all of them, otherwise consider the next steps
    9. **Oldest Path**: suggests stability
    10. **Router ID**: prefer the path with the lowest router ID. Router ID is derived from the highest interface address
    11. **Neighbor IP address**: prefer the lowest neighbor IP

## Local Preference

- Allows us to chose outbound paths to a given destination
- It is a feature used for outbound traffic, it is internal only
- It is exchanged internally with routers within the AS
- It does win against `AS_PATH` for route choice
- By default every path has a `LOCAL_PREF` of 100. The path with the highest value is preferred

## MED - Multi-Exit Discriminator

- It is a way for network administrators to influence which path will be used for given routes that we advertise
- It is used when an AS has multiple links to another AS
- We can use it to influence the routing behavior of a neighbor without having any arrangement with it
- `ASPATH` takes priority over MED => is only used when we have paths with equal lengths
- The route with the lowest MED path win, 0 being the lowest
- Many routers view 0 as default MED => we should use MED to deprioritisze a route by increasing its value
- MED affects incoming traffic, not outgoing
- MED is advertised to neighbors only, it is not transitive

## Does Local Preference and MED Matter for AWS?

- AWS recommends to use `LOCAL_PREF` prioritize a specific DX connection leaving on-prem to AWS
- We should use `AS_PATH` prepending to get AWS to preference a specific DX for inbound traffic for the on-prem
- AWS sets an MED of `100` on VPN BGP Sessions - meaning the default of 0 of DX is preferred
- If we are using a VGW for both the VPN and DX, these will use the same ASN number, so MED will work
- If we use a TGW in our architecture, we will potentially have different ASNs involved, so MED wont be use to select between different advertisements, unless we use the `bgp always-compare-med` on the customer router