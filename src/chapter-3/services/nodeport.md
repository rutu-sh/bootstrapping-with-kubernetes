# NodePort Service

## Creating a NodePort Service

A NodePort Service exposes a specified port on all the nodes in the cluster. Any traffic that comes to the Node on that port is routed to the selected Pods. 

To Create a NodePort Service, update the `service.yaml` file to have the following configuration:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: NodePort
  selector:
    app: simple-deployment
  ports:
    - protocol: TCP
      port: 8000
      targetPort: 8000
      nodePort: 30001
```

Apply the changes to the Service:

```bash
kubectl apply -f service.yaml
```

Now, we'll send a request to the Node directly. First, get the nodes where the Pods are running: 

```shell
$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE    IP                NODE      NOMINATED NODE   READINESS GATES
simple-deployment-794f78c89-cpwdd   1/1     Running   0          7m4s   192.168.189.66    worker2   <none>           <none>
simple-deployment-794f78c89-gq7ts   1/1     Running   0          7m4s   192.168.235.130   worker1   <none>           <none>
simple-deployment-794f78c89-nfzzk   1/1     Running   0          7m4s   192.168.219.71    master    <none>           <none>
```

In this case, the Pods are running on all the nodes. So, you can send a request to any of the nodes.

Next get the IP address of the node: 

If you're using Minikube, you can use the following command: 

```bash
minikube ip
```

If you're hosting the cluster locally, you can use the following command:

```bash
kubectl get nodes -o wide
```

If you're using CloudLab, you can visit the node's page and get the IP under `Control IP`. 

Now, send a request to the Node:

```bash
curl -i http://<node-ip>:30001/rest/v1/health/
```

```shell
$ curl -i http://<node-ip>:30001/rest/v1/health/
HTTP/1.1 200 OK
date: Fri, 05 Jul 2024 18:44:34 GMT
server: uvicorn
content-length: 15
content-type: application/json

{"status":"ok"}
```

Here, we're hitting the node port `30001`. This is the port we've specified in the `service.yaml` file. The traffic is routed to the Pods based on the selector.

### Understanding the Service manifest

Let's break down the `service.yaml` file:
- `apiVersion: v1`: This tells Kubernetes to use the `v1` API version.
- `kind: Service`: This specifies the type of object we're creating, which is a Service.
- `metadata`: This field specifies the additional metadata that should be associated with the Service.
    - `name`: This field specifies the name of the Service. In this case, it's `backend-service`.
- `spec`: This field specifies the specification of the Service.
    - `type`: This field specifies the type of the Service. In this case, it's `NodePort`.
    - `selector`: This field specifies the selector that the Service uses to route the traffic to the Pods. In this case, the selector is `app: simple-deployment`.
    - `ports`: This field specifies the ports that the Service listens on.
        - `protocol`: This field specifies the protocol that the Service listens on. In this case, it's `TCP`.
        - `port`: This field specifies the port on which the Service listens. In this case, it's `8000`. This port should be used by the pods talk to the service within the cluster.
        - `targetPort`: This field specifies the port on the Pods to which the traffic is routed. In this case, it's `8000`.
        - `nodePort`: This field specifies the port on the Node to which the traffic is routed. In this case, it's `30001`.
        - `name`: [Optional] This field specifies the name of the port. In this case, it's not specified.


To access a NodePort service from within a cluster, you can use the same method as for a ClusterIP service. You can use the Service name to access the service. 

```shell
$ kubectl run -i --tty --rm debug --image=alpine -- sh

If you don't see a command prompt, try pressing enter.
/ # apk add curl
fetch https://dl-cdn.alpinelinux.org/alpine/v3.20/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.20/community/x86_64/APKINDEX.tar.gz
(1/10) Installing ca-certificates (20240226-r0)
(2/10) Installing brotli-libs (1.1.0-r2)
(3/10) Installing c-ares (1.28.1-r0)
(4/10) Installing libunistring (1.2-r0)
(5/10) Installing libidn2 (2.3.7-r0)
(6/10) Installing nghttp2-libs (1.62.1-r0)
(7/10) Installing libpsl (0.21.5-r1)
(8/10) Installing zstd-libs (1.5.6-r0)
(9/10) Installing libcurl (8.8.0-r0)
(10/10) Installing curl (8.8.0-r0)
Executing busybox-1.36.1-r29.trigger
Executing ca-certificates-20240226-r0.trigger
OK: 13 MiB in 24 packages
/ # 
/ # curl -i ^C

/ # curl -i http://backend-service:8000/rest/v1/health/
HTTP/1.1 200 OK
date: Fri, 05 Jul 2024 21:12:04 GMT
server: uvicorn
content-length: 15
content-type: application/json

{"status":"ok"}/ # 
/ # 
```

Note that here the port is `8000` and not `30001`. This value is defined by the `port` field in the `service.yaml` file.

## Cleaning up

To clean up the resources created in this section, run the following commands:

```bash
kubectl delete -f deployment.yaml
```

```bash
kubectl delete -f service.yaml
```

## Summary

In this section, we learned how to create a NodePort Service. We also learned how to access the Service from outside the cluster. We used the `nodePort` field to specify the port on the Node to which the traffic is routed. We also learned how to access the Service from within the cluster using the Service name.