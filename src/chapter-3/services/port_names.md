# Using Port Names

This feature is not talked about much, but it's a good feature nevertheless. You can use port names in the Service manifest. This decouples the port number from the Service configuration and provides a more human-readable way to access the Service. Here's is an example for how you'll use port names:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: simple-deployment
  template:
    metadata:
      labels:
        app: simple-deployment
    spec:
      containers:
      - name: apiserver
        image: rutush10/simple-restapi-server-py:v0.0.1
        ports:
        - containerPort: 8000
          name: apiserver-http
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 200m
            memory: 200Mi
---

apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: NodePort
  selector:
    app: simple-deployment
  ports:
    - protocol: TCP
      port: 8000
      targetPort: apiserver-http
      nodePort: 30001
      name: backend-http
```

The `---` is called the YAML separator. It's used to separate multiple documents in a single file. In this case, we have two documents: the Deployment and the Service. 

First, we define the port name `apiserver-http` in the Deployment. Then, we use this port name in the Service, specifying the `targetPort` as `apiserver-http`. This way, we can use the port name to access the Service. The benefit of this approach is that you can change the port number in the Deployment without changing the Service configuration. 

Also, you can name the Service port as well, here we've named it `backend-http`. Another resource, like a Pod, can access the Service using the port name `backend-http`, or another resource like an `Ingress` can use the port name to route the traffic to the Service.


## Summary

In this chapter, we learned about the different types of Services in Kubernetes. We started with the ClusterIP Service, which is used to route traffic within the cluster. Then, we moved on to the NodePort Service, which is used to expose the Service on a specific port on each node. Finally, we discussed the LoadBalancer Service, which is used to expose the Service outside the cluster.