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

