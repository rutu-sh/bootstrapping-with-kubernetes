# kube-scheduler

The **kube-scheduler** is responsible for scheduling pods to run on nodes, so that you don't worry about monitoring the cluster's state and assigning pods to nodes.

Run the following command to check the status of the **kube-scheduler** component:

```bash
kubectl get pods -n kube-system -l component=kube-scheduler --show-labels
```

```shell
$ kubectl get pods -n kube-system -l component=kube-scheduler --show-labels -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES   LABELS
kube-scheduler-master   1/1     Running   0          5h45m   192.168.1.1   master   <none>           <none>            component=kube-scheduler,tier=control-plane
```
