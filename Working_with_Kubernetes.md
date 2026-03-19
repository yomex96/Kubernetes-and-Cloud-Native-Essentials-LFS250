# Working_with_Kubernetes

# A simple pod.yaml for your Nginx container
vim pod.yaml 

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo-pod
  labels:
    app: nginx-demo
spec:
  containers:
  - name: nginx-container
    image: nginx-demo:v1
    ports:
    - containerPort: 80
```
---

# ✅ Explanation

apiVersion: v1 → core API

kind: Pod → we are creating a Pod

metadata.name → the name of the Pod

labels → useful for Services or Deployments

containers → list of containers in the Pod

name → container name

image → the Docker image to use

ports → expose container port 80


---

```
kubectl create -f pod.yaml
```

```
kubectl apply -f pod.yaml
```
