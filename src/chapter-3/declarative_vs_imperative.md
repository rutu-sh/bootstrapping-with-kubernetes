# Declarative vs. Imperative object management

Kubernetes provides [two ways](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/) to manage objects: **declarative** and **imperative**. 


## Imperative object management

In imperative object management, you tell Kubernetes exactly what to do. You provide the command and Kubernetes executes it. 

## Declarative object management

In declarative object management, you tell Kubernetes what you want to achieve. You provide a configuration file and Kubernetes makes sure that the cluster matches the desired state.

## The difference

The difference between the two is subtle. Let's understand this with examples. 

### Imperative pod creation

```bash
kubectl create pod nginx --image=nginx --labels=app=nginx
```

In this command, you're telling Kubernetes to create a pod named `nginx` with the `nginx` image with the label `app=nginx`.

This will create a pod in the cluster with the specified configuration.

Now, if you run this command again, Kubernetes will try to create another pod with the same configuration. This can lead to conflicts and issues in the cluster.



### Declarative pod creation

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
    containers:
    - name: nginx
        image: nginx
```

In this configuration file, you're telling Kubernetes that you want a pod named `nginx` with the `nginx` image and the label `app=nginx`. Now if you apply this configuration file, Kubernetes will make sure that the cluster matches the desired state. 

If you apply this configuration file again, Kubernetes will check if the cluster matches the desired state. If it does, it won't do anything. If it doesn't, it will make the necessary changes to match the desired state.

## Which one to use?

Declarative object management is the recommended way to manage objects in Kubernetes. It's more reliable and easier to manage. Plus, it's easier to track changes and manage configurations.