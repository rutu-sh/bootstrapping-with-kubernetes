# Installation and Setup

In this chapter, we'll discuss how to get Kubernetes up and running on your local machine. I will cover two methods to set up Kubernetes on your local machine:

1. **Minikube**: For those who want to get started with Kubernetes quickly and easily. This method is recommended for beginners. You won't have to worry about setting up a cluster from scratch.

2. **A local Kubernetes setup from scratch**: We'll use three laptops and create a Kubernetes cluster from scratch. This is a more advanced method and it'll help you understand the internals of Kubernetes better.

3. **CloudLab**: If you have access to CloudLab, you can use this guide to set up a 3-node Kubernetes cluster on CloudLab.


## Kubernetes Components 

If you're going with second and third methods, you should know about the common components being created with the installation. These will help you understand the working of Kubernetes better and come in handy when troubleshooting.

1. **container runtime**: Essentially, running kubernetes comes down to running containers on your machines. The container runtime is responsible for running the containers. In our case, we have Docker installed. Though Docker has it's own runtime, called `containerd`, Kubernetes requires a runtime that implements the Container Runtime Interface (CRI). So we'll be installing `cri-dockerd` which is a CRI implementation for Docker. The installation steps specify a flag `--cri-socket=unix:///var/run/cri-dockerd.sock`. This flag tells Kubernetes to use `cri-dockerd` as the container runtime. 

2. **Pod Network CIDR**: Every pod in the cluster gets an IP Address. The `Pod Network CIDR` specifies the range of IP addresses that can be assigned to pods. We use `192.168.0.0/16` which is a pool of `65,536` IP addresses. You must be careful while choosing this range as it should not overlap with your local network. For most cases, this range should work fine. The `--pod-network-cidr` flag is used to specify this range.

3. **kubeadm**: This is a tool used to bootstrap the Kubernetes cluster. It's used to set up the control plane nodes and the worker nodes. A `kubeadm init` run on a node will set up a control plane on that node, i.e. make it a master node. A `kubeadm join` run on a node will join that node to the master node, i.e. make it a worker node. The job of `kubeadm` ends once the cluster is set up.

4. **kubectl**: This is a tool used to manage the resources in the Kubernetes cluster. It's a command-line tool that communicates with the Kubernetes API server to manage the resources. 

5. **kubelet**: This is responsible for managing the containers created by Kubernetes on the node. It runs as a service on the node and communicates with the master node to get the work assigned to it. This component runs in the background on all nodes and communicates with the master node to get the work assigned to it. For a kubelet to start, the `kubeadm init` or `kubeadm join` command must have been run on the node.

6. **kube-proxy**: While kubelet manages the containers, kube-proxy manages the networking. It's responsible for routing the traffic to the correct container. It manages the `iptables` rules on the node to route the traffic.

7. **Container Network Interface (CNI)**: This is a plugin that provides networking capabilities to the pods. It's responsible for assigning IP addresses to the pods and providing network policies. We'll be using Calico as the CNI plugin in this guide. 