# Namespaces

In Kubernetes, a [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) is a way to partition the resources in a cluster. They are intended for use in environments with multiple users, projects, or teams. This prevents the resources from interfering with each other. 

When we installed Kubernetes, we created a few namespaces. Let's list them with the following command: 

```shell
kubectl get ns
```

You should see the following output:

```shell
$ kubectl get ns
NAME               STATUS   AGE
calico-apiserver   Active   10m
calico-system      Active   11m
default            Active   11m
kube-node-lease    Active   11m
kube-public        Active   11m
kube-system        Active   11m
tigera-operator    Active   11m
```

Let's go through the namespaces we see here: 

**kube-system**: This namespace contains the core Kubernetes resource which form the control plane. Let's list the resources in this namespace: 

```shell
kubectl get all -n kube-system
```

```shell
$ kubectl get all -n kube-system
NAME                                 READY   STATUS    RESTARTS   AGE
pod/coredns-55cb58b774-f7bds         1/1     Running   0          9m20s
pod/coredns-55cb58b774-fmv59         1/1     Running   0          9m20s
pod/etcd-master                      1/1     Running   0          9m35s
pod/kube-apiserver-master            1/1     Running   0          9m34s
pod/kube-controller-manager-master   1/1     Running   0          9m34s
pod/kube-proxy-qm8lq                 1/1     Running   0          7m28s
pod/kube-proxy-sjdsf                 1/1     Running   0          9m20s
pod/kube-proxy-xjcf7                 1/1     Running   0          6m2s
pod/kube-scheduler-master            1/1     Running   0          9m36s

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   9m34s

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   3         3         3       3            3           kubernetes.io/os=linux   9m34s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   2/2     2            2           9m34s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-55cb58b774   2         2         2       9m20s
```

As you can see from the output above, the `kube-system` namespace important Kubernetes resources like the DNS server, etcd, apiserver, controller manager, proxy, and scheduler. This namespace is managed by Kubernetes and should not be modified by users. 

**kube-public**: This namespaces is readable by everyone, even the non-authenticated users. By default, there are no resources created in this namespace. 

```shell
kubectl get all -n kube-public
```

```shell
$ kubectl get all -n kube-public
No resources found in kube-public namespace.
```

**kube-node-lease**: This namespace contains the lease objects associated with each node. A lease is how a node tells the control plane that it is alive. The node sends a heartbeat to the control plane to extend its lease. If the control plane does not receive a heartbeat from the node, it assumes that the node is dead and reschedules the pods running on that node. 

To list the leases in the `kube-node-lease` namespace, run the following command: 

```shell
kubectl get leases -n kube-node-lease
```

```shell
$ kubectl get leases -n kube-node-lease
NAME      HOLDER    AGE
master    master    28m
worker1   worker1   26m
worker2   worker2   25m
```

**default**: This is the default namespace for objects created without any namespace specified. Ex: if you create a pod without specifying a namespace, it will be created in the `default` namespace. 

The namespaces `calico-apiserver`, `calico-system`, and `tigera-operator` are created by the Calico CNI plugin.


## Creating a Namespace

Navigate to the `bootstrapping-with-kubernetes-examples/deploy/simple-namespace` directory, observe the `namespace.yaml` file: 

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: simple-namespace
  labels:
    name: simple-namespace
```

> **Note**: The manifests are available [here](https://github.com/rutu-sh/bootstrapping-with-kubernetes-examples/tree/main/deploy/simple-namespace)

Create the namespace by running the following command: 

```shell
kubectl apply -f namespace.yaml
```

```shell
$ kubectl apply -f namespace.yaml 
namespace/simple-namespace created
```

To list the namespaces, run the following command: 

```shell
kubectl get ns
```

```shell
$ kubectl get ns

NAME               STATUS   AGE
calico-apiserver   Active   93m
calico-system      Active   94m
default            Active   95m
kube-node-lease    Active   95m
kube-public        Active   95m
kube-system        Active   95m
simple-namespace   Active   114s
tigera-operator    Active   95m
```

## Understanding the Namespace Manifest

Now let's understand the specifications in the `namespace.yaml` file:

- `apiVersion`: This field specifies where the object is defined. In this case, it's defined in the `v1` version of the Kubernetes API. This field is mandatory for all Kubernetes objects as it helps the API server to locate the object definition.
- `kind`: This field specifies the type of object you're creating. In this case, it's a `Namespace`.
- `metadata`: This field specifies the details about the Namespace.
    - `name`: This is the name assigned to the namespace
    - `labels`: This is a map of key-value pairs that will be associated with the namespace. 


## Cleaning up

To delete the namespace, run the following command: 

```shell
kubectl delete -f namespace.yaml
```

## Summary 

In this chapter you learned about namespaces in Kubernetes. You learned how to list the namespaces and create new namespaces. In the next chapter, we will see how resource quotas can be used to limit the resources consumed by a namespace. We will use namespaces throughout the book, so more examples will follow.