# Working with Kubernetes

## A simple pod.yaml for your Nginx container
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

then

---

1. create pod 
```
kubectl create -f pod.yaml
```

2. Apply pod
```
kubectl apply -f pod.yaml
```

3.Get pod
```
kubectl get pod
```

4.Delete pod
```
kubectl delete pod nginx-demo-pod
```

5. verify
   
```
kubectl get pods
```

6. Explanation
```
kubectl explain --help
```

---

## demo pod  2

```
docker run --detach nginx:1.19
```

```
docker ps
```

```
kubectl run nginx --image=nginx:1.19
```

   
```
kubectl get pods
```

   
```
kubectl describe pod nginx
```

   
```
Curl  IP address
```

You’ll see:
```
NAME              READY   STATUS    RESTARTS   AGE
nginx-demo-pod    1/1     Running   0          10s
```

# Expose Pod to Access in Browser

```
kubectl expose pod nginx-demo-pod --type=NodePort --port=80
kubectl get service nginx-demo-pod
```
Note the NodePort assigned (e.g., 31234)

Access in browser:

```
minikube service nginx-demo-pod
# OR manually: http://<minikube-ip>:<NodePort>
```
