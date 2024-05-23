# A local Kubernetes setup from scratch

This is one of the most interesting experiments I've done with Kubernetes. I had three linux machines lying around and I decided to create a Kubernetes cluster from scratch. This was a great learning experience and I highly recommend you try this out if you have the resources. I wanted everything to be interconnected over my home wifi network. This is how I did it:

## Machines used

3 laptops running Ubuntu 22.04 LTS. These laptops had Docker installed on them and were connected to the same wifi network.

## Common setup

### Install Docker on all machines

I followed the official Docker documentation [here](https://docs.docker.com/engine/install/ubuntu/). Here are some points to note to ease the process:

1. If the `apt-get update` fails with an error in reading from the `docker.list` file, make sure that the url in the file ends with `ubuntu` and not with `debian`. 
2. Follow the [post installation instructions](https://docs.docker.com/engine/install/linux-postinstall/) to run Docker as a non-root user. Restart the machine after this step, you'll be able to run Docker commands without `sudo`.

### Install CRI-Dockerd on all machines

This is a Kubernetes requirement. It's important to have the Container Runtime Interface (CRI) installed on all machines. I used the following method to install CRI-Dockerd:

1. Go to [this page](https://github.com/Mirantis/cri-dockerd/releases). 
2. Expand the Assets section and download the `cri-dockerd` deb file for your OS version. I went with `cri-dockerd_0.3.14.3-0.ubuntu-jammy_amd64.deb`. 
3. Open the Terminal and `cd` to the directory where the downloaded file is present and run 

```bash
sudo dpkg -i <deb-file-name>
```
4. After the installation is complete, run 

```bash
sudo systemctl enable cri-dockerd
sudo systemctl start cri-dockerd
```
5. Verify the installation by running 

```bash
sudo systemctl status cri-dockerd
```


<br>

Now that the container runtime is installed on all machines, we can proceed to set up the Kubernetes cluster.

### Install kubeadm, kubelet, and kubectl on all machines

First turn off swap on all machines by running 

```bash
sudo swapoff -a
```

Go to the [official installation instructions](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl) and follow the steps to install `kubeadm`, `kubelet`, and `kubectl` on all machines.


<br>

At this point we've set up the common requirements on all machines. Now we can proceed to set up the Kubernetes cluster.


## Setting up the Master Node

1. Run the following command to initialize the master node:

```bash
kubeadm init --cri-socket=unix:///var/run/cri-dockerd.sock --pod-network-cidr=192.168.0.0/16
``` 

The `--cri-socket=unix:///var/run/cri-dockerd.sock` is used to specify which CRI to use. Since we're using CRI-Dockerd, we need to specify the socket path. 

The `--pod-network-cidr=192.168.0.0/16` is used to specify the range of IP addresses that pods can use. This is a pool of `65,536` IP addresses that Kubernetes can assign to pods.

2. After the command completes, you'll see a message like this:

3. Next, we'll install [Calico](https://www.tigera.io/tigera-products/calico/) as the CNI plugin. A CNI plugin is used to provide networking and security services to pods. Calico is a popular choice for Kubernetes clusters. This plugin is used to assign IP addresses to pods and provide network policies. Head on [here](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart#install-calico) and setup Calico on the Kubernetes cluster. 

Great! Now our master node is set up. Let's move on to setting up the worker nodes. Instead of running `kubeadm init` on the worker nodes, we have to run `kubeadm join` to join the worker nodes to the master node. To get the join command, run the following command on the master node:

```bash
kubeadm token create --print-join-command
```

This will print the join command. Copy this command. 

## Setting up the Worker Nodes

1. Run the join command on the worker nodes. This will join the worker nodes to the master node.
2. Verify that the worker nodes have joined the cluster by running 

```bash
kubectl get nodes
```

This will show the master node and the worker nodes.


And that's it! We have a Kubernetes cluster up and running from scratch. You can now deploy applications to this cluster and experiment with Kubernetes.

From now on, I'll be using this setup to demonstrate various Kubernetes concepts, but you can use the three node Minikube cluster as mentioned [here](minikube.md) to simulate this setup. 