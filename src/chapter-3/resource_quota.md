# Resource Quotas

In an environment where multiple users or teams are using the same Kubernetes cluster, it is important to ensure that the resources are fairly distributed. This is where the resource quotas help. In the previous chapter, we discussed how namespaces are a way to logically partition the cluster resources. Resource quotas are a way to parameterize the amount of resources that can be consumed within a namespace. 

> **Note**: The manifests used in this repository are available [here](https://github.com/rutu-sh/bootstrapping-with-kubernetes-examples/blob/main/deploy/simple-resource-quota). 

Resource quotas can be set for the following: 

1. **Compute**: This includes CPU and memory.

2. **Storage**: This includes PersistentVolumeClaims, storage classes, and persistent volumes.

3. **Object Count**: This resource quota sets restriction on the number of objects that can be created for a particular resource type. 

Resource quotas are defined using the `ResourceQuota` object. 

With resource quotas you can set hard and soft limits on the resources. A hard limit is a strict limit which cannot be exceeded by the namespace. A soft limit can be exceeded but the cluster will issue alerts when the limit is reached.

> It should be noted that the resource quotas are applied per namespace and not per pod, i.e. if you set a hard limit of 100Gi for memory and 10 CPUs for a namespace, the sum of memory and CPU consumed by all the pods in the namespace should not exceed these limits.

## Creating a Resource Quota

Let's use the application we deployed in the previous chapter to understand how resource quotas work. We will start with creating a namespace and then apply a resource quota to it. For the purpose of demonstration, we will go with a very low limit on resource constraints. Next, we will deploy an application as a replica set in the namespace and see how the resource quota is enforced. 

Navigate to the `simple-resource-quota` directory: 

```bash 
cd bootstrapping-with-kubernetes-examples/deploy/simple-resource-quota
```

The `namespace.yaml` file in this directory contains the following configuration:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: simple-namespace
  labels:
    name: simple-namespace
```

This file creates a namespace named `simple-namespace`.

Create the namespace by running the following command:

```shell
kubectl apply -f namespace.yaml
```

```shell
$ kubectl apply -f namespace.yaml 
namespace/simple-namespace created
```

Next, observe the `resource-quota.yaml` file in the same directory:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: simple-resource-quota
  namespace: simple-namespace
spec:
  hard:
    cpu: "3"
    memory: 10Gi
    pods: "3"
```

This file specifies that the namespace `simple-namespace` has a hard limit of 3 CPUs, 10Gi of memory, and 3 pods. This means that: 

- The sum of memory consumed by all the pods in the namespace should not exceed 10Gi.
- The sum of CPU consumed by all the pods in the namespace should not exceed 3.
- There cannot be more than 3 pods in the namespace.

Create the resource quota by running the following command:

```shell
kubectl apply -f resource-quota.yaml
```

```shell
$ kubectl apply -f resource-quota.yaml 
resourcequota/simple-resource-quota created
```

Now that the namespace and resource quota are created, let's deploy the application. The `replica-set.yaml` file in the same directory contains the following configuration:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-replicaset
  namespace: simple-namespace
  labels:
    env: dev
    app: simple-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: simple-replicaset
  template:
    metadata:
      labels:
        app: simple-replicaset
    spec:
      containers:
      - name: apiserver
        image: rutush10/simple-restapi-server-py:v0.0.1
        ports:
        - containerPort: 8000
        resources:
          requests:
            cpu: "1"
            memory: "2Gi"
          limits:
            cpu: "1"
            memory: "2Gi"
```

This file specifies that the application should be deployed as a replica set with 3 replicas. Each pod in the replica set is configured to consume 1 CPU and 2Gi of memory. 

The overall resource consumption will be 3 CPUs and 6Gi of memory. This should not exceed the resource quota limits set for the namespace.

Create the replica set by running the following command:

```shell
kubectl apply -f replica-set.yaml
```

```shell
$ kubectl apply -f replica-set.yaml 
replicaset.apps/simple-replicaset created
```

To check if the replica set is running, run:

```shell
kubectl get rs -n simple-namespace
```

```shell
$ kubectl get rs -n simple-namespace
NAME                DESIRED   CURRENT   READY   AGE
simple-replicaset   3         3         3       2m30s
```

This shows that the replica set is running with 3 replicas. 

Now let's check how much the resource quota is being consumed. Run the following command:

```shell
kubectl describe quota -n simple-namespace
```

```shell
$ kubectl describe quota -n simple-namespace
Name:       simple-resource-quota
Namespace:  simple-namespace
Resource    Used  Hard
--------    ----  ----
cpu         3     3
memory      6Gi   10Gi
pods        3     3
```

This shows that the resource quota is being consumed as expected. The CPU and memory limits are being enforced.

Now let's modify the replica set to consume more resources than the quota allows. Update the `replica-set.yaml` file to have 4 replicas (which will consume 4 CPUs and 8Gi of memory) and apply the changes:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-replicaset
  namespace: simple-namespace
  labels:
    env: dev
    app: simple-replicaset
spec:
  replicas: 4
  selector:
    matchLabels:
      app: simple-replicaset
  template:
    metadata:
      labels:
        app: simple-replicaset
    spec:
      containers:
      - name: apiserver
        image: rutush10/simple-restapi-server-py:v0.0.1
        ports:
        - containerPort: 8000
        resources:
          requests:
            cpu: "1"
            memory: "2Gi"
          limits:
            cpu: "1"
            memory: "2Gi"
```

Apply the changes by running:

```shell
kubectl apply -f replica-set.yaml
```

```shell
$ kubectl apply -f replica-set.yaml         
replicaset.apps/simple-replicaset configured
```

Now check the resource quota again:

```shell
kubectl describe quota -n simple-namespace
```

```shell
$ kubectl get rs -n simple-namespace
NAME                DESIRED   CURRENT   READY   AGE
simple-replicaset   4         3         3       17m
```

This shows that the resource quota is being enforced. The replica set could not scale to 4 replicas as the resource quota allows only 3 pods.

Check the resource quota again:

```shell
kubectl describe quota -n simple-namespace
```

```shell
$ kubectl describe quota -n simple-namespace
Name:       simple-resource-quota
Namespace:  simple-namespace
Resource    Used  Hard
--------    ----  ----
cpu         3     3
memory      6Gi   10Gi
pods        3     3
```

This shows that the resource quota is being enforced. The CPU and memory limits are being enforced.

## Understanding the Resource Quota Manifest

Now let's understand the specifications in the `resource-quota.yaml` file:

- `apiVersion`: This field specifies where the object is defined. In this case, it's defined in the `v1` version of the Kubernetes API. This field is mandatory for all Kubernetes objects as it helps the API server to locate the object definition.
- `kind`: This field specifies the type of object you're creating. In this case, it's a `ResourceQuota`.
- `metadata`: This field specifies the details about the ResourceQuota.
    - `name`: This is the name assigned to the ResourceQuota.
    - `namespace`: This is the namespace to which the ResourceQuota is applied.
- `spec`: This field specifies the hard limits for the resources. The hard limits are the maximum limits that can be consumed by the namespace. In this case, the hard limits are set for CPU, memory, and pods.
    - `hard`: This field specifies the hard limits for the resources.
        - `cpu`: This is the hard limit for CPU.
        - `memory`: This is the hard limit for memory.
        - `pods`: This is the hard limit for the number of pods that can be created in the namespace.


## Cleaning up

To delete the namespace and the resource quota, run the following commands:

```shell
kubectl delete -f replica-set.yaml
```

```shell
kubectl delete -f resource-quota.yaml
```

```shell
kubectl delete -f namespace.yaml
```

## Summary

In this chapter, we discussed how resource quotas can be applied to limit the resources used by the namespace. We created a namespace and applied a resource quota to it. We then deployed an application as a replica set in the namespace and saw how the resource quota was enforced. Resource quotas are an important tool to ensure fair distribution of resources in a multi-tenant environment.