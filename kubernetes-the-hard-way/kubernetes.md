# VPC networks

a virtual private cloud network is a virtual version of a physical network.
Properties of a VPC:

- subnets regioanlly segmet the network IP space into prefixes (subnets) and control which prefix an instance's internal IP address is allocated from
- traffic to and from instances can be controlled with network firewall rules
- resources within a VPC network communicate with one another by using internal (private) IPv4 addresses
- network administration can be secured by using IAM roles
- an org can use a shared VPC to keep a VPC network in a common host project. Teams in the same org can create resources that use subnets of the shared VPC network
- VPC networks can connect to other VPC networks in different projects or orgs by ysing VPC peering

# networks and subnets

## Subnet

Each VPC network consists of one or many IP range partitions. These IP range partitions are called subnets.
For instance, when we create resource in Google Cloud, we choose a network and subnet. Subnets are regional objects

## Classless Inter-Domain Routing (CIDR)

system of defining the network part of an IP address, It allows a way to break IP networks down more flexibly

## IPv4 exhaustion

## Egress and Ingress

Egress and ingress refer to outgoing and incoming traffic in the network. A VPC network has implied firewall rules, we can overwrite these rules.

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. An ingress controller is needed to satisfy an ingress resource. Typically, an ingress controller is a load balancer

load balancers route http/s requests to services

## Layer-3 network

In the Open Systems Interconnection (OSI) standards - a layer-3 network refers to the networking layer.
The network layers works the transmission of data from one host to the other. It takes care of packet routing (selection of shortest path to transmit the packet from the routes available).
Routing: the network layer protocols determine which route is suitable from source to destination. This function of the network layer is known as routing
Logical Addressing: In order to identify each device on internetwork uniquely, the network layer defines an addressing scheme. The sender

A segment in the network layer is known as a packet
Network layer is implemented by networking devices such as routers

Transport Layer is operated by the OS. It communicates with the application layer by making system calls
source: https://www.geeksforgeeks.org/layers-of-osi-model/

## Flannel

Flannel is an agent that provides a layer 3 IPv4 network between multiple nodes in a cluster. It does not control how containers are networked to the host, only how the traffic is transported between hosts. Flannel uses K8s to store the network configuration and then allocates a subnet lease to each host.
K8s assume each container (pod) has a unique routable IP inside the cluster. The advantage of this model is that it removes port mapping that come with sharing a single host IP.
see: https://github.com/coreos/flannel#flannel

## Router v. switch

Router
Layer 3 device. Used for networks that are not alike, networks that need translation to connect over.
When we define boundaries, routers allow for layer to layer communication
Routers connect any type of technology via interface that we enable, allows devices to flow communication back and forth

Switch
A layer 2 device. Designed to connect a number of clients (physically has a lot of ports). Any that connects to a switch, you're looking at an ethernet style connection
Switches are used to connect many computers together in a single area like in a business or at a school. local area networks (LAN)

## Internet Control Message Protocol (ICMP)

A network layer protocol used by network devices, like routers, to send error messages and operational information indicating success or failure when communicating with another IP address.
There is no port number associated with ICMP packets, whereas TCP and UDP are transport layer protocols and have a port number associated with their protocol.
Ex: an error is indicated when a requested service is not available or a host can't be reached. ICMP is not like TCP or UDP because it is not used to exchanged data between systems

## TCP/UDP Network Load balancing

network load balancing directs TCP or UDP traffic across VM instances in the same region in a VPC network.
Network load balancers as a managed service, they aren't proxies, responses from the backend VMs go directly to clients, and not back through the load balancer, preserves the source IP addresses of packets

- revisit: https://cloud.google.com/load-balancing/docs/network/
- for visuals: https://cloud.google.com/load-balancing/images/network-load-balancer.svg

## Internet access requirements with a VPC

