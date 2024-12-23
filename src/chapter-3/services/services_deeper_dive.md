# Services - A deeper dive

Now that we have a basic understanding of Kubernetes Services, let's go a level deeper and figure out how they work under the hood.

This is the most elaborate chapter I have written till date, about 4 days of reasearch and writing has gone into this. I am proud of this chapter, and I hope you find it useful. 

> **Note**: This chapter is only inteded for a deeper understanding of Kubernetes, can be skipped.


## Launch a simple web application with a service

Let's start by deploying a simple web application and expose it using a ClusterIP service. The folllowing steps will launch a deployment with 3 pods and expose it using a ClusterIP service named `backend-service`.

Navigate to the `simple-service` directory:

```shell
cd bootstrapping-with-kubernetes-examples/deploy/simple-service
```

Create a deployment and a service by running the following command: 

```shell
kubectl apply -f deployment.yaml -f service.yaml 
```

```shell
$ kubectl apply -f deployment.yaml -f service.yaml 
deployment.apps/simple-deployment created
service/backend-service created
```

List the pods and services in the cluster:

```shell
kubectl get pods -o wide
```

```shell
$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE    IP                NODE      NOMINATED NODE   READINESS GATES
simple-deployment-794f78c89-4646m   1/1     Running   0          132m   192.168.189.66    worker2   <none>           <none>
simple-deployment-794f78c89-q6k4p   1/1     Running   0          132m   192.168.235.130   worker1   <none>           <none>
simple-deployment-794f78c89-x4tzb   1/1     Running   0          132m   192.168.219.71    master    <none>           <none> 
```

This output shows that the deployment has created three pods, running on master, worker1, and worker2 nodes with IPs `192.168.219.71`, `192.168.235.130`, and `192.168.189.66` respectively. 

List the services in the cluster:

```shell
kubectl get service -o wide
```

```shell
$ kubectl get service -o wide
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE    SELECTOR
backend-service   ClusterIP   10.97.199.177   <none>        8000/TCP   133m   app=simple-deployment
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP    138m   <none>
```

From the above output, we can see that the service `backend-service` is of type `ClusterIP` and has the IP address `10.97.199.177`. 

List the endpoints belonging to the service: 

```shell
kubectl get endpoints
``` 

```shell
$ kubectl get endpoints
NAME              ENDPOINTS                                                      AGE
backend-service   192.168.189.66:8000,192.168.219.71:8000,192.168.235.130:8000   135m
kubernetes        <master-node-ip>:6443                                            141m
```

This output shows that the service `backend-service` has three endpoints corresponding to the three pods. 

We will insvestigate the `worker1` node and see how the routing works in a ClusterIP service. 

## Sending requests to the ClusterIP service

Now that our setup is ready, let's see how the traffic routing works in a ClusterIP service.

First, let's create a pod for testing purposes.

```shell
kubectl run -it --rm debug --image=alpine -n default --restart=Never -- sh
```

The output of the above command will be:

```shell
$ kubectl run -it --rm debug --image=alpine -n default --restart=Never -- sh
If you don't see a command prompt, try pressing enter.
/ #
```

This will launch a pod in the default namespace with the busybox image.

Send the request to the pods through the `backend-service`:

First install curl

```shell
apk add curl
```

Now, send a request to the service:

```shell
curl -i backend-service.default.svc.cluster.local:8000/rest/v1/health
```


```shell
/ # apk add curl
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/community/x86_64/APKINDEX.tar.gz
(1/9) Installing brotli-libs (1.1.0-r2)
(2/9) Installing c-ares (1.34.3-r0)
(3/9) Installing libunistring (1.2-r0)
(4/9) Installing libidn2 (2.3.7-r0)
(5/9) Installing nghttp2-libs (1.64.0-r0)
(6/9) Installing libpsl (0.21.5-r3)
(7/9) Installing zstd-libs (1.5.6-r1)
(8/9) Installing libcurl (8.11.1-r0)
(9/9) Installing curl (8.11.1-r0)
Executing busybox-1.37.0-r8.trigger
OK: 12 MiB in 24 packages
/ #
/ #
/ #
/ # curl -i backend-service.default.svc.cluster.local:8000/rest/v1/health
HTTP/1.1 307 Temporary Redirect
date: Mon, 23 Dec 2024 04:37:31 GMT
server: uvicorn
content-length: 0
location: http://backend-service.default.svc.cluster.local:8000/rest/v1/health/
/ #
```

Now that we can confirm that the service is working as expected, let's investigate how the traffic routing works in a ClusterIP service.

