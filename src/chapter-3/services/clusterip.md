# ClusterIP Service

## Creating a ClusterIP Service 

We'll use the deployment created in the [previous section](./deployments.md) to create applications. Then, we'll use a Service to route traffic to the Pods created by the Deployment.

Navigate to the `simple-service` directory:

```bash
cd bootstrapping-with-kubernetes-examples/deploy/simple-service
```

The `service.yaml` file in this directory contains the following configuration:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: simple-deployment
  ports:
    - protocol: TCP
      port: 8000
      targetPort: 8000
```

Start the deployment and service with the following commands:

```bash
kubectl apply -f deployment.yaml
```

```bash
kubectl apply -f service.yaml
```
Here's a sample output:

```shell
$ kubectl apply -f deployment.yaml 
deployment.apps/simple-deployment created
$ kubectl apply -f service.yaml
service/backend-service created
```

Verify if the service and deployment are created:

```shell
$ kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   3/3     3            3           10m
$ kubectl get pods --show-labels
NAME                                READY   STATUS    RESTARTS   AGE   LABELS
simple-deployment-794f78c89-9h7jh   1/1     Running   0          10m   app=simple-deployment,pod-template-hash=794f78c89
simple-deployment-794f78c89-hs2s9   1/1     Running   0          10m   app=simple-deployment,pod-template-hash=794f78c89
simple-deployment-794f78c89-tgnhl   1/1     Running   0          10m   app=simple-deployment,pod-template-hash=794f78c89
$ kubectl get services
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
backend-service   ClusterIP   10.98.60.140   <none>        8000/TCP   11m
kubernetes        ClusterIP   10.96.0.1      <none>        443/TCP    20m
```


Let's send a request to the Service. 

To send request to a ClusterIP Service, we need to be inside the Kubernetes cluster. So we'll use the `kubectl` command to run a Pod with interactive shell:

```bash
kubectl run -i --tty --rm debug --image=alpine -- sh
```

Install curl 
```bash
apk add curl
```

Send a request to the Service:

```bash
curl -i http://backend-service:8000/rest/v1/health/
```

Here's the output:

```shell
$ kubectl run -i --tty --rm debug --image=alpine -- sh

If you don't see a command prompt, try pressing enter.

/ # 
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
/ # 
/ # curl -i http://backend-service:8000/rest/v1/health/
HTTP/1.1 200 OK
date: Thu, 04 Jul 2024 04:48:21 GMT
server: uvicorn
content-length: 15
content-type: application/json

{"status":"ok"}/ # 
/ # 
```

Here's a breakdown of what happened here: 
- Kubernetes has a built-in DNS Server that creates DNS records for the Services. The resolution happens by the Service name. In this case, the Service name is `backend-service`.
- When you create a Service, you spcify a `selector` field. Based on this, the endpoint slice controller creates Endpoint objects. The Endpoint object contains the IP addresses of the Pods that match the selector. You can see the Endpoint object created for the Service using the following command:

```shell
$ kubectl get endpoints backend-service
NAME              ENDPOINTS                                                      AGE
backend-service   192.168.189.66:8000,192.168.189.67:8000,192.168.235.130:8000   38m
```

- You can see to which Pods the Service is routing the traffic. Run the following command to see the IP addresses of the Pods:

```shell
$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP                NODE      NOMINATED NODE   READINESS GATES
simple-deployment-794f78c89-9h7jh   1/1     Running   0          39m   192.168.189.66    worker2   <none>           <none>
simple-deployment-794f78c89-hs2s9   1/1     Running   0          39m   192.168.189.67    worker2   <none>           <none>
simple-deployment-794f78c89-tgnhl   1/1     Running   0          39m   192.168.235.130   worker1   <none>           <none>
```
- Along with this a DNS record for the Service is created by the DNS server.
- When you send a curl http://backend-service:8000/rest/v1/health/ request, the DNS server resolves the Service name to the IP addresses of the Pods. The request is then routed to one of the Pods.

### Understanding the Service manifest

Let's break down the `service.yaml` file:
- `apiVersion: v1`: This tells Kubernetes to use the `v1` API version.
- `kind: Service`: This specifies the type of object we're creating, which is a Service.
- `metadata`: This field specifies the additional metadata that should be associated with the Service.
    - `name`: This field specifies the name of the Service. In this case, it's `backend-service`.
- `spec`: This field specifies the specification of the Service.
    - `type`: This field specifies the type of the Service. In this case, it's `ClusterIP`.
    - `selector`: This field specifies the selector that the Service uses to route the traffic to the Pods. In this case, the selector is `app: simple-deployment`.
    - `ports`: This field specifies the ports that the Service listens on.
        - `protocol`: This field specifies the protocol that the Service listens on. In this case, it's `TCP`.
        - `port`: This field specifies the port on which the Service listens. In this case, it's `8000`.
        - `targetPort`: This field specifies the port on the Pods to which the traffic is routed. In this case, it's `8000`.
        - `name`: [Optional] This field specifies the name of the port. In this case, it's not specified.

## Cleaning up

To clean up the resources created in this section, run the following commands:

```bash
kubectl delete -f deployment.yaml
```

```bash
kubectl delete -f service.yaml
```

## Summary

In this section, we created a ClusterIP Service to route traffic to the Pods created by the Deployment. We then sent a request to the Service from inside the cluster to see how the traffic is routed to the Pods. 