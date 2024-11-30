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
    - Each pod gets dedicated ENI (branch ENI) mapped to trunk ENI
    - Independent Security Groups can be assigned for each pod
    - Limitations:
        - Trunk and Branch ENIs cannot be used with Windows nodes
        - If the cluster is using IPv6 address family then this feature only works with Fargate nodes
        - Supported by most Nitro based systems (aside from t instance family)

## Exposing Kubernetes Services

- Accessing applications by their pod's IP address is usually an anti-pattern because:
    - Pods are non-permanent objects
    - Pods may be created/destroyed
    - Pods move between the cluster's nodes due to scaling events, node replacement or configuration
- Kubernetes Service is a way to expose an application on a set of pods as a network service
- Kubernetes and EKS support the following Service types:
    - ClusterIP (access services from inside EKS cluster using virtual IPs)
    - NodePort (access services externally using node's static port)
    - LoadBalancer (Network load balancing, access services externally using CLB/NLB Layer 4)
    - Ingress (Application load balancing, access services externally using ALB Layer 7)
- ClusterIP:
    - Default service type in k8s
    - Makes a service reachable  only from withing the cluster
    - Service is exposed on a virtual IP on each node. This IP is not exposed outside of cluster
    - The service virtual IP is assigned from a pool which is configured by setting following parameter in kube-apiserver: `--service-cluster-ip-rage`
    - If not configured explicitly then Amazon EKS provisions either 10.100.0.0/16 or 172.20.0.0/16 range for this virtual IP
    - A kube-proxy daemon on each cluster node defines the ClusterIP to Pod IP mapping in iptables rules
    - Service is accessible with private DNS `<service-name>.<namespace-name>.svc.cluster.local`
- NodePort:
    - NodePort is used to make a Kubernetes service accessible from outside the cluster
    - Exposes the service on each worker node's IP at a static port, called the NodePort
    - One Node port per service
    - Port range: 30000-32767
    - NodePort internally used ClusterIP to route the NodeIP/Port requests to ClusterIP service
    - Not a feasible option to expose service to the outside world!

## EKS Network and Application Load Balancing

- `ServiceType=LoadBalancer`:
    - For Network Load Balancer we should use `ServiceType=LoadBalancer`
    - Originally the LoadBalancer service type in EKS used to be handled by Kubernetes Controller Manager (in-tree cloud controller)
    - Deploys AWS CLB (default) or NLB in instance mode
    - Supports Layer 4 with NLB and Layer 4/7 with CLB
    - Supported by the newer controller called AWS Load Balancer Controller with which we can use NLB (CLB is not recommended to be used)
    - AWS Load Balancer Controller supports registering services both in instance mode an IP mode
    - Each service needs a dedicated NLB => scaling and management is a challenge for huge number of services
- `ServiceType=Ingress`:
    - For Application Load Balancer we should use `ServiceType=Ingress`
    - Handled by AWS Load Balancer Controller
    - Deploys ALB in instance and IP mode for ingress resources
    - Exposes services to the clients outside of the cluster
    - Ingress exposes HTTP and HTTPS routes from outside the cluster o services withing the cluster
    - We can also share ALB with multiple services by using the following annotation: `alb.ingress.kubernetes.io/group.name: my-group`
    - Traffic for IPv6 is supported for IP targets only
- Preserving client IP address:
    - In case of NLB with LoadBalancer service:
        - `externalTrafficPolicy` service spec defines how load-balancing happens in the cluster
            - `externalTrafficPolicy=Cluster`: traffic may be sent to another node, source IP is changed to node's IP address, thereby the IP of the client is not preserved. Load is evenly spread (default policy)
            - `externalTrafficPolicy=Cluster`: traffic is not routed outside of the node and client IP addresses are propagated to the pods. This policy could result in uneven distribution of traffic
    - For ALB ingress service:
        - HTTP header `X-Forwarded-For` is used to get the client IP

## EKS Custom Networking

- We can allocated secondary CIDR to the VPC in the range of 100.64.0.0/16
- The constraint with this CIDR is that the IP addresses are only routable from within the VPC. Traffic from the outside cannot directly go to these addresses
- To be able to use these IP addresses we can enable VPC CNI Custom Networking
- VPC CNI plugin creates a secondary ENI in the separate subnet, the range of the subnet can be from 100.64.0.0/16 range
- Once enabled only IPs from secondary ENI are assigned to pods