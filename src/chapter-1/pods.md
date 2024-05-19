# Pods  

[Pods](https://kubernetes.io/docs/concepts/workloads/pods/) are the simplest Kubernetes resources that you can manage. A pod is a group of one or more containers that share network and storage. 

## Why not just manage containers? 

You might be wondering why we need pods when we can manage containers directly. 

Though it might seem reasonable to simply schedule containers, issues might arise when those containers share dependencies and must be scheduled together to simplify the sharing of networking and memory resources. If we only manage containers, then in a highly distributed system the interdependent containers might get scheduled in different and far-apart machines — introducing delays within the system. Hence, pods were introduced as an abstraction over the containers.

This multi-container capability of pods is not commonly used in deploying applications: *you would usually end up running a single container inside a pod*. However, it has also given rise to design patterns like sidecar (we’ll cover this later) used in tools like [istio](https://www.istio.io) (also later).

**From here on, unless specified, we will be referring to a pod as a single container running inside it.**