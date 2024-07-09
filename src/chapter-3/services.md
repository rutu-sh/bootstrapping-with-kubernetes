# Services

A [Service](../chapter-1/services.md) is an abstraction that enables communication to the Pods. It provides a stable endpoint for the pods so that we don't have to worry about the dynamic allocation of IP addresses to the Pods. Services can be of different types, such as ClusterIP, NodePort, LoadBalancer, and ExternalName. Here, I'll cover only the ClusterIP and NodePort type services.