## Troubleshooting Guide

Absolutely! When running **Minikube or Kubernetes locally**, you’ll inevitably run into **common challenges** with Pods, Deployments, Services, and ports. Here’s a **structured troubleshooting guide** with commands and tips — perfect for your **LFS250 hands-on session**.

---

## 1️⃣ General Kubernetes Troubleshooting Commands

| Problem                                  | Command                                                      | Explanation                                              |
| ---------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| See all Pods in the cluster              | `kubectl get pods --all-namespaces`                          | Checks if Pods are running, pending, or failed           |
| Describe a specific Pod                  | `kubectl describe pod <pod-name>`                            | Shows events, container state, and errors                |
| View logs of a Pod                       | `kubectl logs <pod-name>`                                    | Outputs logs of a container inside the Pod               |
| View logs of a specific container in Pod | `kubectl logs <pod-name> -c <container-name>`                | Useful for multi-container Pods                          |
| Check Deployments                        | `kubectl get deployments`                                    | Shows replicas, available pods, and rollout status       |
| Rollout status of Deployment             | `kubectl rollout status deployment/<deployment-name>`        | Confirms if deployment is successful                     |
| Debug Pod interactively                  | `kubectl exec -it <pod-name> -- /bin/sh`                     | Enter the Pod to inspect filesystem, network, processes  |
| Check Services                           | `kubectl get svc`                                            | See which ports are exposed and their NodePort/ClusterIP |
| Inspect network                          | `kubectl describe svc <service-name>`                        | Shows port mappings and selectors                        |
| View events in cluster                   | `kubectl get events --sort-by='.metadata.creationTimestamp'` | See recent failures or errors                            |

---

## 2️⃣ Port Troubleshooting (Common in Minikube)

### 🔹 Check free ports on your system

```bash
lsof -iTCP -sTCP:LISTEN -n -P
```

* Lists all ports currently in use.
* Choose **unused ports** for NodePort services (default range: 30000–32767).

---

### 🔹 NodePort allocation

* Kubernetes NodePort services use **ports 30000–32767** by default.
* You can specify a port manually in your Service YAML:

```yaml
ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30080
```

* **Make sure 30080 is free**. Otherwise, Kubernetes will fail to create the Service.

---

### 🔹 Minikube IP & Access

* Get Minikube node IP:

```bash
minikube ip
```

* Then access NodePort service:

```text
http://<minikube-ip>:<nodePort>
```

---

### 🔹 Troubleshoot Service not accessible

1. Check Pods:

```bash
kubectl get pods -l app=<label>
```

2. Check Service:

```bash
kubectl get svc <service-name>
kubectl describe svc <service-name>
```

3. Test connectivity inside cluster:

```bash
kubectl run -it --rm curlpod --image=curlimages/curl -- curl <service-name>:<port>
```

---

## 3️⃣ Common Issues & Fixes

| Issue                                 | Possible Cause                             | Fix                                                           |
| ------------------------------------- | ------------------------------------------ | ------------------------------------------------------------- |
| Pod stuck in `Pending`                | Not enough resources                       | Check `kubectl describe pod` events                           |
| Pod crashloop (`CrashLoopBackOff`)    | Container failing to start                 | Inspect logs with `kubectl logs`                              |
| Service not reachable                 | NodePort conflict or Pod selector mismatch | Ensure NodePort is free and labels match the Service selector |
| Minikube service doesn’t open browser | Browser not opening or firewall block      | Use `minikube service <svc-name> --url` to get direct URL     |
| Port already in use                   | NodePort overlapping with local process    | Pick another free NodePort in the 30000–32767 range           |

---

## 4️⃣ Extra Debug Tips

* **Watch resources live**:

```bash
kubectl get pods -l app=echoserver -w
```

* **Check all Minikube services and ports**:

```bash
kubectl get svc -A
```

* **Force delete stuck Pod**:

```bash
kubectl delete pod <pod-name> --grace-period=0 --force
```

* **Reset Minikube if cluster misbehaves**:

```bash
minikube delete
minikube start
```

---


**what to do if a port is already allocated** in Kubernetes/Minikube, and how to free it or choose a new one for your Service. Here’s a clear guide:

---

## 1️⃣ Check Which Ports Are Already in Use

On your **local system**, check all ports in use:

```bash
# Linux/macOS
lsof -iTCP -sTCP:LISTEN -n -P

# or using netstat
netstat -tuln
```

Look for the port you want to use (e.g., `30080` or `8080`) and note the process using it.

---

## 2️⃣ Free the Port

If a process is occupying the port:

```bash
# Kill process (replace <PID> with actual PID)
kill -9 <PID>
```

