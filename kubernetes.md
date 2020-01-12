The server that hosts an app has specific OS, configurations, security policies, installations of versions of software, fileystem etc.

We can port an application and its environment from one server to another using containers. Containers allow us an isolated environment; however,

- containers need to be managed
- networking to the container is hard to do dynamically
- containers must be scheduled, distributed, and load balanced
- the data that a container has must persist _somewhere_

Enter Kubernetes, what problems does it solve?
Kubernetes uses pods to partition units of work and we place containers inside of pods.
We use labels to indicate roles, attributes for pods
Pods connect to the outside network and the communicate with the Kubernetes environment
We use Replication controllers to create a number of pod copies. Replication controllers also manage pods' lifecycles, like scaling up and down, rolling out deployments, and monitoring.

Pods are made discoverable via a Service
A service tells the Kubernetes env what services our application provides. The service IP and ports remains the same, the pods can come and go

A Volume is the local storage (data store, file system) that applications can access.

Namespaces

https://www.cncf.io/the-childrens-illustrated-guide-to-kubernetes/
