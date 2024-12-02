# AWS Global Accelerator

- Designed to optimize the flow of data from user to AWS
- Similar to CloudFront, both improve performance when communicating with services hosted in AWS
- Global Accelerator provides 2 **anycast** IP addresses. Anycast IP addresses are special type of IP addresses
- Normal IP addresses are called **unicast** IP addresses, these refer to one device in the network
- In contrast anycast IP addresses can be used by multiple devices at the same time, the traffic will be routed to the device closest to the source
- Global Accelerator uses the AWS Edge Locations. Since multiple locations advertise the given anycast IP addresses, the traffic will be routed to the Edge Location closer to the user
- AWS has its own dedicated network consisting in fiber links. Edge Locations relay traffic to this network improving performance

## CloudFront vs Global Accelerator

- CloudFront moves the content closer to the customer by caching it on the Edge Location. Global Accelerator moves the customer closer to the service pe providing access to the AWS global network
- Global Accelerator is a network product: works on any TCP/UDP applications including web apps (HTTP/HTTPS). CloudFront only caches HTTP/HTTPS content
- Global Accelerator does not cache anything. It does not understand the HTTP/HTTPS protocol

## Custom Routing Accelerator

- Custom routing accelerator lets us use our own application logic to route user traffic to a specific Amazon EC2 instance destination in a single or multiple AWS Regions
- Because the traffic is routed over the AWS global network, we get all the performance improvements of going through Global Accelerator
-  A custom routing accelerator is an alternative to the standard accelerator, which automatically routes traffic to a healthy endpoint that is nearest to our users
- Because standard accelerators are designed to load balance traffic, we can't use them to route users to a specific EC2 instance destination behind our accelerator. Being able to deterministically route traffic is required for some interactive applications such as multi-player gaming, EdTech, social media, and real-time communications
- With a custom routing accelerator, we direct multiple users to a unique port on our accelerator. The accelerator maps each port on our accelerator to a specific destination, an EC2 instance private IP address and port, so it can route our traffic there. This mapping makes it easier for us to integrate Global Accelerator with our application logic, such as matchmaking servers or session border controllers (network elements that protect and regulate IP traffic flows for real-time communication workflows)
- With custom routing accelerators, we can leverage Global Accelerator as the single point of entry for our application while deterministically sending our user traffic to specific EC2 destinations in any AWS Region