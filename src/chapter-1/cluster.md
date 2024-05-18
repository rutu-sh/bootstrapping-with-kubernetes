# Cluster

## What is a Cluster?

Let's say you design a system of services which can interact with each other to provide a functionality. You have containerized these services and want to deploy them.

Assume you have 3 spare laptops, and want to run your services on them. Ideally, you would want to connect your laptops together and make an abstraction such that all the three laptops appear as a single machine at your disposal. This abstraction is called a **Cluster**.

A cluster is just an interconnected set of machines which work together to provide a service. In the context of Kubernetes, a cluster is a set of machines which run your containerized applications.