* On macOS/Linux, you can find PID from `lsof -iTCP:<port>`

```bash
lsof -iTCP:30080 -sTCP:LISTEN
```

* Then kill it:

```bash
kill -9 12345
```

> ⚠️ Be careful not to kill essential processes.

---

## 3️⃣ Change the NodePort in Kubernetes Service

If you **don’t want to kill anything**, just pick a **different free NodePort**:

```yaml id="nodeport-service"
apiVersion: v1
kind: Service
metadata:
  name: echoserver-service
spec:
  selector:
    app: echoserver
  type: NodePort
  ports:
    - protocol: TCP
      port: 8080       # Service port inside cluster
      targetPort: 8080 # Container port
      nodePort: 30081  # New free NodePort on host
```

Apply the Service:

```bash
kubectl apply -f echoserver-service.yaml
```

---

## 4️⃣ Remove a Service to Free Its Port

If a Service is already using a NodePort:

```bash
kubectl delete svc <service-name>
```

* This frees the allocated NodePort so you can reuse it.

Check NodePorts in use:

```bash
kubectl get svc
```

* NodePort column shows ports currently allocated.

---

## 5️⃣ Quick Minikube Tip

* Minikube automatically assigns a NodePort if you **don’t specify** `nodePort`.
* Example:

```yaml id="auto-nodeport"
type: NodePort
ports:
  - port: 8080
    targetPort: 8080
```

* Kubernetes picks a **free port in 30000–32767**, avoiding conflicts.

---

💡 **Demo Tip:**

* Always check `kubectl get svc` first before assigning a NodePort.
* Use `minikube service --url <svc-name>` to get a working URL without worrying about the port.


Your demo is solid already 👏 — you’re covering Docker + Nginx + basic container lifecycle very well.

Let me tighten it up and show you **what you *must verify before starting*** (this is what instructors and real DevOps engineers actually check first).

---

# ✅ 🔍 Pre-Flight Checklist (VERY IMPORTANT)

Before running anything, verify these:

## 1. Docker is installed and running

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

---

## 2. Minikube is installed (for Kubernetes part)

```bash
minikube version
```

---

## 3. Minikube cluster is running

```bash
minikube status
```

👉 You should see:

```
host: Running
kubelet: Running
apiserver: Running
```

If not:

```bash
minikube start
```

---

## 4. kubectl is configured

```bash
kubectl get nodes
```

👉 Expected:

```
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    control-plane
```

---

## 5. Check available ports (VERY IMPORTANT ⚠️)

Make sure port **8080 or 80 is free**

```bash
lsof -i :8080
```

If busy → change port:

```bash
docker run -d -p 9090:80 nginx:1.20
```

---

## 6. Internet connection (to pull images)

Test:

```bash
ping google.com
```

---

## 7. Disk space (your Mac is already tight 👀)

```bash
df -h
```

👉 If low:

```bash
docker system prune -a
```

(Be careful: deletes unused images)

---

## 🚀 Clean Demo Flow 

If you want to present like a **pro instructor**, use this:

```bash
# 1. Verify environment
docker --version
minikube status

# 2. Pull image
docker pull nginx:1.20

# 3. Confirm image
docker images | grep nginx

# 4. Run container
docker run -d -p 8080:80 nginx:1.20

# 5. Verify running container
docker ps

# 6. Test app
curl http://localhost:8080
```

---

# ⚠️ Fix Small Mistakes in Your Script

### ❌ This is wrong:

```bash
docker run --detach --publish-all nginx1.20
```

👉 Correct:

```bash
docker run -d -P nginx:1.20
```

---

### ❌ This is wrong:

```bash
docker ps ls
```

👉 Correct:

```bash
docker ps
```

---

### ❌ WordPress container (missing DB ⚠️)

```bash
docker run -d -p 80:80 tutum/wordpress
```

👉 This is outdated and incomplete.

Use:

```bash
docker run -d -p 8081:80 wordpress
```

---

# 🔥 Pro-Level Additions (to impress students)

### 1. Show logs

```bash
docker logs <container_id>
```

---

### 2. Exec into container

```bash
docker exec -it <container_id> bash
```

---

### 3. Remove container

```bash
docker rm -f <container_id>
```

---

### 4. Show container details

```bash
docker inspect <container_id>
```

---

# 🧠 Real DevOps Insight (Important)

What you're doing here teaches:

* Container lifecycle
* Image vs Container
* Port mapping
* Stateless services

BUT 👇

👉 This is **NOT Kubernetes yet**

This is just:

> Docker runtime level

Next step (important for LFS250):

* Run Nginx using **Kubernetes Pod**
* Use:

```bash
kubectl run nginx --image=nginx:1.20
```

---

