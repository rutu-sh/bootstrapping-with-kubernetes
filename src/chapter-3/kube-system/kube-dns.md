# kube-dns

Kubernetes has a built-in DNS service that helps in resolving the DNS names. This service is called `kube-dns`. 

There is a DNS record for each service and pod created in the cluster. The DNS server is responsible for resolving the DNS names to the IP addresses. 

Internally, Kubernetes uses [CoreDNS](https://coredns.io/) as the DNS server. Let's see the components that make up the `kube-dns` service:

**CoreDNS Deployment**: This is the deployment for the CoreDNS server. It manages the CoreDNS replica sets. 

```shell
$ kubectl get deployments -n kube-system -l k8s-app=kube-dns -o wide
NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                    SELECTOR
coredns   2/2     2            2           10h   coredns      registry.k8s.io/coredns/coredns:v1.11.1   k8s-app=kube-dns
``` 

**CoreDNS Replica Set**: This is the replica set for the CoreDNS server. It manages the CoreDNS pods. 

```shell
kubectl get replicaset -n kube-system -l k8s-app=kube-dns -o wide
NAME                 DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                    SELECTOR
coredns-7db6d8ff4d   2         2         2       10h   coredns      registry.k8s.io/coredns/coredns:v1.11.1   k8s-app=kube-dns,pod-template-hash=7db6d8ff4d
```

**CoreDNS Pods**: These are the pods that run the CoreDNS server. There are usually two pods running in the cluster.

```shell
$ kubectl get pods -n kube-system -l k8s-app=kube-dns -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
coredns-7db6d8ff4d-thngk   1/1     Running   0          10h   192.168.219.68   master   <none>           <none>
coredns-7db6d8ff4d-vr428   1/1     Running   0          10h   192.168.219.67   master   <none>           <none>
```

The **Service** `kube-dns` is a ClusterIP service that exposes the CoreDNS server to the cluster. 

```shell
$ kubectl get svc -n kube-system -l k8s-app=kube-dns -o wide
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   10h   k8s-app=kube-dns
```

