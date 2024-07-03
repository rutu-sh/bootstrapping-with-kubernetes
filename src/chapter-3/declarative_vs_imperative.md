# Declarative vs. Imperative object management

Kubernetes provides [two ways](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/) to manage objects: **declarative** and **imperative**. 


## Imperative object management

In imperative object management, you tell Kubernetes exactly what to do. You provide the command and Kubernetes executes it. 

## Declarative object management

In declarative object management, you tell Kubernetes what you want to achieve. You provide a configuration file and Kubernetes makes sure that the cluster matches the desired state.

## The difference

The difference between the two is subtle. Let's understand this with examples. 

### Imperative pod creation

Let's understand imperative object management with an example.

Run the following command to create a pod named `simple-pod`:

```bash
cd bootstrapping-with-kubernetes-examples/deploy/simple-pod
```

```bash
kubectl create -f pod.yaml
```

It should give the following output:

```shell
$ kubectl create -f pod.yaml
pod/simple-pod created
```

In this command, you're telling Kubernetes to exactly perform the `create` operation on the `pod.yaml` file. This will create a pod named `simple-pod` with no labels, as shown below: 

```shell
$ kubectl get pods --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
simple-pod   1/1     Running   0          25m   <none>
```

Now, say you want to add a label `app=simple-pod` to the pod. You can do this by running the following command:

```bash
kubectl label pod simple-pod app=simple-pod
```

This will add the label `app=simple-pod` to the pod. 

```shell
$ kubectl get pods --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
simple-pod   1/1     Running   0          27m   app=simple-pod
```

In this command, you're telling Kubernetes to exactly perform the `label` operation on the `simple-pod` pod.

Another way to add label would be edit the `metadata` field in the `pod.yaml` file and add the label there, as shown below:

```yaml
metadata:
  name: simple-pod
  labels:
    app: simple-pod
```

Now, if you run `create` on the updated configuration again, kubectl will give the following output:

```shell
$ kubectl create -f pod.yaml
Error from server (AlreadyExists): error when creating "pod.yaml": pods "simple-pod" already exists
```

This happens because the `create` operation is idempotent. It will only create the object if it doesn't exist. It won't update the object if it already exists. This creates issues when you are working with existing objects and want to update them using imperative object management. To perform such updates, the imperative way, you will need to keep using commands like `kubectl label pod simple-pod app=simple-pod`, `kubectl edit pod simple-pod`, etc.

This behavior is not ideal when you want to manage objects in a more reliable and consistent way. This is where declarative object management helps.

Before going into declarative object management, let's delete the `simple-pod` pod:

```bash
kubectl delete -f pod.yaml
```


### Declarative pod creation

Using declarative object management, you provide a configuration file that describes the desired state of the object. Kubernetes will make sure that the cluster matches the desired state.

Let's understand this with an example.

```bash
kubectl apply -f pod.yaml
```

This will again create a pod named `simple-pod` with no labels. 

```shell
$ kubectl get pods --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
simple-pod   1/1     Running   0          48s   <none>
```

Now, to add a label `app=simple-pod` to the pod, you can edit the `pod.yaml` file and add the label there, as shown below:

```yaml
metadata:
  name: simple-pod
  labels:
    app: simple-pod
```

Now, if you run `apply` on the updated configuration again, kubectl will update the pod with the new label:

```shell
$ kubectl apply -f pod.yaml
pod/simple-pod configured
```

```shell
$ kubectl get pods --show-labels
NAME         READY   STATUS    RESTARTS   AGE     LABELS
simple-pod   1/1     Running   0          2m42s   app=simple-pod
```

The benefit here is that you don't have to worry about the current state of the object. You just provide the desired state and Kubernetes will make sure that the cluster matches the desired state. This is more reliable and easier to manage.

Also, if you run `apply` command again, without making any changes, the command will still run successfully.

Delete the `simple-pod` pod:

```bash
kubectl delete -f pod.yaml
```


## Which one to use?

Declarative object management is the recommended way to manage objects in Kubernetes. This way, you don't have to consistently keep track of the current state of the object, Kubernetes will do that for you. All you need to do is to specify the desired state. 
