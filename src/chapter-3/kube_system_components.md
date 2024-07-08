# Kube System Components

> **Note**: This chapter is only inteded for a deeper understanding of Kubernetes, can be skipped.

There are some default resources created when you set up a Kubernetes cluster, called the Kubernetes System Components. These components are responsible for the working of the cluster. 

List the components by running the following command: 

```bash
kubectl get all -n kube-system -o wide
```

Here's a sample output: 

```shell
$ kubectl get all -n kube-system -o wide
NAME                                 READY   STATUS    RESTARTS   AGE   IP               NODE      NOMINATED NODE   READINESS GATES
pod/coredns-7db6d8ff4d-792wd         1/1     Running   0          20m   192.168.219.65   master    <none>           <none>
pod/coredns-7db6d8ff4d-nvxsf         1/1     Running   0          20m   192.168.219.68   master    <none>           <none>
pod/etcd-master                      1/1     Running   0          20m   192.168.1.1      master    <none>           <none>
pod/kube-apiserver-master            1/1     Running   0          20m   192.168.1.1      master    <none>           <none>
pod/kube-controller-manager-master   1/1     Running   0          20m   192.168.1.1      master    <none>           <none>
pod/kube-proxy-9l64r                 1/1     Running   0          20m   192.168.1.1      master    <none>           <none>
pod/kube-proxy-svnvd                 1/1     Running   0          17m   ***.***.***.**   worker2   <none>           <none>
pod/kube-proxy-zfvgt                 1/1     Running   0          19m   ***.***.***.**   worker1   <none>           <none>
pod/kube-scheduler-master            1/1     Running   0          20m   192.168.1.1      master    <none>           <none>

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   20m   k8s-app=kube-dns

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE   CONTAINERS   IMAGES                               SELECTOR
daemonset.apps/kube-proxy   3         3         3       3            3           kubernetes.io/os=linux   20m   kube-proxy   registry.k8s.io/kube-proxy:v1.30.2   k8s-app=kube-proxy

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                    SELECTOR
deployment.apps/coredns   2/2     2            2           20m   coredns      registry.k8s.io/coredns/coredns:v1.11.1   k8s-app=kube-dns

NAME                                 DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                    SELECTOR
replicaset.apps/coredns-7db6d8ff4d   2         2         2       20m   coredns      registry.k8s.io/coredns/coredns:v1.11.1   k8s-app=kube-dns,pod-template-hash=7db6d8ff4d
```

I've hidden the IP addresses for security reasons.

The following chapters will cover each of these components. 
