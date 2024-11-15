# Pods - A deeper dive

In this chapter, we'll take a deeper look into pods by inspecting the internals of a pod. We'll explore the configuration of the pod's container, the network configuration, and the filesystem of the pod. Next, we'll look at the lifecycle of a pod and how it interacts with the Kubernetes API server. 

## Launch a pod 

Navigate to the `simple-pod` directory: 

```shell
cd bootstrapping-with-kubernetes-examples/deploy/simple-pod
```

Create a pod by running the following command:

```shell
kubectl apply -f pod.yaml 
```

```shell
$ kubectl apply -f pod.yaml 
pod/simple-pod created
```

List the pods to check if the pod is running:

```shell
$ kubectl get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP               NODE      NOMINATED NODE   READINESS GATES
simple-pod   1/1     Running   0          70s   192.168.189.66   worker2   <none>           <none>
```

## Inspecting the Pod 

The pod is running in the `worker2` node, let's see the details of the pod:

SSH into `worker2` node and run the following command:

```shell
docker container ls
```

```shell
worker2$ docker container ls
CONTAINER ID   IMAGE                               COMMAND                   CREATED          STATUS          PORTS     NAMES
0f5939ad0388   rutush10/simple-restapi-server-py   "/bin/sh -c '\"python…"   2 minutes ago    Up 2 minutes              k8s_simple-pod_simple-pod_default_a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a_0
82ebd1e524c3   registry.k8s.io/pause:3.9           "/pause"                  3 minutes ago    Up 3 minutes              k8s_POD_simple-pod_default_a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a_0
e846a7697b13   calico/node-driver-registrar        "/usr/bin/node-drive…"    34 minutes ago   Up 34 minutes             k8s_csi-node-driver-registrar_csi-node-driver-nbxnx_calico-system_7e156333-7fee-4ae9-bc6c-9364dcb91f7f_0
33001e4e0e05   calico/csi                          "/usr/bin/csi-driver…"    34 minutes ago   Up 34 minutes             k8s_calico-csi_csi-node-driver-nbxnx_calico-system_7e156333-7fee-4ae9-bc6c-9364dcb91f7f_0
d928c0344731   registry.k8s.io/pause:3.9           "/pause"                  34 minutes ago   Up 34 minutes             k8s_POD_csi-node-driver-nbxnx_calico-system_7e156333-7fee-4ae9-bc6c-9364dcb91f7f_1
31c2a5be5eba   calico/node                         "start_runit"             34 minutes ago   Up 34 minutes             k8s_calico-node_calico-node-c7ggl_calico-system_8ce36768-9d3d-42c0-ad21-ad532dc4ae8c_0
5c8359b97e29   registry.k8s.io/kube-proxy          "/usr/local/bin/kube…"    34 minutes ago   Up 34 minutes             k8s_kube-proxy_kube-proxy-p9dvm_kube-system_d12c6ee0-e804-4b9c-933d-7f49e32629a8_0
4cfb7018165f   registry.k8s.io/pause:3.9           "/pause"                  34 minutes ago   Up 34 minutes             k8s_POD_calico-node-c7ggl_calico-system_8ce36768-9d3d-42c0-ad21-ad532dc4ae8c_0
fa97f998bb9f   registry.k8s.io/pause:3.9           "/pause"                  34 minutes ago   Up 34 minutes             k8s_POD_kube-proxy-p9dvm_kube-system_d12c6ee0-e804-4b9c-933d-7f49e32629a8_0
```

We can see that the pod container is running on the `worker2` node with the name: `k8s_simple-pod_simple-pod_default_a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a_0`

The pod name can be broken down as follows:

- `k8s`: This is a prefix added by Kubernetes to all the containers it creates.
- `simple-pod`: This is the name of the pod's container, as specified in the manifest file.
- `simple_pod`: This is the name of the pod, as specified in the manifest file.
- `default`: This is the namespace in which the pod is running.
- `a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a`: This is the UID of the pod. 
- `0`: This is the container number. If there are multiple containers in the pod, they will be numbered starting from 0.


We can find pod's UID by running the following command:

```shell
kubectl get pod simple-pod -n default -o json | jq '.metadata.uid'
```

```shell
$ kubectl get pod simple-pod -n default -o json | jq '.metadata.uid'
"a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a"
```

Let's inspect the container by running the following command: 

```shell
worker-2:~$ docker inspect k8s_simple-pod_simple-pod_default_a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a_0
```

The output will be a JSON object with all the details of the container. Here is a snippet of the output:

