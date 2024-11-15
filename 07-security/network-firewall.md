# AWS Network Firewall

## Components

- **Firewall**: defines the VPC that it is protecting, subnets for the firewall endpoints and Firewall Policy
- **Firewall Policy**:
    - Configuration entity for Network Firewall
    - Defines the filtering behavior
    - It is a 1:M relationship between the Firewall Policy and the Firewall. A policy can be used on many Firewalls, but one Firewall can have only one Firewall Policy
- **Rule Groups**:
    - They are defined by the Firewall Policy
    - They are collection of rules
    - They are either stateless or stateful
    - Rules groups contain a **processing order** and a **default action**
- **Rules**: evaluate the traffic and decide to pass, drop or forward it

## Traffic Processing with Network Firewall

- At first we define a Firewall: set a name and description, specify VPC and subnets, define logging options and tags
- Finally we define the Firewall Policy that will be used by the Firewall
- A Firewall Policy is a discrete unit of configuration. It can be applied to many Firewalls (1:M attachment)
- Inside the Firewall policy we define Rule Groups
- Based on what rules we define, the traffic will flow through 2 types of rule engines:
    - Stateless Engine:
        - Inspects traffic statelessly, simply looking at individual packets
        - It checks against 5-Tuple rules: protocol, src/dst IPs, src/dst ports
        - The engine can drop of pass the traffic. Dropped traffic never exists the firewall, passed traffic leaves the Firewall VPC endpoint
        - Default rule: if there is no explicit rule matched, traffic will match this rule. Usually, this will forward traffic to a stateful processing engine
    - Stateful Engine:
        - Rules in this engine are ordered by action: PASS -> DROP -> ALERT RULES
        - The Firewall Policy configures all these rules

## Rule Groups

- Rule groups contain rules that can be customer created or AWS managed
- Rule groups are defined upfront as stateful or stateless. This is the type of the Rule Group
- We also have to define the **Capacity**: limit on the processing requirements for the group (all the rules in the group)
- Capacity cannot be changed afterwards!
- Types of Rule Groups:
    - Stateless:
        - Inspects each packet in isolation, it is not state aware
        - Request and response components are different from the Rule Groups' point of view
        - Similarly to NACLs, we have to have separate rules for requests and responses => INBOUND and OUTBOUND traffic is treated differently
        - Work with 5-Tuple based checks: protocol, src/dst IPs, src/dst ports
        - Actions for stateless rules can be: Pass, Drop, Forward and Custom
        - Rules have a priority. Lower numbered rule has the highest priority
    - Stateful:
        - Uses an open standard named **suricata** for its rule engine
        - The default rule is Pass. Order of processing is pass, drop, alert
        - When we define rules we can use the suricata format for all different types if rules or the builtin standard for simple/domain lists
        - Simple stateful rules are 5-Tuple rules, but with a state awareness (similar to Security Groups)
        - Domain lists allow us to define good or bad lists of domains including protocol types
        - For domain lists there is support for individual domains, wildcards, and there is support for headers in case of HTTP or SNI for HTTPS
        - For **IPS Rules** we need to use the suricata compatible format

## Firewall Logging

- We can configure AWS Network Firewall logging for our firewall's stateful engine
- Logging gives us detailed information about network traffic, including the time that the stateful engine received a packet, detailed information about the packet, and any stateful rule action taken against the packet
- The logs are published to the log destination that we've configured, where we can retrieve and view them
- Firewall logging is only available for traffic that we forward to the stateful rules engine. We forward traffic to the stateful engine through stateless rule actions and stateless default actions in the firewall policy