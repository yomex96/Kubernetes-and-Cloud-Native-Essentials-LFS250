# Kubernetes-and-Cloud-Native-Essentials-LFS250


---

# Kubernetes-and-Cloud-Native-Essentials-LFS250

## 🛫 Pre-Flight Checklist

Before running anything, verify these:

### 1. Docker is installed and running

```bash
docker --version
```

**Expected output:**

```
Docker version XX.X.X
```

If not:

* Install Docker
* Start Docker daemon
* OR open Docker Desktop application

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

---

## 🐳 Running Container - Demo

Start Minikube:

```bash
minikube start
```

Check status:

```bash
minikube status
```

**Expected output:**

```
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

---

### ✅ Quick Verification

Check nodes with kubectl:

```bash
kubectl get nodes
```

**Expected output:**

```
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    control-plane   1m   v1.30.0
```

---

## 🧪 Test a Pod

```bash
kubectl run nginx --image=nginx:1.20 --port=80

kubectl get pods

kubectl expose pod nginx --type=NodePort --port=80

minikube service nginx
```

---

#  Manual Way

If you want to manually open it:

Get the service details:
```
kubectl get svc
```

Example output:
```
NAME    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx   NodePort   10.96.123.45   <none>        80:30001/TCP   2m

```
```
http://<minikube-ip>:<node-port>
```
Here 30001 is the NodePort.

Get Minikube IP:
```
minikube ip
```
Suppose it returns: 192.168.49.2

Open in browser:

```
http://192.168.49.2:30001
```


### Optional: View Dashboard

```bash
minikube dashboard
```

Opens the Kubernetes dashboard in your browser for visual monitoring — very handy for demos.

---

## 🧹 Clean Up

* Delete test Pod and Service:

```bash
kubectl delete pod nginx && kubectl delete service nginx
```

* Confirm deletion:

```bash
kubectl get pods
kubectl get svc
```

* Completely reset Minikube (wipe all resources):

```bash
minikube delete && minikube start
```

---

## 🏗 Pull NGINX Image Locally

```bash
docker pull nginx:1.20
```

### List all images

```bash
docker images
```

**Example output:**

```
REPOSITORY   TAG     IMAGE ID       SIZE
nginx        1.20    abc123xyz      140MB
```

**Confirm:** Image is downloaded, stored locally, and ready to run.

---

### Filter only nginx

```bash
docker images | grep nginx
```

### Inspect the image

```bash
docker image inspect nginx:1.20
```

---

## 🚀 Run NGINX Container

```bash
docker run -d -p 8080:80 nginx:1.20
```

Or alternatives:

```bash
docker run -d -P nginx:1.20
docker run nginx:1.20
```

### View running containers

```bash
docker ps
```

Open in browser:

```
http://localhost:8080
```

---

## 💡 Pro Demo Flow (clean version)

```bash
docker pull nginx:1.20
docker images
docker run -d -p 8080:80 nginx:1.20
docker ps
```

---

## 🔥 Extra Commands

* Show image history:

```bash
docker history nginx:1.20
```

* Stop container:

```bash
docker stop <containerID>
```

* Run WordPress:

```bash
docker run -d -p 80:80 tutum/wordpress
curl http://localhost:80
```

* List images:

```bash
docker image ls
```

* Docker image help:

```bash
docker image --help
```

---

## 🏗 Building a Custom Container Image - Demo

1. Create project directory:

```bash
mkdir Docker_Demo
cd Docker_Demo
```

2. Create Dockerfile:

```bash
vim Dockerfile
```

**Dockerfile content:**

```dockerfile
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

3. Create `index.html`:

```bash
vim index.html
```

**index.html content:**

```html
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

### 🚀 Demo Flow

1. Build the image:

```bash
docker build -t nginx-custom:v1 .
```

2. List the image:

```bash
docker image ls
```

3. Run the container:

```bash
docker run -d -p 8080:80 nginx-custom:v1
docker container ls
# or
docker ps
```

4. Open in browser:

```
http://localhost:8080
```

5. Check your IP address:

```bash
curl ifconfig.io
```









