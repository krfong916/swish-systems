k8s walkthrough

## Compute Resources

- We create a custom VPC network
  - `gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom`
- According to our IP address range, we create a subnet in the custom VPC network we just created

```gcloud compute networks subnets create kubernetes
  --network kubernetes-the-hard-way
  --range 10.240.0.0/24
```

- The 10.240.0.0/24 IP address range can host up to 254 compute instances.

* We create a firewall rule that allows internal communication across all network protocols: we allow tcp, udp, and icmp
  ```gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal
  --allow tcp,udp,icmp
  --network kubernetes-the-hard-way
  --source-ranges 10.240.0.0/24,10.200.0.0/16
  ```
* We also create a firewall rule that allows external SSH, HTTPS, and ICMP
  ```gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
    --allow tcp:22,tcp:6443,icmp \
    --network kubernetes-the-hard-way \
    --source-ranges 0.0.0.0/0
  ```
* By default, external load balancer exposes the Kubernetes API to remote clients

  ```
  NAME
  kubernetes-the-hard-way-allow-external
  NETWORK
  kubernetes-the-hard-way
  DIRECTION
  INGRESS
  PRIORITY
  1000
  ALLOW
  tcp:22,tcp:6443,icmp

  NAME
  kubernetes-the-hard-way-allow-internal
  NETWORK
  kubernetes-the-hard-way
  DIRECTION
  INGRESS
  PRIORITY
  1000
  ALLOW
  tcp,udp,icmp
  ```

* We need to attach a static IP to the external load balancer fronting the K8s API servers. A static IP address is created in our default compue region
  - `gcloud compute addresses create kubernetes-the-hard-way --region $(gcloud config get-value compute/region)`
* We create compute instances provisioned using an ubuntu server. The ubuntu server has good support for containerd - the container runtime. Each compute instance is provided a fixed private IP address (we do this to simplify the Kubernetes bootstrapping process)
  - Kubernetes Controllers - we create three compute instances that will host the Kubernetes control plane
    ```
    for i in 0 1 2; do
      gcloud compute instances create controller-${i} \
        --async \
        --boot-disk-size 200GB \
        --can-ip-forward \
        --image-family ubuntu-1804-lts \
        --image-project ubuntu-os-cloud \
        --machine-type n1-standard-1 \
        --private-network-ip 10.240.0.1${i} \
        --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
        --subnet kubernetes \
        --tags kubernetes-the-hard-way,controller
    done
    ```
  - We create Kubernetes worker instances. Each worker is allocated a pod subnets from the Kubernetes cluster CIDR range. The pdo subnet will be later configured for networking.
  - The pod-cidr instance contains metadata. The instance's metadata will be used to expose the pod subnet allocations to compute instances at runtime.
  - We create three compute instances which will host the Kubernetes worker nodes
  ```
  for i in 0 1 2; do
    gcloud compute instances create worker-${i} \
      --async \
      --boot-disk-size 200GB \
      --can-ip-forward \
      --image-family ubuntu-1804-lts \
      --image-project ubuntu-os-cloud \
      --machine-type n1-standard-1 \
      --metadata pod-cidr=10.200.${i}.0/24 \
      --private-network-ip 10.240.0.2${i} \
      --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
      --subnet kubernetes \
      --tags kubernetes-the-hard-way,worker
  done
  ```
* Lastly, we need to configure SSH access to the controller and worker node instances
  - we simply ssh access to one of the controllers and enter a pub/priv keygen pair. This will log us into the ubuntu instance

## Certificate Authority

Generate CA and TLS certificates for the following components:

- admin client
- worker nodes (kubelet)
- controller nodes (controller manager)
- api server (kubernetes api server)
- proxy (kube proxy client)
- scheduler (kube scheduler)
- service account key pair

and distribute the client and server certificates

## Generating configuration files

We can use config files to organize cluster access, like authenticating various entities according to rules (contexts) specified in a config file, using the CLI tool, kubectl, and directing the entity to the right cluster.

When we have several clusters, they can be accessed by a variety of entities:

- A kubelet (worker node) may need to authenticate using certs
- An administrator may have sets of certs they provide to individual users
- A user may need to authenticate using a token

Kubeconfig files help us organize clusters, users, and namespaces.
A context element in a kubeconfig file groups access parameters together under a unique name. The `kubectl` uses parameters from the current context to communicate with the cluster.

Context are defined within a config file. They allow us to choose the correct cluster to handle a given request and communicate with the API server of the chosen cluster.

Depending on the method of access and the env variables of the current context will determine which cluster and api server handles the auth.
For example, the `dev-frontend` context may specify, use the credentials of the `developer` user to access the frontend namespace of the `development` cluster.

A kubeconfig uses an API server to connect to. We use the static IP of the load balancers fronting the API servers in order to achieve HA

When generating a kubeconfig file for a kubelet (worker node), we must use the cert of the kubelet in order to comply with the node authorizer.

We have to generate configs for the kubelets, controllers, proxy, admin, and scheduler

https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
