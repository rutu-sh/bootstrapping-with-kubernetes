# Architecture


## Control Plane - Data Plane Design Pattern

On a very high level, Kubernetes architecture can be divided into two planes: the **control plane** and the **data plane**. This design pattern used in distributed systems as a way for *separation of concerns*. It's fairly simple, the **control plane** makes decisions and the **data plane** carries out those decisions.

Keeping this in mind, let's dive into the architecture of Kubernetes.

## The Kubernetes Control Plane

