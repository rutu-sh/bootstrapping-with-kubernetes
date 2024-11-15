# Summary
[Bootstrapping With Kubernetes](./cover/cover.md)
[Introduction](./chapter-0/chapter_0.md)

# Kubernetes Basics

- [Concepts](./chapter-1/chapter_1.md)
    - [Cluster and Kubernetes](./chapter-1/cluster.md)
    - [Nodes](./chapter-1/nodes.md)
    - [Pods](./chapter-1/pods.md)
    - [Services](./chapter-1/services.md)
    - [Replica Sets](./chapter-1/replica_sets.md)
    - [Deployments](./chapter-1/deployments.md)
    - [Quick Recap](./chapter-1/quick_recap.md)
    - [Architecture](./chapter-1/architecture.md)
    - [Architecture - Behind the Scenes](./chapter-1/architecture_bts.md)
- [Installation and Setup](./chapter-2/chapter_2.md)
    - [Minikube](./chapter-2/minikube.md)
    - [A local Kubernetes setup from scratch](./chapter-2/local_k8s_setup.md)
    - [CloudLab - For Researchers and Educators](./chapter-2/cloudlab.md)
- [Common Resources](./chapter-3/chapter_3.md)
    - [Declarative vs. Imperative object management](./chapter-3/declarative_vs_imperative.md)
    - [Kube System Components](./chapter-3/kube_system_components.md)
        - [kube-dns](./chapter-3/kube-system/kube-dns.md)
        - [etcd](./chapter-3/kube-system/etcd.md)
        - [kube-apiserver](./chapter-3/kube-system/kube-apiserver.md)
        - [kube-controller-manager](./chapter-3/kube-system/kube-controller-manager.md)
        - [kube-proxy](./chapter-3/kube-system/kube-proxy.md)
        - [kube-scheduler](./chapter-3/kube-system/kube-scheduler.md)
    - [Workloads](./chapter-3/workloads.md)
        - [Pods](./chapter-3/pods.md)
            - [Pods - A deeper dive](./chapter-3/pods_deeper_dive.md)
        - [Replica Sets](./chapter-3/replica_sets.md)
        - [Deployments](./chapter-3/deployments.md)
    - [Namespaces]()
    - [Networking](./chapter-3/networking.md)
        - [Services](./chapter-3/services.md)
            - [ClusterIP Service](./chapter-3/services/clusterip.md)
            - [NodePort Service](./chapter-3/services/nodeport.md)
            - [Using Port Names](./chapter-3/services/port_names.md)
            - [Services - A deeper dive](./chapter-3/services/services_deeper_dive.md)
    - [Configuration]()
        - [ConfigMaps]()
        - [Secrets]()
    - [Storage]()
        - [Storage Classes]()
        - [Persistent Volumes]()
        - [Persistent Volume Claims]()

# Kubernetes Intermediate

- [Workloads]()
    - [Jobs]()
    - [CronJobs]()
    - [StatefulSets]()
    - [DaemonSets]()
    - [Horizontal Pod Autoscaler]()
- [Networking]()
    - [Ingress]() 
    - [Gateway API]()
<!-- - [Access Control]()
    - [RBAC]()
    - [Service Accounts]()
- [Fault Tolerance]()
    - [Liveness and Readiness Probes]()
    - [Pod Disruption Budgets]()
- [Monitoring and Logging]()
    - [Metrics Server]()
    - [Prometheus]()
    - [Grafana]()
    - [Elasticsearch]()
    - [Fluentd]()
    - [Kibana]() -->
 
<!-- # Advanced Usage I:  

- [Custom Resource Definitions]()
- [Operators]() 
- [Helm]() 

-->

<!-- 

# Advanced Usage II:

# Service Mesh
    ## Istio

# Serverless
    ## Knative

# MLOps
    ## Kubeflow
    ## MLflow

# Observability and Monitoring
    ## Prometheus
    ## OpenTelemetry
    ## Grafana

# Event-Driven Architecture
    ## Kafka
    ## TriggerMesh
-->

---

# Supplementary

- [Developing Applications](./supplementary/developing-applications/developing_applications.md)
    - [Controller Service Repository Pattern](./supplementary/developing-applications/controller_service_repository_pattern.md)
    - [Building a Python FastAPI application](./supplementary/developing-applications/building_fastapi_app.md)
    - [Building a Go application]()

