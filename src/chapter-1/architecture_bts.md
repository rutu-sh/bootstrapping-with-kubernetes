# Architecture - Behind the Scenes

We have gone through the high-level architecture of Kubernetes. And we've learned what's the purpose of each component in the architecture. Now let's get a bit deeper and understand how everything comes together behind the scenes. 

I'm taking the following scenario and show you how it'll work in the background. 

> You have three spare laptops running linux, and you want to create a Kubernetes cluster using these laptops. And you are assigned with a task to create a web application. There are two microservices, a frontend microservice and a backend microservice. You want to deploy these microservices over the cluster such that each microservice has three replicas and can communicate with each other. To keep it simple, I'm not going to use any cloud services or databases. 

***Note that this is an example scenario, don't run these commands on your machines. This is just to give you an idea of the underlying steps.***

## Setting up the Cluster

To create a Kubernetes cluster using these laptops you need to install some Kubernetes-specific software on each laptop. Let's call these laptops as nodes. We have three nodes, `node-1` will be our **Master Node** and `node-2` and `node-3` will be our **Worker Nodes**.

#### Master Node - Control Plane

The **Master Node** will have the following **Control Plane** components:

1. **API Server**: The API Server is how any interaction with the cluster happens.

2. **Scheduler**: The Scheduler is responsible for scheduling the pods on the nodes.

3. **Controller Manager**: The Controller Manager is responsible for managing the controllers that manage the resources in the cluster.

4. **etcd**: The `etcd` is a distributed key-value store that Kubernetes uses to store all of its data.

5. **Cloud Controller Manager**: We'll skip this component for simplicity. It's not as important from the perspective of understanding the architecture in this scenario.

#### Worker Nodes - Data Plane

The following components run on all worker nodes in the cluster, these are the components that make up the **Data Plane** of Kubernetes:

1. **Container Runtime**: Essentially, running any workload on Kubernetes comes down to spinning up a container. Therefore, each node in the cluster should have a container runtime. 

2. **Kubelet**: The Kubelet is responsible for interacting with the API Server and the container runtime. It's the Kubelet's job to make sure that the containers are running as expected.

3. **Kube Proxy**: The Kube Proxy will handle all the network-related operations within the node. 

![](./assets/k8s_example_scenario_1.drawio.svg)


*It must be noted that you can run the data plane components on the Master Node as well. I'm not showing that for the sake of simplicity.*

<br>

Now that we have set up the cluster, let's move on to creating the microservices

## Designing the Microservices

We have two microservices, a frontend microservice and a backend microservice.

#### Frontend Microservice

This is a simple node.js server that serves a static HTML page. The frontend microservice will be listening on port `3000`. This application makes REST API calls to the backend microservice. 

And you've designed it such that it takes the endpoint of the backend microservice as an environment variable. We'll call this environment variable `BACKEND_ENDPOINT`. 

#### Backend Microservice

This is a simple Go server that serves a JSON response. The backend microservice will be listening on port `8080`. 


## Designing the Deployment and Services

We have the following requirements for the frontend microservice: 

1. It should have three replicas running at all times.
2. It should be able to communicate with the backend microservice.

We have the following requirements for the backend microservice:

1. It should have three replicas running at all times.
2. It should be reachable by an external client. 


### Frontend Microservice Deployment

Using the requirements, we design the following specification. 

**Do not worry about the specification, we'll get through that in the next chapter, but for now, we're only concerned with a few key points in the specification.**

The following is the deployment specification for the frontend microservice:

`frontend-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: frontend:v0.0.1
        env:
        - name: BACKEND_ENDPOINT
          value: http://backend
```

You can note that: 

1. `replicas: 3`: specifies that we need three replicas of the frontend microservice running at all times.
2. `template: ...`: specifies the pod template. It tells what properties the pod should have.
3. `containers: ...`: specifies the container running within the pod. It tells what image to use and what environment variables to set. Notice that we've set the `BACKEND_ENDPOINT` environment variable to `http://backend`, we'll see where this comes from in the Services section. The `image: frontend:v0.0.1` specifies the image to use for the container.

### Backend Microservice Deployment

The following is the deployment specification for the backend microservice:

`backend-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: backend:v0.0.1
        ports:
        - containerPort: 8080

```

You can note that:

1. `replicas: 3`: specifies that we need three replicas of the backend microservice running at all times.
2. `template: ...`: specifies the pod template. It tells what properties the pod should have. 
3. `containers: ...`: specifies the container running within the pod. It tells what image to use. The `image: backend:v0.0.1` specifies the image to use for the container.

To make the backend microservice reachable by an external client, we need to create a service for it. The following is the service specification for the backend microservice:

`backend-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

You can note that:

1. `selector: ...`: specifies that the service will only target the pods with the label `app: backend`. Any traffic to this service will be routed to the pods with this label. 
2. `ports: ...`: specifies which port on the service receives the traffic and which port on the pod should the traffic be forwarded to.

Now that we have the deployment and service specifications, let's see how everything comes together.

## Deploying everything

To deploy anything over the Kubernetes cluster, you would use the `kubectl` command-line tool. This is a special tool that helps you interact with the Kubernetes API Server. 

To deploy the resources, in this setup, you would head on to the **Master Node**, open the terminal, and run the following commands: 

```bash
kubectl apply -f frontend-deployment.yaml
```

```bash
kubectl apply -f backend-deployment.yaml
```

```bash
kubectl apply -f backend-service.yaml
```

Behind the scenes, the following happens (I've mentioned `etcd` and `API Server` a lot, just to highlight the importance of these components):

1. In `node-1`, `kubectl` serializes the YAML files into a JSON payload and sends it to the **API Server**. So, all the files, `frontend-deployment.yaml`, `backend-deployment.yaml`, and `backend-service.yaml` are sent to the API Server by `kubectl` after converting them to JSON.
2. The **API Server** in `node-1` receives the payload and processes it. It validates the specification and writes the data to the `etcd`. Each of our specifications for the frontend deployment, backend deployment, and backend service are stored in the `etcd`. *The API Server is the only component that directly interacts with the `etcd`*.
3. In `node-1`, the **Controller Manager**, specifically the `Deployment Controller`, sees the new deployment specifications in the `etcd` (via the API Server). Then it creates/updates the Replica Sets in `etcd` (via API Server) for the frontend and backend deployments, which defines how many replicas of the pods should be running at any given time. The `Replica Set Controller` (another controller) watches the Replica Sets sees that there are no pods running for the frontend and the backend deployments. It then creates the pod specifications in the `etcd` (via the API Server).
4. The **Scheduler** in `node-1` sees the new pod specifications in the `etcd`. It then assigns the pods to the worker nodes. In this case, let's say, it assignes two frontend pods to `node-2` and one to `node-3`. And one backend pod to `node-2` and two to `node-3`.
5. The **Kubelet** in `node-2` and `node-3` sees the new pod specifications in the `etcd` (via API Server). It then creates the containers for the pods. Running the containers is the job of the container runtime. The Kubelet interacts with the container runtime to create the containers.
5. The **Kube Proxy** in `node-2` and `node-3` sees the new service specification in the `etcd` (via API Server). It then configures the network rules in the node to route the traffic to the backend pods, which basically involves setting up the iptables rules.

And that's how everything comes together behind the scenes.

![](./assets/k8s_example_scenario_2.drawio.svg)
