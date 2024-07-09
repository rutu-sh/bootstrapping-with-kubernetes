# kube-controller-manager

The **kube-controller-manager** is responsible for running the controllers that regulate the state of the cluster. It polls the **kube-apiserver** for changes to the cluster state and makes changes to the cluster to match the desired state.

View the status of the **kube-controller-manager** component by running the following command:

```bash
kubectl get pods -n kube-system -l component=kube-controller-manager --show-labels
```

Here's a sample output:

```shell
$ kubectl get pods -n kube-system -l component=kube-controller-manager --show-labels
NAME                             READY   STATUS    RESTARTS   AGE    LABELS
kube-controller-manager-master   1/1     Running   0          110m   component=kube-controller-manager,tier=control-plane
```

Next, view the logs of the **kube-controller-manager** component by running the following command:

```bash
kubectl logs kube-controller-manager-master -n kube-systema --tail 10
```

Here's a sample of output logs showing that the **replicaset-controller** is polling the **kube-apiserver** for changes:

```shell
$ kubectl logs kube-controller-manager-master -n kube-system --tail 10 
I0709 13:30:10.948824       1 replica_set.go:676] "Finished syncing" logger="replicaset-controller" kind="ReplicaSet" key="default/simple-deployment-794f78c89" duration="68.198µs"
I0709 13:30:38.878189       1 replica_set.go:676] "Finished syncing" logger="replicaset-controller" kind="ReplicaSet" key="default/simple-deployment-794f78c89" duration="119.836µs"
I0709 13:30:38.919270       1 replica_set.go:676] "Finished syncing" logger="replicaset-controller" kind="ReplicaSet" key="default/simple-deployment-794f78c89" duration="74.039µs"
I0709 13:30:39.476992       1 replica_set.go:676] "Finished syncing" logger="replicaset-controller" kind="ReplicaSet" key="default/simple-deployment-794f78c89" duration="102.913µs"
I0709 13:30:39.567695       1 replica_set.go:676] "Finished syncing" logger="replicaset-controller" kind="ReplicaSet" key="default/simple-deployment-794f78c89" duration="92.123µs"
I0709 13:30:40.495761       1 replica_set.go:676] "Finished syncing" logger="replicaset-controller" kind="ReplicaSet" key="default/simple-deployment-794f78c89" duration="109.909µs"
I0709 13:30:40.523994       1 replica_set.go:676] "Finished syncing" logger="replicaset-controller" kind="ReplicaSet" key="default/simple-deployment-794f78c89" duration="81.243µs"
I0709 13:30:40.555419       1 replica_set.go:676] "Finished syncing" logger="replicaset-controller" kind="ReplicaSet" key="default/simple-deployment-794f78c89" duration="77.062µs"
I0709 13:30:40.595865       1 replica_set.go:676] "Finished syncing" logger="replicaset-controller" kind="ReplicaSet" key="default/simple-deployment-794f78c89" duration="83.422µs"
I0709 13:30:40.609762       1 replica_set.go:676] "Finished syncing" logger="replicaset-controller" kind="ReplicaSet" key="default/simple-deployment-794f78c89" duration="63.266µs"
```

It shows that the **replicaset-controller** is syncing the **ReplicaSet** `simple-deployment-794f78c89` with the desired state.