# Architecture


## Control Plane - Data Plane Design Pattern

On a very high level, Kubernetes architecture can be divided into two planes: the **control plane** and the **data plane**. This design pattern used in distributed systems as a way for *separation of concerns*. To put it simply, 

> The **control plane** makes decisions and the **data plane** carries out those decisions.

![](./assets/controlplane_dataplane.drawio.svg)

Keeping this in mind, let's dive into the architecture of Kubernetes.

## The Kubernetes Control Plane

The [Kubernetes control plane](https://kubernetes.io/docs/concepts/overview/components/) has the following components:

1. API Server
2. etcd
3. Scheduler
4. Controller Manager
5. Cloud Controller Manager

### 1. API Server

The API Server is how you interact with Kubernetes. In the following chapters, you will use a command-line tool called `kubectl`. This tool communicates with the API Server and directs the cluster to do what you want.

### 2. etcd

[etcd](https://etcd.io/) is a key-value store. The `d` in `etcd` stands for distributed. 

Kubernetes uses `etcd` to store data for all of it's resources, such as pods, services, deployments, and more. Kubernetes uses `etcd` because it's a reliable way to store data for distributed systems, as you can have multiple instances of `etcd` running at the same time, synchronizing data between them. 

*Remind me to use `etcd` as an example for distributed systems in the future.*

### 3. Scheduler

It's job is straightforward: it schedules pods to run on nodes. 

When you create a pod, you don't have to worry about where it runs. The Scheduler monitors the cluster and assigns pods to nodes. The scheduler interacts with the `etcd` through the API Server to get information about the cluster and make decisions about where to run pods.

### 4. Controller Manager

It is a collection of controllers. Each controller is responsible for managing a specific resource in the cluster. 

For example, the `ReplicaSet` controller is responsible for managing `ReplicaSets`. The `Deployment` controller is responsible for managing `Deployments`. 

### 5. Cloud Controller Manager

This component is helps you run your Kubernetes cluster on a cloud provider. It's a way to abstract the cloud provider's APIs from the core Kubernetes code.

For example, if you're running Kubernetes on AWS, the Cloud Controller Manager will help you interact with AWS APIs. If you're running Kubernetes on GCP, the Cloud Controller Manager will help you interact with GCP APIs.

<br>

Now that we have seen the control plane, let's move on to the data plane.

<br>

## The Kubernetes Data Plane

The data plane is where the actual work happens. It mainly consists of nodes which run the following components:

1. Kubelet
2. Kube Proxy
3. Container Runtime

### 1. Kubelet

The [Kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) runs as a [service](https://lym.readthedocs.io/en/latest/services.html) on each node. This is responsible for registering the node with the API Server, and managing the containers within the node. It must be noted that the Kubelet only manages the containers that are created through Kubernetes, so any other containers on your nodes that are not managed by Kubernetes are not managed by the Kubelet.

### 2. Kube Proxy

It handles the network related operations. Remember we talked about services in the [previous chapter](./services.md)? The Kube Proxy handles just that. 

### 3. Container Runtime

It is responsible for running containers on the nodes. 

<br>

Now that we've gone through the definitions, let's see how it looks in a diagram. 

The following diagram is taken from the [official Kubernetes documentation](https://kubernetes.io/docs/concepts/overview/components/), I've added a distinction to show the control plane and data plane.

![](./assets/k8s_control_plane.drawio.svg)

<br>

To show the interaction between the components, we'll go through behind the scene working of a Kubernetes deployment in the next chapter. This will help you truly understand and appreciate how the control plane and data plane work together.