```json
[
    {
        "Id": "0f5939ad0388a0780b65a1b208c6eaa65e6a99030a17cee62e1b676b4962501a",
        "Created": "2024-11-14T14:25:29.844718436Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "\"python3\" \"main.py\""
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 28263,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2024-11-14T14:25:30.488961484Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:90745eeb9a750a5fb4a92e804c7ab09727cf3bd8615e005333cf2f4fb15dafe4",
        "ResolvConfPath": "/var/lib/docker/containers/82ebd1e524c3b8920acb3cd0196cb34410cc80e44145626359b538ef2aff8578/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/82ebd1e524c3b8920acb3cd0196cb34410cc80e44145626359b538ef2aff8578/hostname",
        "HostsPath": "/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/etc-hosts",
        "LogPath": "/var/lib/docker/containers/0f5939ad0388a0780b65a1b208c6eaa65e6a99030a17cee62e1b676b4962501a/0f5939ad0388a0780b65a1b208c6eaa65e6a99030a17cee62e1b676b4962501a-json.log",
        "Name": "/k8s_simple-pod_simple-pod_default_a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a_0",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/volumes/kubernetes.io~projected/kube-api-access-qj5tq:/var/run/secrets/kubernetes.io/serviceaccount:ro",
                "/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/etc-hosts:/etc/hosts",
                "/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/containers/simple-pod/fbf17dce:/dev/termination-log"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "container:82ebd1e524c3b8920acb3cd0196cb34410cc80e44145626359b538ef2aff8578",
            "PortBindings": null,
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                0,
                0
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": null,
            "DnsOptions": null,
            "DnsSearch": null,
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "container:82ebd1e524c3b8920acb3cd0196cb34410cc80e44145626359b538ef2aff8578",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 999,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": [
                "seccomp=unconfined"
            ],
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 256,
            "Memory": 134217728,
            "NanoCpus": 0,
            "CgroupParent": "kubepods-burstable-poda4259cb7_26fc_47eb_87e9_d3e57ba7bb0a.slice",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 100000,
            "CpuQuota": 50000,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 134217728,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware",
                "/sys/devices/virtual/powercap"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/03cf4b8e9ea55337533d965b2d9e16778d4978f60812fa73bcc5749fb9f0bc20-init/diff:/var/lib/docker/overlay2/6a2c58abab266b7735aea477fb0ce40cd1882b2577ce0d8e5875a21379f217ef/diff:/var/lib/docker/overlay2/abb1bdc38b6369baf88ad83dac5aae128ecd6341ad7da8382f2f2466c3a32a7e/diff:/var/lib/docker/overlay2/623b0f4639ffb69a660668abadbb64b6e9db6828c9a781119856c2d5c0bb2d1e/diff:/var/lib/docker/overlay2/9dd03fb186fefb00540cb6d8fea2df15096733a19ed77ca0487dd1a97394115c/diff:/var/lib/docker/overlay2/07a611603cc0c62094196c9f8cf03de6c2bd41c9466e740ca9ee83f186f28da4/diff:/var/lib/docker/overlay2/f2b16c1c2ea43dfec7099840bf751af30ab16aa6d88f185e0557f3508e6d3543/diff:/var/lib/docker/overlay2/c3dd7c202af8747f01db40546eee4fb56ffbc389ee1b0e874de13b8875d44d67/diff:/var/lib/docker/overlay2/703d954d4c1bf29a37a836118dc97d6ae4d2542e28d94cb1823f4101bcd34db0/diff:/var/lib/docker/overlay2/82b9cbd96041be35306dd401329fa8c7036711134b01c1bc618d1a9a74afd213/diff:/var/lib/docker/overlay2/ccdebad004d4a8ba3dd4e2fa117fd1d67d324aefb992d497a43533c1ac0fe34a/diff",
                "MergedDir": "/var/lib/docker/overlay2/03cf4b8e9ea55337533d965b2d9e16778d4978f60812fa73bcc5749fb9f0bc20/merged",
                "UpperDir": "/var/lib/docker/overlay2/03cf4b8e9ea55337533d965b2d9e16778d4978f60812fa73bcc5749fb9f0bc20/diff",
                "WorkDir": "/var/lib/docker/overlay2/03cf4b8e9ea55337533d965b2d9e16778d4978f60812fa73bcc5749fb9f0bc20/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/volumes/kubernetes.io~projected/kube-api-access-qj5tq",
                "Destination": "/var/run/secrets/kubernetes.io/serviceaccount",
                "Mode": "ro",
                "RW": false,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/etc-hosts",
                "Destination": "/etc/hosts",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/containers/simple-pod/fbf17dce",
                "Destination": "/dev/termination-log",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
        "Config": {
            "Hostname": "simple-pod",
            "Domainname": "",
            "User": "0",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "KUBERNETES_SERVICE_HOST=10.96.0.1",
                "KUBERNETES_SERVICE_PORT=443",
                "KUBERNETES_SERVICE_PORT_HTTPS=443",
                "KUBERNETES_PORT=tcp://10.96.0.1:443",
                "KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443",
                "KUBERNETES_PORT_443_TCP_PROTO=tcp",
                "KUBERNETES_PORT_443_TCP_PORT=443",
                "KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1",
                "PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "GPG_KEY=E3FF2839C048B25C084DEBE9B26995E310250568",
                "PYTHON_VERSION=3.9.19",
                "PYTHON_PIP_VERSION=23.0.1",
                "PYTHON_SETUPTOOLS_VERSION=58.1.0",
                "PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/dbf0c85f76fb6e1ab42aa672ffca6f0a675d9ee4/public/get-pip.py",
                "PYTHON_GET_PIP_SHA256=dfe9fd5c28dc98b5ac17979a953ea550cec37ae1b47a5116007395bfacff2ab9",
                "PYTHONPATH=/app/src:"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "\"python3\" \"main.py\""
            ],
            "Healthcheck": {
                "Test": [
                    "NONE"
                ]
            },
            "Image": "rutush10/simple-restapi-server-py@sha256:be0b72b57e7a8e222eafe92fff61145352ef90c67819cd33c5edffbc4978f81c",
            "Volumes": null,
            "WorkingDir": "/app/src",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "annotation.io.kubernetes.container.hash": "62b86e75",
                "annotation.io.kubernetes.container.ports": "[{\"containerPort\":8000,\"protocol\":\"TCP\"}]",
                "annotation.io.kubernetes.container.restartCount": "0",
                "annotation.io.kubernetes.container.terminationMessagePath": "/dev/termination-log",
                "annotation.io.kubernetes.container.terminationMessagePolicy": "File",
                "annotation.io.kubernetes.pod.terminationGracePeriod": "30",
                "io.kubernetes.container.logpath": "/var/log/pods/default_simple-pod_a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/simple-pod/0.log",
                "io.kubernetes.container.name": "simple-pod",
                "io.kubernetes.docker.type": "container",
                "io.kubernetes.pod.name": "simple-pod",
                "io.kubernetes.pod.namespace": "default",
                "io.kubernetes.pod.uid": "a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a",
                "io.kubernetes.sandbox.id": "82ebd1e524c3b8920acb3cd0196cb34410cc80e44145626359b538ef2aff8578"
            },
            "StopSignal": "SIGINT"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "",
            "SandboxKey": "",
            "Ports": {},
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {}
        }
    }
]
``` 

