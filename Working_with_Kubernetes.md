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

You’ll see:
```
NAME              READY   STATUS    RESTARTS   AGE
nginx    1/1     Running   0          10s
```
   
```
kubectl describe pod nginx
```

If using EC2 instance 
```
Curl  IP address
```

## Expose Pod to Access in Browser

```
kubectl expose pod nginx --type=NodePort --port=80
kubectl get service nginx
```
Note the NodePort assigned (e.g., 31234)

✅ This will automatically:

Open a browser

```
Point to the correct URL (http://<minikube-ip>:<nodePort>)
Access in browser:
```

```
minikube service nginx-demo-pod
```
OR 
manually: http://<minikube-ip>:<NodePort>


----

# Find Minikube IP & NodePort Manually

## 1. Get Minikube IP:
   
```
minikube ip
```

```
Example output:

192.168.49.2
```
# 2. Get the service NodePort:

```
kubectl get service nginx-demo-pod
```

Output:

```
NAME              TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx-demo-pod    NodePort   10.96.187.102    <none>        80:31234/TCP   1m

```

31234 is the NodePort

```
Open browser:

http://192.168.49.2:31234
```
You’ll see your Nginx page.

## 3. Optional: Port Forward (Another Easy Way)

You can port-forward a Pod directly to localhost:

```
kubectl port-forward pod/nginx-demo-pod 8080:80
```

This forwards localhost:8080 → Pod’s port 80

```
Open in browser:

http://localhost:8080
```

✅ Advantages:

Works even without exposing NodePort

Perfect for quick demos on your laptop

----

## demo   3

Multiple Containers in a Pod (Sidecar Pattern)

Example: Multiple Containers in a Pod (Sidecar Pattern)

Multiple containers can run in a single Pod. One container is usually the main application, and additional containers provide supporting functionality (called sidecars).

vim pod-two-containers.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-sidecar
spec:
  containers:
    - name: nginx
      image: nginx:1.29
      ports:
        - containerPort: 80
    - name: count
      image: busybox:1.34
      args:
        - /bin/sh
        - -c
        - |
          i=0
          while true; do
            echo "$i: $(date)"
            i=$((i+1))
            sleep 1
          done
```
# 🚀 How to Test Locally (Minikube)

Apply the Pod:
```
kubectl apply -f myapp-pod.yaml
```

Check Pod status:

```
kubectl get pods
```
```
kubectl logs myapp-pod -c myapp-container
```


## demo   4

🔹 Using Init Containers

If you need some tasks to run before the main application starts, you can use initContainers.

They run sequentially and must complete successfully before the main container starts.

Common use case: waiting for a service to be available before starting your app.

vim init-myservice.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  initContainers:
    - name: init-myservice
      image: busybox
      command:
        - sh
        - -c
        - |
          until nslookup myservice; do
            echo waiting for myservice
            sleep 2
          done
  containers:
    - name: myapp-container
      image: busybox
      command:
        - sh
        - -c
        - echo "The app is running!" && sleep 3600
```

# 🚀 How to Test Locally (Minikube)

Apply the Pod:
```
kubectl apply -f myapp-pod.yaml
```

Check Pod status:

```
kubectl get pods
```

View logs
```
kubectl logs myapp-pod -c init-myservice
```

---




