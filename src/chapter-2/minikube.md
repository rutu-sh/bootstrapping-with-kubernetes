# Minikube

This is the easiest way to get started with Kubernetes. 

Head on to the [Minikube installation page](https://minikube.sigs.k8s.io/docs/start/) and just follow the instructions, be sure to have the dependencies satisfied. The [official documentation](https://minikube.sigs.k8s.io/docs/) has a lot of information on how to use Minikube. Here are some commands you might find useful:


Once you have Minikube installed, you can start a cluster by running the following command:

```bash
minikube start
```

Minikube has a concept called `profile`. You can create a new profile by running the following command:

```bash
minikube start -p <profile-name>
```

To set this profile as the default

```bash
minikube profile <profile-name>
```

This isolates the cluster from other clusters you might have running on your machine.

You can also simulate a cluster with three nodes by running the following command:

```bash
minikube start -p <profile-name> --nodes 3
```

To view the Kubernetes dashboard, run the following command:

```bash
minikube dashboard
```