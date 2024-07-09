# etcd

This component is the distributed key-value store used by Kubernetes to store all cluster data. 

Run the following command to check the status of the etcd cluster:

```bash
kubectl get pods -n kube-system -l component=etcd
```

```shell
$ kubectl get pods -n kube-system -l component=etcd -o wide --show-labels
NAME          READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES   LABELS
etcd-master   1/1     Running   0          4h24m   192.168.1.1   master   <none>           <none>            component=etcd,tier=control-plane
```

SSH into the `etcd` pod by running the following command: 

```bash
kubectl exec -it etcd-master -n kube-system -- /bin/sh
```

Set the environment variables:

```bash
export ETCDCTL_API=3
export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
```

View the Keys created for the Services in the etcd cluster, by running the following command:

```bash
etcdctl get /registry/services/specs --prefix --keys-only
```

Similarly, view the pods in the cluster by running the following command:

```bash
etcdctl get /registry/pods --prefix --keys-only
```
*To view the data for each key, remove the `--keys-only` flag.*

Here's the sample output: 

```shell
$ kubectl exec -it etcd-master -n kube-system -- /bin/sh
sh-5.2#                                                                                                                                                                            
sh-5.2# export ETCDCTL_API=3
sh-5.2# export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
sh-5.2# export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
sh-5.2# export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
sh-5.2# 
sh-5.2# etcdctl get /registry/services/specs --prefix --keys-only
/registry/services/specs/calico-apiserver/calico-api

/registry/services/specs/calico-system/calico-kube-controllers-metrics

/registry/services/specs/calico-system/calico-typha

/registry/services/specs/default/kubernetes

/registry/services/specs/kube-system/kube-dns
sh-5.2# 
sh-5.2# etcdctl get /registry/pods/ --prefix --keys-only    
/registry/pods/calico-apiserver/calico-apiserver-7cb798b74b-dmrx6

/registry/pods/calico-apiserver/calico-apiserver-7cb798b74b-s59cs

/registry/pods/calico-system/calico-kube-controllers-798d6c8f99-6mq5x

/registry/pods/calico-system/calico-node-gbxp7

/registry/pods/calico-system/calico-node-gpbrj

/registry/pods/calico-system/calico-node-wkjxt

/registry/pods/calico-system/calico-typha-65f8578fc5-8vshb

/registry/pods/calico-system/calico-typha-65f8578fc5-z5srw

/registry/pods/calico-system/csi-node-driver-6s4zr

/registry/pods/calico-system/csi-node-driver-7lqxb

/registry/pods/calico-system/csi-node-driver-kxp7r

/registry/pods/kube-system/coredns-7db6d8ff4d-792wd

/registry/pods/kube-system/coredns-7db6d8ff4d-nvxsf

/registry/pods/kube-system/etcd-master

/registry/pods/kube-system/kube-apiserver-master

/registry/pods/kube-system/kube-controller-manager-master

/registry/pods/kube-system/kube-proxy-9l64r

/registry/pods/kube-system/kube-proxy-svnvd

/registry/pods/kube-system/kube-proxy-zfvgt

/registry/pods/kube-system/kube-scheduler-master

/registry/pods/tigera-operator/tigera-operator-76ff79f7fd-2qddl

sh-5.2# 
```