- the network must have a valid default gateway route whose destination IP range is general `0.0.0.0/0`. This route defines the path to the internet
- Firewall rules must allow egress traffic from the instance
- Either the instance must have an external IP address. (An external IP address can be assigned to an instance when it is created or after it's been created). Or the isntance be able to use NAT or an instance-based proxy that is the target for a static `0.0.0.0/0` route.
- see https://cloud.google.com/vpc/images/vpc-overview-example.svg

## Network Policies

By default, pods accept traffic from any source. Pods become isolated by defining a network policy.

When a network policy is defined, only that policy applies, the pod will reject all other connections. If multiple policies select a pod, the pod is restricted by the union of the policies. The order of evaluation does not affect the policy result.

Examples of polciies:
default deny all ingress/egress traffic

## Network address translation (NAT)

What is NAT?
NAT uses a router to act as an agent between the internet (public network) and local network (private). This means a single unique IP address can represent a cluster of computers.

Another explanation:
Network address translation is a method of remapping one IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic routing device

How does NAT work?
NAT usually connecting two networks together, and translates the private addresses in the internal network into a legal address, before packets are forwarded to another network.

see: https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/26704-nat-faq-00.html

- In Kubernetes, pods on a node can communicate with all pods on all nodes without NAT
- agents on a node (daemons) can communicate with all pods on that node
- see: cluster networking kubernetes

## containerd

Manages complete container lifecycle in the host system:

- Image transfer and storage
- container execution and supervision
- low-level storage
- network attachments

A container runtime, it is various kernel features tied together
Containerd is a layer of abstraction between management code and the syscalls and OS specific functionality, so that we can run containers on any OS, linux, windows etc (we don't have to deal with underlying OS details).

```
Things like networking are out of scope for containerd.  The reason for this is, when you are building a distributed system, networking is a very central aspect.  With SDN and service discovery today, networking is way more platform specific than abstracting away netlink calls on linux.  Most of the new overlay networks are route based and require routing tables to be updated each time a new container is created or deleted.  Service discovery, DNS, etc all have to be notified of these changes as well.  It would be a large chunk of code to be able to support all the different network interfaces, hooks, and integration points to support this if we added networking to containerd.  What we did instead is opted for a robust events system inside containerd so that multiple consumers can subscribe to the events that they care about.  We also expose a task API that lets users create a running task, have the ability to add interfaces to the network namespace of the container, and then start the container’s process without the need for complex hooks in various points of a container’s lifecycle.
```

source: https://www.docker.com/blog/what-is-containerd-runtime/

## node authorizer

the node authorizer allows a kubelet perform API operations
Read ops:

- services
- endpoints
- nodes
- pods

## Certificate Authority and Certificates

TLS certificates are needed for verifying the authority of requests over the network. Alongside SSL, the certificates are used to check the integrity of data

## Kubernetes Components

see: https://kubernetes.io/docs/concepts/overview/components/

### kubelet

The kubelet is a node agent that runs on each node.
Registers the node using the apiserver
Configured using a podspec and ensures the container described is healthy

### kube-controller-manager

A daemon that watches the shared state of the cluster through the api server makes changes attempting to move the current state closer to the desired state.
Controllers include: replication controller, endpoints controller, namespace controller, and serviceaccounts controller
control loop is non-terminating loop that regulates the state of the system.

### kube-scheduler

workload-specific function, takes into account

### kube-apiserver

Exposes REST operations and servers as the frontend to the clsuter's shared state through which all components interact

## Service Accounts

A cluster is either a service or user cluster. If a service cluster, the processes run in pods
Service Account Admission Controller:

- ? (a few short sentences)
  Token Controller
- ? (a few short sentences)
  Service account Controller
- ? (a few short sentences)
  see: https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/
  A partition is a space crafted out of disk
  A volume is a partition that has been formatted into a filesystem

## Extensions and details included in SSL certificates

https://www.ibm.com/support/knowledgecenter/en/SSS28S_3.0.0/com.ibm.help.forms.doc/Designer_User_Manual/i_bpfd_g_certificate_attributes.html

## runc

a CLI tool for spawning and running containers according to the OCI specification (a standards specification). This tool can handle creating, starting, and deleting containers such as Docker.

## Kubelet TLS bootstrapping

## kubectl

## federation

## persistent sessions

## dynamic weights

## linux filesystem

- /var/lib
  - holds state information (data that programs modify while they run, and pertains to one specific host) about an application or the system. We should never need to modify any of packages installed here. An application will use a subdirectory for its data

source: https://www.howtogeek.com/117435/htg-explains-the-linux-directory-structure-explained/
https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch05s08.html

## tar

the tar command in Linux compresses a group of files into an archive. The tar archive combines multiple files and/or directories together into a single file. They don't have to be, but they can be compressed.
source: https://www.freecodecamp.org/news/tar-in-linux-example-tar-gz-tar-file-and-tar-directory-and-tar-compress-commands/

## bridge networking

Bridging is important for networking between container.
A bridge allows containers to communicate with the external network or other containers when connected with the same bridge.
How do namespace, virtual ethernet netwokring, NAT, and IPtables effect bridge networking?
source:
https://static.sched.com/hosted_files/kccna18/c1/slides.pdf

## ipam

IP address management - an aspect of container networking. Simple approaches out of the box for k8s assume static allocation of a fixed set of addresses to each node. A software package like Calico provides users more control of dynamic IPAM.

## gateway

an entrypoint for client requests. requests are proxied/routed to
appropriate services

## ipmasq

https://kubernetes.io/docs/tasks/administer-cluster/ip-masq-agent/

## Open Container Initiative - OCI

CNI, the container network interface, consists of libraries of plugins to configure network interfaces in Linux containers. It is about making the networking layer modular and pluggable. CNI is about network connectivity and removing resources when the container is deleted.

## K8s proxies

- kubectl proxy
  - runs in a pod
  - locates api server
  - adds authentication headers
  - client to proxy uses HTTP
  - proxy to apiserver uses HTTPS
- apiserver proxy
  - a bastion (special-use computer, typically only hosts a proxy server to limit vector of attack) built into apiserver
  - connects user outside of cluster to cluster IPs
  - can be used to reach a node, pod, or Service
  - does lb when used to reac a Service
  - client to proxy uses HTTPS
  - proxy to target may use either HTTP/HTTPS as chosen by proxy using availabile info
- kube proxy
  - runs on each node
  - only used to reach services
  - proxies UDP, TCP, SCTP
  - does not understand HTTP
- proxy/lb in front of apiserver
  - sits between clients and one or more apiservers
  - see source for more info
- cloud lbs on external servers
  - see source for more info
    source: https://kubernetes.io/docs/concepts/cluster-administration/proxies/

to visit:

- bridges
  - https://medium.com/@tao_66792/how-does-the-kubernetes-networking-work-part-1-5e2da2696701
  - https://kubernetes.io/docs/concepts/cluster-administration/networking/
- ipmasq
  - https://kubernetes.io/docs/tasks/administer-cluster/ip-masq-agent/
- ipam and calico
  - https://www.projectcalico.org/calico-ipam-explained-and-enhanced/
- VPC networking 101
  - https://www.youtube.com/watch?v=bGDMeD6kOz0&list=PLHXypkA_QmQJELQLjQmanpFagpf8yZGZp&index=7&t=0s

## pods

pods encapsulate the container runtime
pods networking model solves issues of port allocation, naming, service discovery, load balancing, and app config. Pods on a node can communicate wiht all pods on all nodes without network address tranlsation.
Agents on a node (daemons, kubelets) can communicate with all pods on that node.
In order to setup pod network routes (pods being able to communicate with other pods), we need to setup network routes. Pods will receive an IP address from the node's Pod CIDR range via a routing table

Clusters can be distinguished according to the way they route traffic from one Pod to another Pod. A cluster that uses google cloud routes is called a routes-based cluster.
A routes-based cluster has a range of IP addresses used for Pods and Services. The Pod address range is called `clusterIpv4Cidr`, and the range of services is called `servicesIpv4Cidr`.
A route provides a next hop for any packet that is destined for a particular Pod address range. Routes match packets by destination IP address.

source:

- https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this
- https://cloud.google.com/kubernetes-engine/docs/how-to/routes-based-cluster
- https://www.google.com/search?q=kubernetes+architecture&source=lnms&tbm=isch&sa=X&ved=2ahUKEwjx3Zry45DnAhUFWq0KHRhhCj4Q_AUoAXoECBAQAw&biw=1118&bih=645#imgrc=QUejyBfRN5jDxM:

## scheduler

In Kubernetes, scheduling refers to making sure that Pods are matched to Nodes so that Kubelet can run them.
A scheduler watches for newly created Pods that have no Node assigned. For every Pod that the scheduler discovers, the scheduler becomes responsible for finding the best Node for that Pod to run on.
