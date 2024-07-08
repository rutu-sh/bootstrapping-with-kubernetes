# kube-dns

Kubernetes has a built-in DNS service that helps in resolving the DNS names. This service is called `kube-dns`. 

There is a DNS record for each service and pod created in the cluster. The DNS server is responsible for resolving the DNS names to the IP addresses. 

Internally, Kubernetes uses [CoreDNS](https://coredns.io/) as the DNS server. Let's see the components that make up the `kube-dns` service:

**CoreDNS Deployment**: This is the deployment for the CoreDNS server. It manages the CoreDNS replica sets. 

```shell
$ kubectl get deployments -n kube-system -l k8s-app=kube-dns -o wide
NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                    SELECTOR
coredns   2/2     2            2           21m   coredns      registry.k8s.io/coredns/coredns:v1.11.1   k8s-app=kube-dns
``` 

**CoreDNS Replica Set**: This is the replica set for the CoreDNS server. It manages the CoreDNS pods. 

```shell
kubectl get replicaset -n kube-system -l k8s-app=kube-dns -o wide
NAME                 DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                    SELECTOR
coredns-7db6d8ff4d   2         2         2       21m   coredns      registry.k8s.io/coredns/coredns:v1.11.1   k8s-app=kube-dns,pod-template-hash=7db6d8ff4d
```

**CoreDNS Pods**: These are the pods that run the CoreDNS server. There are usually two pods running in the cluster. These pods are labeled with `k8s-app=kube-dns`.

```shell
$ kubectl get pods -n kube-system -l k8s-app=kube-dns -o wide
NAME                       READY   STATUS    RESTARTS   AGE    IP               NODE     NOMINATED NODE   READINESS GATES   LABELS
coredns-7db6d8ff4d-792wd   1/1     Running   0          157m   192.168.219.65   master   <none>           <none>            k8s-app=kube-dns,pod-template-hash=7db6d8ff4d
coredns-7db6d8ff4d-nvxsf   1/1     Running   0          157m   192.168.219.68   master   <none>           <none>            k8s-app=kube-dns,pod-template-hash=7db6d8ff4d
```

The **Service** `kube-dns` is a ClusterIP service that exposes the CoreDNS server to the cluster. A ClusterIP Service is only accessible within the cluster. 

```shell
$ kubectl get svc -n kube-system -l k8s-app=kube-dns -o wide
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   22m   k8s-app=kube-dns
```

As shown above, this service uses the `kube-dns` label selector, which is the same label used by the CoreDNS pods. 

## DNS Resolution in Kubernetes

The DNS Server, i.e. the codedns pods, always run in the `kube-system` namespace in the master node. The DNS server is responsible for resolving the DNS names to the IP addresses. Whenever `kubelet` creates a pod, it injects a DNS configuration file, `/etc/resolv.conf`, into the pod. This file contains the IP address of the DNS server and the search domains. 

Run the following command to create a simple pod and `ssh` into it: 

```bash
kubectl run -i --tty alpine --image=alpine --restart=Never -- sh
```

Once you're inside the pod, view the `/etc/resolv.conf` file:

```bash
cat /etc/resolv.conf
```

You'll see the IP address of the DNS server and the search domains.

Let's test the DNS resolution. Run the following command to resolve the IP address of the `kube-dns` service:

```bash
nslookup kube-dns.kube-system.svc.cluster.local
```

Here's the sample output: 

```shell
$ kubectl run -i --tty alpine --image=alpine --restart=Never -- sh
If you don't see a command prompt, try pressing enter.
/ # 
/ # 
/ # cat /etc/resolv.conf 
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
/ # 
/ # 
/ # nslookup kube-dns.kube-system.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53


Name:   kube-dns.kube-system.svc.cluster.local
Address: 10.96.0.10
```

From the `/etc/resolv.conf` file, you can see that the DNS Server is located at `10.96.0.10` on port `53`, which is the `kube-dns` service. When you run the `nslookup` command, the request is sent to the DNS server. The `search` field in the `/etc/resolv.conf` file specifies the search domains to append to the DNS query. 

## Putting it all together


Kubernetes has an internal DNS service, called `kube-dns`. This service is responsible for resolving the DNS names to the IP addresses. 

Kubernetes creates DNS records for: 
1. Services
2. Pods

Kubernetes uses **CoreDNS** as the DNS server. The CoreDNS server is run as a deployment and exposed via a ClusterIP service called `kube-dns`. The DNS server pods are scheduled on the Master node. 

Kubelet is responsible for injecting the DNS configuration file into the pod. The `/etc/resolve.conf` file contains the IP address of the Master node having the DNS server and the search domains. Whenever a request is made, the pods append the search domains to the DNS query till the DNS server resolves the IP address. 

Whenever a pod has to make a request, it looks up the `/etc/resolve.conf` file to find the IP of the DNS server. It then sends the **DNS query** to the DNS server. The DNS server resolves the IP address and sends it back to the pod. The pod then makes the request to the resolved IP address. 
