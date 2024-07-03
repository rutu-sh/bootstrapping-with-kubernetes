# CloudLab - For Researchers and Educators

If you have access to [CloudLab](https://cloudlab.us/) or a similar infrastructure where you can create virtual machines, you can use it to set up a Kubernetes cluster.

## Prerequisites

1. Access to CloudLab or a similar infrastructure where you can create virtual machines.
2. `kubectl` installed on your local machine. Refer the docs [here](https://kubernetes.io/docs/tasks/tools/#kubectl). 

## Creating CloudLab Experiment

1. Head on [here](https://www.cloudlab.us/p/GWCloudLab/rutu-k8s-3n) and instantiate this profile on CloudLab. This profile will create three nodes for you: one master node and two worker nodes.

2. Clone the [cloudlab-kubernetes](https://github.com/rutu-sh/cloudlab-kubernetes) repository locally. Follow the instructions in the README to set up the Kubernetes cluster and configure the nodes. This step will also configure the `kubectl` on your local machine to connect to the Kubernetes cluster.

3. Verify the installation by running 

```bash
kubectl get nodes
```

If everything is set up correctly, you should see the master and worker nodes listed as shown below: 

```shell
$ kubectl get nodes
NAME      STATUS   ROLES           AGE     VERSION
master    Ready    control-plane   4h3m    v1.30.2
worker1   Ready    <none>          3h53m   v1.30.2
worker2   Ready    <none>          3h52m   v1.30.2
```
