# Deployments

Deployments control the Replica Sets. 

## Why do we need Deployments?

Say you have a Replica Set managing the number of replicas of a pod. And you've setup a service to communicate with these pods.

Now, you create a new version of your application and want to deploy it. You could simply update the description of the pod in the Replica Set. And let the Replica Set handle the rest. 

But what if the new version of the application has a bug? Or what if you want to rollback to the previous version?

This is where Deployments come in.

A Deployment is a higher-level concept which manages Replica Sets. It allows you to deploy new versions of your application, rollback to a previous version, and scale the application up and down.


**A deployment is how you truly manage your application on Kubernetes.**