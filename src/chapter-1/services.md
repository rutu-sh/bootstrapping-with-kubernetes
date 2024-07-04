# Services

Services is how any communication to a pod is done in Kubernetes. 

## Why do we need services?

Let's say you manage to get your applications running in pods, and you have a pod for each frontend and backend application. And these pods should communicate with each other. 

**How do you do that?**

You look through and find out that every pod is assigned an IP Address. You hardcode these IP Addresses in your application code and deploy them. Would the pods still have the same IP Address when they are rescheduled? Most likely not. 

You want to hand over the responsibility of managing the IP Addresses to Kubernetes, and just program an *endpoint*, like `http://frontend` or `http://backend`, and let Kubernetes handle the rest. 

This is where services come in.

A service exposes a pod or a set of pods as a network service. This service has an IP Address and a DNS name. You can use this DNS name to communicate with the pods. 

**TLDR; any request that should go to a pod should be directed through a service.**
