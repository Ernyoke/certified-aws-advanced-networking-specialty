# Advanced EC2 Networking

- EC2 instances are created with a primary ENI (Elastic Network Interface). This cannot be detached or removed
- Additional ENIs can be added at creation time or afterwards
- ENIs should be on the same region in order to be attachable to EC2 instances
- Secondary ENIs can be attached and re-attached to other EC2 instances. EC2 instances can have more than one ENIs attached
- ENIs are tied to AZs. ENIs can be placed on different subnets as long as they remain in the same AZ
- Security Groups are attached to ENIs, not EC2 instances
- Most networking configurations for the EC2 instances are performed on the ENI level, not on the instance level
- Every ENI is allocated a private IPv4 using DHCP. This IP address remains the same for the lifetime of the ENI => every EC2 instance by default has a single primary private IP address that never changes
- This private IP address is exposed to the operating system
- In addition to the primary address, ENIs can also be allocated to one or more secondary private IP addresses. These are also exposed to the operating system of the instance
- IP addresses are linked to the ENI, meaning IPs can be moved between EC2 instances by moving the ENI
- If an instance is launched in a subnet which is set to allocate public IPv4 addresses or if we explicitly decide to launch an instance with a public IPv4 address, the primary ENI is allocated a public IPv4 address. This address is not static, if the instance is stopped/started, this IP is changed.
- The public IP address is not exposed to the operating system, in AWS it is never a valid solution to be exposed
- We can have static public IPs by allocating Elastic IP addresses. These can be associated with an instance (primary ENI) or with an ENI
- If an Elastic IP is associated with an instance, the existing public IP is removed and it is not recoverable
- We can allocate up to one Elastic IP per private IPv4 address. These can be moved between interfaces and instances
- Elastic IP are charged when they are not associated to an instance or if the instance is not running to which they are associated
- ENIs can have one or more public IPv6 addresses, a MAC address and one or more SGs
- If IPv6 is enabled, all IPv6 addresses are publicly routable
- IPv6 are also configured within the operating system network interface
- ENIs can have one or more SGs. SGs are evaluated together, if an allow exists for a specific target, than the traffic is enter/leave the interface
- Each interface has a flag named Source/Destination Check. By default an interface drops traffic if the source or destination  address is not on the interface. If this flag is disabled, this check is not done, traffic wont be dropped. Required to be disabled for NAT instances
- By associating multiple ENIs to the same instance we can:
    - Use different networks for managing the same instance (management or isolated networks, public networks)
    - Use ENIs for software licensing (license based on MAC address)
    - Different network interfaces for security and network appliances
    - Use multiple network interfaces for multi-homed instances with workloads/roles on specific subnets
    - Implement a low budget and simple HA solutions

## EC2 Enhanced Networking (SR-IOV)

- SR-IOV - Single Route IO-Virtualization
- Networking in EC2 is traditionally virtualized
- With VM generally each physical network interface (NIC) is shared between instances, and the hypervisor mediates access between the instances
- Whether an OS is running in a virtualized environment or physical environment, it expects to have direct access to the NIC
- It uses a special type of instruction known as privileged instructions. These means that the os has direct access to the HW
- In case of hypervisor, it looks for these calls and it traps them and redirects them to the physical HW. These process comes with a performance hit
- One way to get around this performance hit is to use device passthrough (1 VM <=> 1 NIC). This limits the ability to migrate the instance and impacts the ability of the VM to cope with other HA events
- Other option is SR-IOV: allows multiple VMs to access the same NIC without performance hits
- This feature is implemented as Enhanced Networking in EC2 and is supported across all types of EC2 instances. It is required for higher performance, especially for certain placement groups
- Device (PCI) Passthrough:

    ![PCI Passthrough Architecture](images/EnhancedNetworking2.png)

- Enhanced Networking Architecture (SR-IOV):
    - The network interface cards are virtually aware
    - Each network card is creating a virtual function (virtual network cards)
    - This functions offer a cut-down feature set enough to send/receive data
    - The management of the overall card is performed by the physical functions offering full functionality
    - Each physical device can offer up to 256 virtual functions
    - The hypervisor is not involved in networking coordination at all, the performance being near to bear-metal
    - Because the hypervisor is not involved, it means better speed, better packets/sec more consistent speeds and less CPU involvement across the board

    ![Enhanced Networking Architecture](images/EnhancedNetworking3.png)

- To enable enhanced networking we can do the following:
    - Use instances with Elastic Network Adapter (ENA):
        - They supports speeds up tt 100 Gbps
        - All instances built on AWS Nitro System use ENA
    - Use Intel 82599 Virtual Function (VF) interface:
        - Supports networks speeds up tp 10 Gbps
        - Following instance types use Intel 82599 Virtual Functions: C3, C4, D2, I2, M4 (excluding m4.16xlarge), and R3

## Network Performance within AWS

