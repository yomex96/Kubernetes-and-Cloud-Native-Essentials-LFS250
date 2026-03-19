Kubernetes-and-Cloud-Native-Essentials-LFS250

---
# Running container - Demo


```bash
docker pull nginx

minikube start
```
---

```bash
docker run -d -p 8080:80 nginx1:20
```

---

# 🔥 Option 2 (If you want to SHOW the full process)

```bash
docker run nginx1:20
```
Docker will automatically:

1. Check local machine
2. Not find image
3. Pull from registry
4. Run container


> “This is how images are fetched from Docker Hub”

---

# 🧠 Bonus 

```bash
docker images
```