Let's look at the following details of the container: 

```json
    "Image": "sha256:90745eeb9a750a5fb4a92e804c7ab09727cf3bd8615e005333cf2f4fb15dafe4",
    "ResolvConfPath": "/var/lib/docker/containers/82ebd1e524c3b8920acb3cd0196cb34410cc80e44145626359b538ef2aff8578/resolv.conf",
    "HostnamePath": "/var/lib/docker/containers/82ebd1e524c3b8920acb3cd0196cb34410cc80e44145626359b538ef2aff8578/hostname",
    "HostsPath": "/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/etc-hosts",
    "LogPath": "/var/lib/docker/containers/0f5939ad0388a0780b65a1b208c6eaa65e6a99030a17cee62e1b676b4962501a/0f5939ad0388a0780b65a1b208c6eaa65e6a99030a17cee62e1b676b4962501a-json.log",
    "Name": "/k8s_simple-pod_simple-pod_default_a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a_0",
```

- `Image`: This is the hash of the image that the container is running. 
- `ResolvConfPath`: This is the path to the resolv.conf file of the container. It contains the DNS configuration of the coredns server the container should use.
- `HostnamePath`: This is the path of the file containing the hostname of the container.
- `HostsPath`: This is the path of the file containing the kubernetes-managed hosts file of the container. It contains the IP address and hostname of the container.
- `LogPath`: This is the path of the log file of the container. 
- `Name`: This is the name of the container.