- We should use enhanced networking for anything beyond base level performance requirements
- If we have 2 instances communicating over the network, the best performance we can achieve is the lowest common denominator (we cannot have better speeds than the maximum capability of the slower instance)
- Factors which can influence an instance's network speed is the size and type of the instance. Larger sizes generally offer better network performance
- We can have different types of ENIs: older ENI up to 10GBps, newer ENA up to 100Gbps
- Other much performance caps:
    - Instance in the same region: performance is dictated by the lowest performing instance
    - Instances in different regions can have a max aggregate performance bandwidth quota of 5Gbps
    - Single Flow or 5 Tuple: Single Flow is defined as the communication between source and destination IP using the same port and same protocol. Single Flow limit is 5Gbps MAX in case we are not using placement groups. Ways around this: Multi-path TCP (MPTCP), other protocols which support multi-flow transfers

    ![Network Performance within AWS](images/EnhancedNetworking4.png)

## Intel Data Plane Development Kit (DPDK)

- It is a set of libraries and drivers for fast packet processing
- While enhanced networking and SR-IOV reduce the overhead of packet processing between the instance and the hypervisor, DPDK reduces the overhead of packet processing inside the OS
- Provides lower latency due to Kernel bypass
- Offers more control, over packet processing
- Requires lower CPU overhead

## EFA - Elastic Fabric Adapter

- Is a special type of ENI for EC2
- We can have 1 of this type per instance
- Can be added when the instance is launched or when the instance is in a shutdown state
- EFI support **OS Bypass**: provides lower and more consistent latency and higher throughput than the TCP transport; it enhances the performance of inter-instance communication for scaling HPC and ML applications
- EFA removes the OSI stack specification from the communication which in this case gets in the way of performance
- Any HPC or ML application which uses **MPI** or **NCCL** are candidates for using an EFA attached to the EC2 instance
- EFAs provide the traditional networking features are ENIs, but they also provide these OS bypass capabilities
- EFA compared to traditional 7 layer stack:

    ![EFA Compared to OSI](images/EFA1.png)

- EFA architecture:

    ![EFA](images/EFA1.png)

- Limitations of OS Bypass:
    - It is single subnet only
    - Can be used with Linux-based instances only
    - EFAs can be used for cross subnet/AZ works but this will be normal IP traffic
    - OS Bypass traffic cannot be routed
    - Security Group attached to the instances needs an ALLOW ALL, self-referential rule for INBOUND and OUTBOUND traffic

## EC2 Placement Groups

- Allow us to influence EC2 instance placements, insuring that instances are closed together or not
- There are 3 types of placements groups in AWS:
    - **Cluster**: any instances in a single placement groups are physically close
    - **Spread**: instances are all using different underlying hardware
    - **Partition**: groups of EC2 instances which are spread apart (different hardware)

### Cluster Placement Groups

- Used for highest possible performance
- Best practice is to launch all of the instances at the same time which will be part of the placement group. This ensures that AWS allocates capacity in the same location
- Cluster placement groups are located in the same AZ, when the first instance is launched, the AZ is locked
- Ideally the instances in a cluster placement group are located on the same rack, often on the same EC2 host
- All instances have fast bandwidth between each other (max 10Gbps vs 5Gbps which can be achieved normally)
- They offer the lowest latency possible and max PPS possible in AWS
- To achieve these levels of performance we need to use instances with high performant networking: instances with more bandwidth and with Enhanced Networking
- Cluster placement groups should be used for highest performance. They offer no HA and very little resilience
- Considerations for cluster placement groups:
    - We can not span AZs, the AZ is locked when the first instance is launching
    - We can span VPC peers, but this will impact performance in a negative way
    - Cluster placement groups are not supported for every instance type
    - *Recommended*: use the same type of instances and launch them at the same time
    - *Recommended*: launch instances at the same tme
    - Cluster placement groups offer 10 Gbps for single stream performance

### Spread Placement Groups

- They offer the maximum possible availability and resiliency
- They can span multiple AZs
- Instances in the same spread placement group are located on different racks, having isolated networking and power supplies
- There is a limit for 7 instances per AZ in case of spread placement groups
- Considerations:
    - Spread placement provides infrastructure isolation
    - Hard limit: 7 instances per AZ
    - We can not use dedicated instances or hosts

### Partition Placement Groups

- Similar to spread placement groups
- They are designed for situations when we need more than 7 instances per AZ but we still need separation
- Can be created across multiple AZs in a region
- At creation we specify the number of partition per AZ (max 7 per AZ)
- Each partition has its own rack with isolated power and networking
- We can launch as many instances as we need in a partition group
- Use cases for partition groups: HDFS, HBase, Cassandra, topology aware applications
- Instances can be placed in a specific partition or we can let AWS to decide

## Instance Metadata

- Is a service provider by AWS to the EC2 instances
- It is data about the instance used to configure/manage the instance
- It is accessible from all the instances by accessing `http:/169.254.169.254` address
- Access latest metadata for the instance: `http:/169.254.169.254/latest/meta-data`
- Information that can be accessed by using instance metadata:
    - Information about the environment
    - Networking
    - Authentication
- Metadata is also used to grant access to user data
- The metadata service in not encrypted and it not requires authentication

## Network I/O Credits

- Instance families such as R4 and C5 use a network I/O credit mechanism
- Most application do not consistently need a high network performance
- These instances perform well above baseline n/w performance during peak requirement
- We should make sure that we consider the accumulated n/w credits before doing performance benchmark for instances supporting network I/O credits mechanism
