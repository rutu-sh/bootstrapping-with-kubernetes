# Cluster and Kubernetes

## What is a Cluster?

Let's say you design a system of services which interact with each other to provide a functionality. You have containerized these services and want to deploy them.

Assume you have 3 spare laptops, and want to run your services on them. Ideally, you would want to connect your laptops together and run your services on them. This interconnected set of laptops is what we call a **Cluster**. In the context of **Kubernetes**, a cluster is a set of machines which run your containerized applications.


## What is Kubernetes? 

According to the [official documentation](https://kubernetes.io/)
> Kubernetes, also known as K8s, is an open source system for automating deployment, scaling, and management of containerized applications.

Let's simplify this a bit. 

When you're deploying applications on a cluster of machines you would like to have the following issues addressed:

1. **High Availability**: You want the applications to be highly available. If instance of an application goes down, you want another instance to take over. 

2. **Scalability**: You want your application to scale up and down as per the load. 

3. **Networking**: You want the applications to be able to communicate with each other. 

... and many more. 

This is the job of Kubernetes. It provides you with a set of tools to manage your applications on a cluster of machines.
