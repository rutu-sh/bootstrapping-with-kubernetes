# kube-proxy

The **kube-proxy** is a network proxy that runs on each node in the cluster. It is responsible for managing the network for the pods running on the node. It maintains network rules on the host and performs connection forwarding.

View the status of the **kube-proxy** component by running the following command:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-proxy --show-labels
```

Here's a sample output for a 3-node cluster: 
```shell
$ kubectl get pods -n kube-system -l k8s-app=kube-proxy --show-labels -o wide
NAME               READY   STATUS    RESTARTS   AGE     IP                NODE      NOMINATED NODE   READINESS GATES   LABELS
kube-proxy-7trr5   1/1     Running   0          3h51m   ***.***.***.***   worker1   <none>           <none>            controller-revision-hash=669fc44fbc,k8s-app=kube-proxy,pod-template-generation=1
kube-proxy-8w9sn   1/1     Running   0          3h49m   ***.***.***.***   worker2   <none>           <none>            controller-revision-hash=669fc44fbc,k8s-app=kube-proxy,pod-template-generation=1
kube-proxy-zp79v   1/1     Running   0          3h52m   192.168.1.1       master    <none>           <none>            controller-revision-hash=669fc44fbc,k8s-app=kube-proxy,pod-template-generation=1
```

In this case, we have 3 nodes - `master`, `worker1`, and `worker2`. The **kube-proxy** is running as a pod on each of these nodes. The state of the kube-proxy is managed by the kubelet on the node.

The kube-proxy can work in three modes:

1. **iptables**: This is the default mode. It uses the `iptables` rules to manage the network. Uses the round robin algorithm for load balancing. We'll use this mode in our cluster.
2. **ipvs**: It uses the `ipvs` kernel module to manage the network, allows for more efficient load balancing.
3. **userspace**: In this mode the routing takes place in the userspace. Not commonly used. 