## Investigating the DNS resolution in a ClusterIP service

Run the following command to perform the DNS lookup for the service:

```shell
nslookup backend-service.default.svc.cluster.local
```

```shell
/ # nslookup backend-service.default.svc.cluster.local
Server:		10.96.0.10
Address:	10.96.0.10:53


Name:	backend-service.default.svc.cluster.local
Address: 10.97.199.177

/ #
```

The command `nslookup backend-service.default.svc.cluster.local` resolves the service name to an IP address. The `etc/resolv.conf` file in the pod contains the IP address of the DNS server. Let's check the contents of this file: 

```shell
/ # cat /etc/resolv.conf
```

```shell
/ # cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

This config file specifies that the for the domain `default.svc.cluster.local`, the DNS server is at the address `10.96.0.10`. Any request to resolve a domain name will be sent to this DNS server. This DNS server is the kube-dns service running in the cluster. The output of the nslookup command shows that the DNS server resolved the service name to the IP address `10.97.199.177`, which is the IP address of the service `backend-service` in the cluster. 

> Now we know that whenever a pod tries to reach the service `backend-service`, the DNS server will resolve the service name to the IP address of the service, i.e. `10.97.199.177`. 

Exit the pod by running the command:

```shell
exit
```

## Investigating the iptables rules in a ClusterIP service

In the `worker1` node, run the following command to list the iptables rules:

```shell
sudo iptables -t nat -L -v -n --line-numbers
```

This produces the following output. If it feels overwhelming, it is. Not to worry, we will break it down step by step.

```shell
rutu_sh@worker-1:~$ sudo iptables -t nat -L -v -n --line-numbers
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1    21780 2164K cali-PREROUTING  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:6gwbT8clXdHdC1b1 */
2    21782 2164K KUBE-SERVICES  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */
3    18533  965K DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1    80410 5807K cali-OUTPUT  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:tVnHkvAo15HuiPy0 */
2    80943 5846K KUBE-SERVICES  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */
3        0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1    80586 5819K cali-POSTROUTING  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:O3lYWMrLQYEMJtB5 */
2    80989 5850K KUBE-POSTROUTING  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes postrouting rules */
3        0     0 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0

