# Replica Sets

Replica Sets are the resources that help you make sure that a specific number of pods are running at any given time. If a pod fails, the Replica Set will create a new one to replace it. 

## How does a Replica Set know which pods to manage?

Replica Sets use selectors, like labels, to identify the pods they should manage. When you create a Replica Set, you specify a selector. This selector tells the Replica Set which pods to manage. 

Let's create a Replica Set that manages pods with the label `app=nginx`. 

Create a YAML file named `replicaset.yaml` and add the following content:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  labels:
    app: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Let's break down the above manifest:

- `apiVersion: apps/v1`: This tells Kubernetes to use the `apps/v1` API group.
- `kind: ReplicaSet`: This tells Kubernetes that we are creating a Replica Set.
- `metadata`: This is the metadata for the Replica Set.
  - `name: nginx-replicaset`: The name of the Replica Set.
  - `labels`: The labels for the Replica Set. Here, we have a label `app: nginx-rs`.
- `spec`: This is the specification for the Replica Set.
    - `replicas: 3`: This tells the Replica Set that we want 3 replicas of the pod.
    - `selector`: This is the selector for the Replica Set.
      - `matchLabels`: This tells the Replica Set to manage pods with the label `app: nginx`.
    - `template`: This is the template for the pods which will be managed by the Replica Set. 
        - `metadata`: This is the metadata for the pod.
          - `labels`: The labels for the pod. Here, we have a label `app: nginx`. Make sure that the labels in the pod template match the labels in the selector.
        - `spec`: This is the specification for the pod. Similar to the one we defined in the [previous section](./pods.md). 
            - `containers`: This is the list of containers in the pod.
              - `name: nginx`: The name of the container.
              - `image: nginx:1.14.2`: The image for the container.
              - `ports`: The ports for the container. Here, we are exposing port 80.

Here's the visual representation of the state of the system. The pod `nginx-pod-4` is not managed by the Replica Set, as it doesn't have the label `app: nginx`.  

![](./assets/replicaset.svg)

To create the Replica Set, run the following command:

```bash
kubectl apply -f replicaset.yaml
```

You can check the status of the Replica Set using the following command:

```bash
kubectl get replicaset
```

