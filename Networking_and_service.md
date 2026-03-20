# Networking and Service section

## Implementation of  NetworkPolicy 

Consider a production namespace with three Deployments:

frontend Pods labeled app=frontend

backend Pods labeled app=backend

database Pods labeled app=db

We want to:

Restrict database access so that only backend Pods can communicate with it on TCP port 5432.

Allow backend Pods to reach an external payment API on TCP port 443.

Restrict backend ingress to only accept traffic from frontend Pods on port 8080.

vim NetworkPolicy.yaml

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-db-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: db
      ports:
        - protocol: TCP
          port: 5432
    - to:
        - ipBlock:
            cidr: 203.0.113.10/32
      ports:
        - protocol: TCP
          port: 443
```

## Hands-on of demo showing the use of service for  Deployment with Echoserver

1. vim echoserver-service.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echoserver
  labels:
    app: echoserver
spec:
  replicas: 3
  selector:
    matchLabels:
      app: echoserver
  template:
    metadata:
      labels:
        app: echoserver
    spec:
      containers:
        - name: echoserver
          image: k8s.gcr.io/echoserver:1.10
          ports:
            - containerPort: 8080
```

---

```
kubectl apply -f echoserver-service.yaml
```

```
 kubectl get pods -o wide
```

then curl if you using ec2 instance

```
curl ip address:8080
```
---

2. Expose the Deployment with a Service

Create a file echoserver-service.yaml

vim echoserver-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: echoserver-service
spec:
  selector:
    app: echoserver
  type: NodePort   # Exposes service on a port of Minikube node
  ports:
    - protocol: TCP
      port: 8080       # Cluster service port
      targetPort: 8080 # Container port
```

---
Apply the service:

```
kubectl apply -f echoserver-service.yaml
```

Check service:
```
kubectl get svc 
```
curl cluster IP:
```
watch curl <clusterIp:8080>
```




# Additional Demo

Students can scale the deployment:

```
kubectl scale deployment echoserver --replicas=5
kubectl get pods
```

You can also show Minikube dashboard:

```
minikube dashboard
```

Everything is local, no cloud cost, and perfect for hands-on sessions.


Google Cloud Platform microservices-demo
```
https://github.com/GoogleCloudPlatform/microservices-demo
```

---
## Access the Service in Browser

Minikube gives a simple command to open the service in your browser:
```
minikube service echoserver-service
```

This will open the service URL automatically.
```

Minikube maps the NodePort to a local port so you can access it like http://<minikube-ip>:<nodeport>.
```

Optional: Access from Inside Cluster

For debugging or demonstration:

```

kubectl run -it --rm curlpod --image=curlimages/curl -- curl echoserver-service:8080
```

This simulates another Pod calling your echoserver service.
