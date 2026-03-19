# Kubernetes-and-Cloud-Native-Essentials-LFS250

---

## Running container - Demo

```bash

minikube start
```

---

# 👀  Pull NGINX Image Locally

```bash
docker pull nginx:1.20
```

---

## ✅ 1. List all images

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

## ✅ 2. Filter only nginx

```bash
docker images | grep nginx
```

---

## ✅ 3. Inspect the image 

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

vim Dockerfile

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
vim index.html

```bash
<h1>Hello LFS250 🚀</h1>
<p>This is running inside a Docker container</p>
```







