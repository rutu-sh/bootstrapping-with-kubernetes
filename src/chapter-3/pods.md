# Pods

[Pods](../chapter-1/pods.md) are the smallest deployable units you'll create in Kubernetes. We already know what a pod is, so let's go ahead and create a pod. 

Create a file named `pod.yaml` and add the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

To create the pod, run:

```bash
kubectl apply -f pod.yaml
```

To check if the pod is running, run:

```bash
kubectl get pods
```

Now let's understand the specifications in the `pod.yaml` file:

- `apiVersion`: This field specifies where the object is defined. In this case, it's defined in the `v1` version of the Kubernetes API. This field is mandatory for all Kubernetes objects as it helps the API server to locate the object definition.
- `kind`: This field specifies the type of object you're creating. In this case, it's a `Pod`.
- `metadata`: This field specifies the additional metadata that should be associated with the pod. 
    - `name`: This field specifies the name of the pod. In this case, it's `nginx-pod`.
    - `labels`: This field specifies the labels attached to the pod. Labels are key-value pairs that can be used to filter and select resources. In this case, the label `app: nginx` is attached to the pod. You can attach multiple labels to a pod like `app: nginx` and `env: dev`.
- `spec`: This field specifies the desired configuration of the pod. 
    - `containers`: This is a list of containers that should be run in the pod. In this case, there's only one container named `nginx`. For every container you specify in the pod, the following fields are mandatory:
        - `name`: This field specifies the name of the container. In this case, it's `nginx`.
        - `image`: This field specifies the image that should be used to create the container. In this case, it's `nginx:1.14.2`.
        - `ports`: This field specifies the ports that should be exposed by the container. In this case, the container is exposing port `80`.

To simply say, we are telling Kubernetes to create a pod named `nginx-pod` with a single container named `nginx` that runs the `nginx:1.14.2` image and exposes port `80`. The pod is also labeled with `app: nginx`. 

![](./assets/pod.drawio.svg)

You can read more about the pod spec [here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#pod-v1-core). 

To delete the pod, run:

```bash
kubectl delete -f pod.yaml
```