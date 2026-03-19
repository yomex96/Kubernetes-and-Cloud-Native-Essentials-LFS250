# Networking and service

## Consider a production namespace with three Deployments:

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

## Hands-on of Deployment using Echoserver

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
kubectl get svc echoserver-service
```

