# Kubernetes-and-Cloud-Native-Essentials-LFS250


## Pre-Flight Checklist 

Before running anything, verify these:


### 1. Docker is installed and running

```bash
docker --version
```

👉 Expected:

```
Docker version XX.X.X
```

If not:

* Install Docker
* Start Docker daemon
* OR Open  Docker Application
---

### 2. Minikube is installed (for Kubernetes part)

```bash
minikube version
```

---

### 3. Minikube cluster is running

```bash
minikube status
```

### Running container - Demo

 Start Minikube

```
minikube start
```

Make sure your Minikube is running the Kubernetes cluster locally.

Check status:
```
minikube status
````

Expected output:

````
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
````

### Quick Verification 

Check Nodes with kubectl

````
kubectl get nodes
````

Expected:

````
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    control-plane   1m   v1.30.0
````

## Test a Pod

````
kubectl run nginx --image=nginx:1.20 --port=80

kubectl get pods

kubectl expose pod nginx --type=NodePort --port=80

minikube service nginx
````

### Optional: View Dashboard

````
minikube dashboard
````

This opens the Kubernetes dashboard in your browser for visual monitoring — very handy for demos.

---

### Clean up 

* To clean up your test Pod and Service in Minikube:

```bash
kubectl delete pod nginx && kubectl delete service nginx
```

* This removes the `nginx` Pod and its Service.
* 
* Confirm deletion with:

```bash
kubectl get pods
kubectl get svc
```

* To completely reset Minikube (wipe all resources):

```bash
minikube delete && minikube start
```

* This deletes all Pods, Services, and the cluster, letting you start fresh.

---

# Pull NGINX Image Locally (Alternatively  uisng docker pull)

```bash
docker pull nginx:1.20
```

---

## List all images

```bash
docker images
```

### Example output:

```
REPOSITORY   TAG     IMAGE ID       SIZE
nginx        1.20    abc123xyz      140MB
```

👉 This confirms:

* Image is downloaded
* Stored locally
* Ready to run

---

## 2. Filter only nginx

```bash
docker images | grep nginx
```

---

## 3. Inspect the image 

```bash
docker image inspect nginx:1.20
```

---


# 🚀 Now Run It 

```bash
docker run -d -p 8080:80 nginx:1.20
```
---
or 

```bash
docker run --detach --publish-all nginx1.20
```
---
or
```bash
docker run nginx:1.20
```
---
## 👀 View running containers

```bash
docker ps
```
## 👀 Then open / View on your browser

```bash
http://localhost:8080
```

---

# 💡 Pro Demo Flow (clean version)

```bash
docker pull nginx:1.20
docker images
docker run -d -p 8080:80 nginx:1.20
docker ps
```

---

## 🔥 Extra 

Show this:

```bash
docker history nginx:1.20
```
## 🔥 stop containner

```bash
docker stop <containerID>
```
## running wordpress site 

```bash
$ docker run -d -p 80:80 tutum/wordpress
```
```bash
curl http://localhost:80
```
# list  image

```bash
docker image ls
```
# image help 
```bash
docker image --help
```
---

## Building Container image - Demo

1.mkdir Docker_Demo

2.cd Docker_Demo

3.vim Dockerfile

```bash
# Base image
FROM nginx:1.20

# Set working directory
WORKDIR /usr/share/nginx/html

# Copy your custom website
COPY index.html .

# Expose port 80
EXPOSE 80

# Keep Nginx running in foreground
CMD ["nginx", "-g", "daemon off;"]
```
---

4.vim index.html

```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LFS250 Demo</title>
</head>
<body>
    <h1>Welcome to LFS250 🚀</h1>
    <p>This Nginx container is running your custom web page.</p>
    <p>Docker + Nginx = Hands-on Learning!</p>
</body>
</html>
```

---
# 🚀 Demo Flow

5.Build the image:

docker build -t nginx-custom:v1 .

list the build image

```bash
docker image ls
```

6.Run the container:

docker run -d -p 8080:80 nginx-custom:v1

```bash
docker container ls
```
or 
```bash
docker ps ls
```

7.Open in Chrome:

http://localhost:8080

To check ip address

view ip address
```
curl ifconfig.io
```







