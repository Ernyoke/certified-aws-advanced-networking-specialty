# Kubernetes and EKS Networking

## Kubernetes Architecture

- Kubernetes building blocks:
    - Kubernetes cluster consists of:
        - Control Plan: 
            - Controls what happens in the cluster and the data plane
            - Consists of a set of control processes hosted in the master node
        - Data Plane:
            - Actual applications run in the data plane
            - Consists of a set of worker nodes called Nodes
            - Applications are hosted on the Nodes inside Pods
            - Pod:
                - It is a deployment unit
                - It consists of one ore more containers
- Control Plane components:
    - kube-apiserver:
        - Exposes Kubernetes APIs
        - It is a frontend for the Kubernetes control plane
    - etcd:
        - It is a key-value store
        - Used to maintain the state of the cluster data
    - kube-scheduler:
        - Watches for newly created pods with no assigned node
        - Selects a node for the new pods to run on
    - kube-controller-manager:
        - Runs controller processes such as Node controller, Replication controller, Namespace controller, Job controller, EndpointSlice controller, Service controller for cloud load balancer, etc.
- Data Plane components:
    - Node: hosts the pods
    - kubelet:
        - An agent that runs on each node in the cluster
        - Makes sure that containers are running in a pod
    - kube-proxy:
        - Enables network communication to pods from network sessions inside or outside of the cluster
    - Container Runtime:
        - Responsible for running containers
        - Kubernetes supports container runtimes such as containerd (Docker), CRI-O and any other implementation of the Kubernetes CRI

## EKS Cluster Networking

- EKS has the control plane residing in an AWS managed VPC
- The data plane is launched in the customer VPC managed by our account
- EKS provisions 2-4 ENIs in the customer VPC to enable the communication between the control plane and the VPC
- It is recommended to have a separate subnet for EKS ENIs. EKS needs at least 6 IP addresses in each subnet (smallest subnet with 16 IPs is recommended)
- EKS creates and associates a security group to these EKS owned ENIs. These security groups are also attached to the nodes if we create our nodes as part of managed node groups
- Kubernetes API server can be accessed over the internet (by default)
- EKS allows assigning IPv4 and IPv6 IP addresses to pods, but not in a dualstack mode
- Public EKS cluster endpoint access:
    - This is the default option for EKS
    - EKS cluster endpoint can be reached by anyone over the internet. We can restrict this access with using allowed CIDR blocks
- Public and Private EKS cluster endpoint access enabled:
    - If both public and private access is enabled, the control plane's API can be accessed both from the internat and from the private VPC serving the data plane
- Private cluster endpoint access:
    - In this case, the cluster endpoint can be reached only from the private VPC

## EKS API Access

- Should not be confused with EKS control plane API. EKS API is used for creating/destroying clusters
- From being accessible from a private VPC, AWS provide connectivity to it via PrivateLink

## EKS POD Networking - CNI

- CNCF network specifications for Kubernetes:
    - Every pod gets its onw IP address
    - Containers in the same pod share the network IP address
    - All pods can communicate with all other pods without using network address translation (NAT)
    - All nodes can communicate with all pods without NAT
    - The IP that a pod sees itself as is the same IP that other see it as
- All these specification should be implemented as a CNI - Container Network Interface
- Amazon's implementation of CNI is called VPC CNI plugin
- Amazon VPC CNI plugin does the following:
    - Creates and attaches ENIs to worker nodes
    - Assigns to ENIs secondary IP addresses and also assigns these IP addresses to pods
- Amazon EKS officially supports the Amazon VPC CNI plugin for Kubernetes
- Maximum pods per node: 
    - Different types of EC2 instances support different number of network interfaces
    - The number of IPv4 and IPv6 addresses per ENI can also differ based on the instance type (usually bigger instance types support more ENIs, more IP addresses per ENI)
    - Number of max pods: total number of ENIs * (max number of IPs per ENI -1) + 2
        - -1 primary IP address is not assigned to the pods
        - +2 => k8s has 8 ports, one for host networking and one for kube proxy on every node
- To increase the available IP addresses for pods:
    - AWS offers a feature which would increase the number of IP addresses beyond the maximum (see above)
    - This feature is only supported for AWS Nitro-based nodes
    - It is called prefix delegation:
        - We can assign a CIDR block to an EC2 instance (/28 for IPv4 and /80 for IPv6)
    - Kubernetes recommends max number of 110 per node
- Assigning IPv6 addresses to pods:
    - Supported with AWS Nitro-based instances and Fargate
    - By default, Kubernetes assigns IPv4 addresses to pods, but we can also configure the cluster with IPv6 addresses
    - EKS does not support dual-stack
    - For Amazon EC2 nodes, we must configure the Amazon VPC CNI add-on with IP prefix delegation and IPv6
    - We must assign IPv4 address to VPC and subnets as VPC does require IPv4 address to function
    - IPv6 addresses are not supported for Windows pods
- Pods within the same VPC can communicate directly using pod's IP address
- Pod to external network communication:
    - When a pod communicates to any IPv4 address that isn't within a CIDR block of the VPC, the VPC CNI plugin translates the pod's IPv4 address to **the primary ENI's primary IPv4 address** of the node the pod is running on (default setting)
    - For IPv6 address family, this translation is not applicable, because all IPv6 addresses are public
    - If the pod is in a public VPC and enable source NAT (`EXTERNALSNAT=false`). This means node's public IPv4 address will be used
    - For private subnet we can still disable the source NAT (`EXTERNALSNAT=true`). In this case, the IP address of the VPC NAT will be visible four applications outside of the VPC

## Mutli-home Pods with Multus CNI

- With multi-home pods we can have multiple ENIs for a single pod
- This feature is supported by VPC CNI by default

## Security Groups in EKS

- When we create an EKS cluster, it will create and associate an SG with the following rules:
    - Inbound: All allowed from Self
    - Outbound: destination to all IPv4/IPv6 is allowed
- This SG is associated with:
    - ENIs created by EKS in customer VPC
    - ENIs of the nodes in managed node group
- At minimum the following outbound rules should be required:
    - TPC 443 - required for Kubernetes API server
    - TCP 10250 - required for the kubelet API
    - TCP/UDP 53 - required for DNS
- The problem with SGs attached to nodes:
    - SGs being attached to node ENIs means that all the pods will share the same SG
    - This is a drawback if we need different SG rules for different kind of pods
    - A solution for this is to use a Network policy engine like Calico which provides network security policies to restrict inter pod traffic using iptables
    - Another solution is to use **Trunk and Branch ENIs**
- Trunk and Branch ENIs:
    - Amazon EKS and ECS support Trunk and Branch ENI feature
    - A VPC resource controller add-on named `amazon-vpc-resource-controller-k8s` managed Trunk and Branch Network Interfaces
    - When `ENABLE_POD_ENI=true`, VPC resource controller creates special network interface called a trunk network interface with description "aws-k8s-trunk-eni" and attaches it to the node
    - The controller also creates branch ENIs with description "aws-k8s-branch-eni" and associates them with the trunk interface