Chain DOCKER (2 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0

Chain KUBE-KUBELET-CANARY (0 references)
num   pkts bytes target     prot opt in     out     source               destination

Chain KUBE-MARK-MASQ (21 references)
num   pkts bytes target     prot opt in     out     source               destination
1        1    60 MARK       all  --  *      *       0.0.0.0/0            0.0.0.0/0            MARK or 0x4000

Chain KUBE-NODEPORTS (1 references)
num   pkts bytes target     prot opt in     out     source               destination

Chain KUBE-POSTROUTING (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1     3010  219K RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0
2        1    60 MARK       all  --  *      *       0.0.0.0/0            0.0.0.0/0            MARK xor 0x4000
3        1    60 MASQUERADE  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service traffic requiring SNAT */ random-fully

Chain KUBE-PROXY-CANARY (0 references)
num   pkts bytes target     prot opt in     out     source               destination

Chain KUBE-SEP-4OII2ED73B73X74G (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.1.1          0.0.0.0/0            /* calico-system/calico-typha:calico-typha */
2        0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* calico-system/calico-typha:calico-typha */ tcp to:192.168.1.1:5473

Chain KUBE-SEP-5ZS5Z45RTWDMUVBI (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.219.68       0.0.0.0/0            /* kube-system/kube-dns:dns */
2       17  1619 DNAT       udp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns */ udp to:192.168.219.68:53

Chain KUBE-SEP-A76J55YYNSUXC5E2 (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.189.66       0.0.0.0/0            /* default/backend-service */
2        2   120 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/backend-service */ tcp to:192.168.189.66:8000

Chain KUBE-SEP-BAX4QE7OCAYMNTNV (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.219.69       0.0.0.0/0            /* calico-apiserver/calico-api:apiserver */
2        0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* calico-apiserver/calico-api:apiserver */ tcp to:192.168.219.69:5443

Chain KUBE-SEP-CFVSV7QPAUXMFFMO (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.219.66       0.0.0.0/0            /* kube-system/kube-dns:dns */
2       26  2373 DNAT       udp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns */ udp to:192.168.219.66:53

Chain KUBE-SEP-HWQSL37KRW77KJ7F (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.219.70       0.0.0.0/0            /* calico-apiserver/calico-api:apiserver */
2        0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* calico-apiserver/calico-api:apiserver */ tcp to:192.168.219.70:5443

Chain KUBE-SEP-JLPVI6ZC3UHQ52RC (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.219.71       0.0.0.0/0            /* default/backend-service */
2        0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/backend-service */ tcp to:192.168.219.71:8000

Chain KUBE-SEP-PLASFI6HQXMOROMY (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       <worker1-node-ip>       0.0.0.0/0            /* calico-system/calico-typha:calico-typha */
2        0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* calico-system/calico-typha:calico-typha */ tcp to:<worker1-node-ip>:5473

Chain KUBE-SEP-QNYF7SNFEW3HR3FG (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.219.66       0.0.0.0/0            /* kube-system/kube-dns:dns-tcp */
2        0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp */ tcp to:192.168.219.66:53

Chain KUBE-SEP-RIEXMMKJKAVQ3ODE (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.219.68       0.0.0.0/0            /* kube-system/kube-dns:dns-tcp */
2        0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp */ tcp to:192.168.219.68:53

Chain KUBE-SEP-RT6T3NHUDRL6ZXKL (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.235.130      0.0.0.0/0            /* default/backend-service */
2        0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/backend-service */ tcp to:192.168.235.130:8000

Chain KUBE-SEP-RYBYHU6GQR7ZWHEC (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.219.66       0.0.0.0/0            /* kube-system/kube-dns:metrics */
2        0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:metrics */ tcp to:192.168.219.66:9153

Chain KUBE-SEP-UP73WHDVCRV7NRDO (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       192.168.219.68       0.0.0.0/0            /* kube-system/kube-dns:metrics */
2        0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:metrics */ tcp to:192.168.219.68:9153

Chain KUBE-SEP-YCHPF2YV4RSURBSB (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       <master-node-ip>       0.0.0.0/0            /* default/kubernetes:https */
2       34  2040 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/kubernetes:https */ tcp to:<master-node-ip>:6443

Chain KUBE-SERVICES (2 references)
num   pkts bytes target     prot opt in     out     source               destination
1        1    60 KUBE-SVC-NPX46M4PTMTKRN6Y  tcp  --  *      *       0.0.0.0/0            10.96.0.1            /* default/kubernetes:https cluster IP */
2        0     0 KUBE-SVC-JD5MR3NA4I4DYORP  tcp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:metrics cluster IP */
3        0     0 KUBE-SVC-TCOU7JCQXEZGVUNU  udp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns cluster IP */
4        0     0 KUBE-SVC-ERIFXISQEP7F7OF4  tcp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns-tcp cluster IP */
5        0     0 KUBE-SVC-I24EZXP75AX5E7TU  tcp  --  *      *       0.0.0.0/0            10.97.160.76         /* calico-apiserver/calico-api:apiserver cluster IP */
6        0     0 KUBE-SVC-RK657RLKDNVNU64O  tcp  --  *      *       0.0.0.0/0            10.101.222.35        /* calico-system/calico-typha:calico-typha cluster IP */
7        0     0 KUBE-SVC-ZZAJ2COS27FT6J6V  tcp  --  *      *       0.0.0.0/0            10.97.199.177        /* default/backend-service cluster IP */
8     3039  194K KUBE-NODEPORTS  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service nodeports; NOTE: this must be the last rule in this chain */ ADDRTYPE match dst-type LOCAL

Chain KUBE-SVC-ERIFXISQEP7F7OF4 (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  tcp  --  *      *      !192.168.0.0/16       10.96.0.10           /* kube-system/kube-dns:dns-tcp cluster IP */
2        0     0 KUBE-SEP-QNYF7SNFEW3HR3FG  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp -> 192.168.219.66:53 */ statistic mode random probability 0.50000000000
3        0     0 KUBE-SEP-RIEXMMKJKAVQ3ODE  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp -> 192.168.219.68:53 */

Chain KUBE-SVC-I24EZXP75AX5E7TU (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  tcp  --  *      *      !192.168.0.0/16       10.97.160.76         /* calico-apiserver/calico-api:apiserver cluster IP */
2        0     0 KUBE-SEP-BAX4QE7OCAYMNTNV  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* calico-apiserver/calico-api:apiserver -> 192.168.219.69:5443 */ statistic mode random probability 0.50000000000
3        0     0 KUBE-SEP-HWQSL37KRW77KJ7F  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* calico-apiserver/calico-api:apiserver -> 192.168.219.70:5443 */

Chain KUBE-SVC-JD5MR3NA4I4DYORP (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  tcp  --  *      *      !192.168.0.0/16       10.96.0.10           /* kube-system/kube-dns:metrics cluster IP */
2        0     0 KUBE-SEP-RYBYHU6GQR7ZWHEC  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:metrics -> 192.168.219.66:9153 */ statistic mode random probability 0.50000000000
3        0     0 KUBE-SEP-UP73WHDVCRV7NRDO  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:metrics -> 192.168.219.68:9153 */

Chain KUBE-SVC-NPX46M4PTMTKRN6Y (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1       34  2040 KUBE-MARK-MASQ  tcp  --  *      *      !192.168.0.0/16       10.96.0.1            /* default/kubernetes:https cluster IP */
2       34  2040 KUBE-SEP-YCHPF2YV4RSURBSB  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/kubernetes:https -> <master-node-ip>:6443 */

Chain KUBE-SVC-RK657RLKDNVNU64O (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  tcp  --  *      *      !192.168.0.0/16       10.101.222.35        /* calico-system/calico-typha:calico-typha cluster IP */
2        0     0 KUBE-SEP-PLASFI6HQXMOROMY  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* calico-system/calico-typha:calico-typha -> <worker1-node-ip>:5473 */ statistic mode random probability 0.50000000000
3        0     0 KUBE-SEP-4OII2ED73B73X74G  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* calico-system/calico-typha:calico-typha -> 192.168.1.1:5473 */

Chain KUBE-SVC-TCOU7JCQXEZGVUNU (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  udp  --  *      *      !192.168.0.0/16       10.96.0.10           /* kube-system/kube-dns:dns cluster IP */
2       26  2373 KUBE-SEP-CFVSV7QPAUXMFFMO  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns -> 192.168.219.66:53 */ statistic mode random probability 0.50000000000
3       17  1619 KUBE-SEP-5ZS5Z45RTWDMUVBI  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns -> 192.168.219.68:53 */

Chain KUBE-SVC-ZZAJ2COS27FT6J6V (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  tcp  --  *      *      !192.168.0.0/16       10.97.199.177        /* default/backend-service cluster IP */
2        2   120 KUBE-SEP-A76J55YYNSUXC5E2  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/backend-service -> 192.168.189.66:8000 */ statistic mode random probability 0.33333333349
3        0     0 KUBE-SEP-JLPVI6ZC3UHQ52RC  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/backend-service -> 192.168.219.71:8000 */ statistic mode random probability 0.50000000000
4        0     0 KUBE-SEP-RT6T3NHUDRL6ZXKL  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/backend-service -> 192.168.235.130:8000 */

Chain cali-OUTPUT (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1    80410 5807K cali-fip-dnat  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:GBTAv2p5CwevEyJm */

Chain cali-POSTROUTING (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1    80586 5819K cali-fip-snat  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:Z-c7XtVd2Bq7s_hA */
2    80586 5819K cali-nat-outgoing  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:nYKhEzDlr11Jccal */
3        0     0 MASQUERADE  all  --  *      vxlan.calico  0.0.0.0/0            0.0.0.0/0            /* cali:e9dnSgSVNmIcpVhP */ ADDRTYPE match src-type !LOCAL limit-out ADDRTYPE match src-type LOCAL random-fully

Chain cali-PREROUTING (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1    21780 2164K cali-fip-dnat  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:r6XmIziWUJsdOK6Z */

Chain cali-fip-dnat (2 references)
num   pkts bytes target     prot opt in     out     source               destination

Chain cali-fip-snat (1 references)
num   pkts bytes target     prot opt in     out     source               destination

Chain cali-nat-outgoing (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1      130  7800 MASQUERADE  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:flqWnvo8yq4ULQLa */ match-set cali40masq-ipam-pools src ! match-set cali40all-ipam-pools dst random-fully
```

The following is a detailed visualization of these rules, this might help understand the description that follows. You can follow the description here to understand the rules better. 

<iframe width="768" height="432" src="https://miro.com/app/live-embed/uXjVL0UGJIg=/?moveToViewport=-1798,-1264,6631,3758&embedId=737082074366" frameborder="0" scrolling="no" allow="fullscreen; clipboard-read; clipboard-write" allowfullscreen></iframe>

The iptable rules have few important that must be understood, here is a reference of the same. 

| IP Address | Port (s) | Description | 
|------------|-----|-------------|
| 10.97.199.177 | 8000/tcp | ClusterIP service `backend-service` |
| 192.168.189.66 | 8000/tcp | simple-deployment's pod running on worker2 node | 
| 192.168.235.130 | 8000/tcp | simple-deployment's pod running on worker1 node |
| 192.168.219.71 | 8000/tcp | simple-deployment's pod running on master node |
| 10.96.0.10 | 53/udp, 53/tcp, 9153/tcp (metrics) | ClusterIP service `kube-dns` |
| 192.168.219.66 | 53/udp, 53/tcp, 9153/tcp (metrics) | coredns pod running on master node |
| 192.168.219.68 | 53/udp, 53/tcp, 9153/tcp (metrics) | coredns pod running on master node | 
| 10.97.160.76 | 443:5443/tcp | ClusterIP service `calico-apiserver` | 
| 192.168.219.69 | 5443/tcp | calico-apiserver pod running on master node |
| 192.168.219.70 | 5443/tcp | calico-apiserver pod running on master node |
| 10.101.222.35 | 5473 | ClusterIP service `calico-typha` |
| 192.168.1.1 | 5473 | calico-typha pod running on master node |
| \<worker1-node-ip\> | 5473 | calico-typha pod running on worker1 node |
| 10.96.0.1 | 443:6443/tcp | ClusterIP service `kubernetes` i.e. the kube-api server |
| \<master-node-ip\> |  6443 | kube-api server running on master node |


Using this information, we take the following example: 

> A pod in `worker1` node tries to reach the service `backend-service`. The pod first makes a request to the DNS server at `10.96.0.10:53` to resolve the service name to an IP address. The DNS server resolves the service name to the IP address `10.97.199.177`. The pod then sends the request to the IP address `10.97.199.177`. 

#### Phase 1: DNS resolution

Since the request is originating from inside the `worker1` node, i.e. it first exits the pod's network namespace and enters the host's network namespace. 

Therefore, the first chain to execute is the `OUTPUT` chain and then the `PREROUTING` chain.

1. The `OUTPUT` chain has the following rule: 

```shell
Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1    80410 5807K cali-OUTPUT  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:tVnHkvAo15HuiPy0 */
2    80943 5846K KUBE-SERVICES  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */
3        0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL
```

OUTPUT's Rule 1 applies `cali-OUTPUT` chain. This chain is a part of the Calico CNI plugin, it helps handle the DNAT ( destination Network Address Translation ). 
OUTPUT's Rule 2 applies the `KUBE-SERVICES` chain. This chain is used to handle the traffic for the ClusterIP services. 


Let's explore further into the `KUBE-SERVICES` chain.

```shell
Chain KUBE-SERVICES (2 references)
num   pkts bytes target     prot opt in     out     source               destination
1        1    60 KUBE-SVC-NPX46M4PTMTKRN6Y  tcp  --  *      *       0.0.0.0/0            10.96.0.1            /* default/kubernetes:https cluster IP */
2        0     0 KUBE-SVC-JD5MR3NA4I4DYORP  tcp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:metrics cluster IP */
3        0     0 KUBE-SVC-TCOU7JCQXEZGVUNU  udp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns cluster IP */
4        0     0 KUBE-SVC-ERIFXISQEP7F7OF4  tcp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns-tcp cluster IP */
5        0     0 KUBE-SVC-I24EZXP75AX5E7TU  tcp  --  *      *       0.0.0.0/0            10.97.160.76         /* calico-apiserver/calico-api:apiserver cluster IP */
6        0     0 KUBE-SVC-RK657RLKDNVNU64O  tcp  --  *      *       0.0.0.0/0            10.101.222.35        /* calico-system/calico-typha:calico-typha cluster IP */
7        0     0 KUBE-SVC-ZZAJ2COS27FT6J6V  tcp  --  *      *       0.0.0.0/0            10.97.199.177        /* default/backend-service cluster IP */
8     3039  194K KUBE-NODEPORTS  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service nodeports; NOTE: this must be the last rule in this chain */ ADDRTYPE match dst-type LOCAL
```

KUBE-SERVICES' Rule 1 applies to the packets destined to the kube api server running at `10.96.0.1`. It is defined as follows: 

```shell
Chain KUBE-SVC-NPX46M4PTMTKRN6Y (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1       34  2040 KUBE-MARK-MASQ  tcp  --  *      *      !192.168.0.0/16       10.96.0.1            /* default/kubernetes:https cluster IP */
2       34  2040 KUBE-SEP-YCHPF2YV4RSURBSB  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/kubernetes:https -> <master-node-ip>:6443 */
```

KUBE-SVC-NPX46M4PTMTKRN6Y's 