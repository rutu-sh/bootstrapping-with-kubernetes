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
NAME                                                  READY   STATUS      RESTARTS   AGE     IP               NODE      NOMINATED NODE   READINESS GATES
pod/coredns-7db6d8ff4d-thngk                          1/1     Running     0          10h     192.168.219.68   master    <none>           <none>
pod/coredns-7db6d8ff4d-vr428                          1/1     Running     0          10h     192.168.219.67   master    <none>           <none>
pod/etcd-master                                       1/1     Running     0          10h     192.168.1.1      master    <none>           <none>
pod/kube-apiserver-master                             1/1     Running     0          10h     192.168.1.1      master    <none>           <none>
pod/kube-controller-manager-master                    1/1     Running     0          10h     192.168.1.1      master    <none>           <none>
pod/kube-proxy-5qw9p                                  1/1     Running     0          10h     ***.***.***.**   worker2   <none>           <none>
pod/kube-proxy-fvkp2                                  1/1     Running     0          10h     ***.***.***.**   worker1   <none>           <none>
pod/kube-proxy-ghvxg                                  1/1     Running     0          10h     192.168.1.1      master    <none>           <none>
pod/kube-scheduler-master                             1/1     Running     0          10h     192.168.1.1      master    <none>           <none>
pod/node-shell-534c9e2d-6ae6-4710-8493-e3ae7f7c8cb2   0/1     Completed   0          4h57m   ***.***.***.**   worker2   <none>           <none>
pod/node-shell-7761b07c-a815-4ddf-963e-7ea65f031cef   0/1     Completed   0          5h7m    ***.***.***.**   worker1   <none>           <none>

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   10h   k8s-app=kube-dns

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE   CONTAINERS   IMAGES                               SELECTOR
daemonset.apps/kube-proxy   3         3         3       3            3           kubernetes.io/os=linux   10h   kube-proxy   registry.k8s.io/kube-proxy:v1.30.2   k8s-app=kube-proxy

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                    SELECTOR
deployment.apps/coredns   2/2     2            2           10h   coredns      registry.k8s.io/coredns/coredns:v1.11.1   k8s-app=kube-dns

NAME                                 DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                    SELECTOR
replicaset.apps/coredns-7db6d8ff4d   2         2         2       10h   coredns      registry.k8s.io/coredns/coredns:v1.11.1   k8s-app=kube-dns,pod-template-hash=7db6d8ff4d
```

I've hidden the IP addresses for security reasons.

The following chapters will cover each of these components. 