Notice that the ResolvConfPath and HostnamePath are part of another container with the ID `82ebd...`, with name `k8s_POD_simple-pod_default_a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a_0`. This is the [pause container](https://kubernetes.io/docs/concepts/windows/intro/#pause-container), also called the Sandbox container. This is a container used by Kubernetes to manage the lifecycle of the network namespace of the pod. If the worker container dies, the pause container will help in restarting another worker container with the same network configuration. All the containers in the pod share the network namespace of the pause container.


Let's look at the `resolv.conf` file of the container by running the following command: 

```shell
worker2$ sudo cat /var/lib/docker/containers/82ebd1e524c3b8920acb3cd0196cb34410cc80e44145626359b538ef2aff8578/resolv.conf 
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

The `resolv.conf` file contains details regarding the DNS configuration of the container. It contains the IP address of the coredns ClusterIP service that the container should use for DNS resolution. We verify this by running the following command to fetch the IP address of the coredns ClusterIP service:

```shell
$ kubectl get svc kube-dns -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   48m
```

The IP address of the coredns ClusterIP service is `10.96.0.10` which matches the IP address in the `resolv.conf` file.



The `etc-hosts` has the following content: 

```shell
worker2$ sudo cat /var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/etc-hosts 
# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
192.168.189.66  simple-pod
```

This file contains static mapping of IP addresses to hostnames. 

- `127.0.0.1    localhost`: This is the loopback address of the container. It allows the container to communicate with itself.
- `::1    localhost ip6-localhost ip6-loopback`: This is the IPv6 loopback address of the container.
- `fe00::0    ip6-localnet`: This is the link-local IPv6 address of the container. It is used for communication within the same network segment.
- `fe00::0    ip6-mcastprefix`: This is the multicast IPv6 address of the container. It is used for multicast communication.
- `fe00::1    ip6-allnodes`: This is the IPv6 address for reaching all nodes in the network.
- `fe00::2    ip6-allrouters`: This is the IPv6 address for reaching all routers in the network.
- `192.168.189.66	simple-pod`: This is the IPv4 address assigned to the pod. This allows any other service or pod in the cluster to communicate with the `simple-pod` pod using the hostname `simple-pod`.


The `HostConfig` section specifies how the container is configured to run on the Host. 

Let's take a look at the Volume mounts of the container:

```json
    "HostConfig": {
        "Binds": [
            "/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/volumes/kubernetes.io~projected/kube-api-access-qj5tq:/var/run/secrets/kubernetes.io/serviceaccount:ro",
            "/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/etc-hosts:/etc/hosts",
            "/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/containers/simple-pod/fbf17dce:/dev/termination-log"
        ],
    }
```

The `Binds` field contains the list of volumes that are mounted to the container. Here are the details of the volumes:

- `/var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/volumes/kubernetes.io~projected/kube-api-access-qj5tq:/var/run/secrets/kubernetes.io/serviceaccount:ro`: This is the volume that contains the certificate, namespace, and the token required for the pod to communicate with the Kubernetes API server. You can find the details of the volume by running the following command:

```shell
worker2$ cd /var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/volumes/kubernetes.io~projected/kube-api-access-qj5tq
worker2$ ls
ca.crt  namespace  token
```

The ca.crt file contains the certificate of the Kubernetes API server, the namespace file contains the namespace of the pod, and the token file contains the token required for the pod to authenticate with the Kubernetes API server. 

Read the contents of the ca.crt file by running the following command in the node running the pod:

```shell
worker2$ cat /var/lib/kubelet/pods/a4259cb7-26fc-47eb-87e9-d3e57ba7bb0a/volumes/kubernetes.io~projected/kube-api-access-qj5tq/ca.crt
```

Now run the following command on your local machine to get the ca.crt:

```shell
$ kubectl describe configmap kube-root-ca.crt
```

This should match the contents of the ca.crt file in the volume mounted to the container. 

The `kube-root-ca.crt` file is mounted by the Kubelet to the pod. This certificate allows the containerized applications to communicate with the K8s services protected by TLS, which includes communication to the K8s API server, and other services like kube-dns, etc. These certificates are signed by the K8s CA. 

Next, let's take a look at the "NetworkMode" field of the container:

```json
    "HostConfig": {
        "NetworkMode": "container:82ebd1e524c3b8920acb3cd0196cb34410cc80e44145626359b538ef2aff8578",
    }
```

As discussed earlier, the worker containers share the network namespace of the pause container. This is specified by the `NetworkMode` field. 

The `Config` section contains the configuration of the container. It contains the details like environment variables, command, health check, image, etc. The `Labels` field contains the associated labels of the container, which are used by Kubelet to manage the lifecycle of the container. 

The `NetworkSettings` section contains the details of the network configuration of the container. It contains details like the IP address, MAC address, and the network namespace of the container. Since this container shares the network namespace of the pause container, the IP address and MAC address are not assigned to the container but are specified in the pause (sandbox) container.