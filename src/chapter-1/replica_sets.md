# Replica Sets

Replica Sets manage the number of replicas of a pod running at any given time. 

## Why do we need Replica Sets?

Say you designed an application and deployed it on one or more pods. And you've setup a service to communicate with these pods. Ideally, you would want your application to be always available. That means, if a pod goes down, another pod should take its place. 

**How do you ensure this?**

Do you manually check if a pod is down and start another one? That's not a good idea, especially when you have thousands of pods running in your cluster. 

Replica Sets handle just this. You specify the intended scenario like "I'd like to have 3 replicas of this pod running at all times", and the Replica Set ensures that this is the case. 

A Replica Set continuously monitors the number of replicas of a pod and tries to match it with the desired number of replicas. If the number of replicas is less than the desired number, it starts a new pod. If the number of replicas is more than the desired number, it stops a pod.

