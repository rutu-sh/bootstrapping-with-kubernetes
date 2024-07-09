# kube-apiserver

This is the component that provides a RESTful interface on the Kubernetes control plane. It is used to manage the lifecycle of resources in the cluster. 

Run the following command to check the status of the `kube-apiserver` component:

```bash
kubectl get pods -n kube-system -l component=kube-apiserver --show-labels
```

```shell
kubectl get pods -n kube-system -l component=kube-apiserver --show-labels
NAME                    READY   STATUS    RESTARTS   AGE   LABELS
kube-apiserver-master   1/1     Running   0          14h   component=kube-apiserver,tier=control-plane
```

This pod is controlled and run by the master node. It listens on the port `6443` for incoming connections.

The **kubectl** command-line-interface communicates with the `kube-apiserver` using the `.kube/config` file. This file contains the details of the cluster, including the server address, the certificate authority, and the user credentials.

To view the `kube-apiserver` logs, run the following command:

```bash
kubectl logs kube-apiserver-master -n kube-system
```

And since this is a RESTful interface that provides a set of APIs, you can get the API documentation by running the following command:

```bash
kubectl get --raw /openapi/v2 > openapi.json
```

You can then import the `openapi.json` file into a REST client like [Postman](https://www.postman.com/) to view the API documentation.

