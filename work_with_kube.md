#  Working with Kubernetes – Hands-on Guide

This guide covers **Pods, Services, Persistent Storage, ConfigMaps, Secrets, Security, Scheduling, and Visual Tools** using Minikube. All examples are tested for local demos.

---

## 🛫 Pre-Flight Checklist

Before starting:

1. **Docker installed and running**

```bash
docker --version
```

Expected output:

```
Docker version XX.X.X
```

If not: install Docker, start the daemon, or open Docker Desktop.

2. **Minikube installed**

```bash
minikube version
```

3. **Minikube cluster running**

```bash
minikube status
```

---

## 1. Simple Pod Demo – `pod.yaml`

Create a Pod YAML:

```yaml
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

### ✅ Explanation

| Field               | Description                        |
| ------------------- | ---------------------------------- |
| apiVersion          | Core API version (`v1`)            |
| kind                | Resource type (`Pod`)              |
| metadata.name       | Name of the Pod                    |
| labels              | Useful for Services or Deployments |
| containers          | List of containers in the Pod      |
| image               | Docker image to use                |
| ports.containerPort | Port exposed by container          |

---

### 🔹 Commands

```bash
# Create Pod
kubectl create -f pod.yaml

# Apply changes
kubectl apply -f pod.yaml

# Get Pod status
kubectl get pod

# Delete Pod
kubectl delete pod nginx-demo-pod
```

```bash
# For more info
kubectl explain pod
```

---

## 2. Expose Pod to Access in Browser

```bash
kubectl expose pod nginx-demo-pod --type=NodePort --port=80
kubectl get svc
```

* Note the **NodePort** (e.g., `31234`).
* Minikube IP:

```bash
minikube ip
```

* Open in browser:

```
http://<minikube-ip>:<NodePort>
# Example: http://192.168.49.2:31234
```

### Optional: Port Forwarding (Quick Demo)

```bash
kubectl port-forward pod/nginx-demo-pod 8080:80
```

* Access: `http://localhost:8080`

✅ Advantage: works without NodePort, perfect for local demos.

---

## 3. Multiple Containers in a Pod (Sidecar Pattern)

`pod-two-containers.yaml`:

```yaml
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

```bash
# Apply Pod
kubectl apply -f pod-two-containers.yaml

# Check status
kubectl get pods

# More info
kubectl describe pod nginx-with-sidecar
```

---

## 4. Init Containers

Init containers run **before main containers**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
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

```bash
kubectl apply -f init-myservice.yaml
kubectl get pods
kubectl describe pod myapp-pod
kubectl logs myapp-pod -c myapp-container
kubectl logs myapp-pod -c init-myservice
```

---

## 5. Workload Objects

### Pod Sample

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx:1.19
      ports:
        - containerPort: 80
```

```bash
kubectl apply -f pod.yaml
kubectl get pods
```

### ReplicaSet Sample

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.19
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f replicaset.yaml
kubectl get pods
kubectl scale rs nginx --replicas=5
kubectl get pods -o wide
```

### Deployment Sample

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.19
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f deployment.yaml
kubectl set image deployment/nginx nginx=nginx:1.20
```

---

## 6. Persistent Volumes (PV) & Claims (PVC)

### PV Example

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs-storage
    server: 192.168.1.10
```

### PVC Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: ebs-gp2
```

### Pod Using PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app-container
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: app-storage
  volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: app-data-pvc
```

---

### StorageClass Example

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp2
provisioner: ebs.csi.aws.com
parameters:
  type: gp2
  fsType: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

---

## 7. ConfigMap Example (Nginx)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  nginx.conf: |
    user nginx;
    worker_processes 3;
    error_log /var/log/nginx/error.log;

    events {
      worker_connections 1024;
    }

    http {
      server {
        listen 80;
        server_name _;
        location / {
          root /usr/share/nginx/html;
          index index.html index.htm;
        }
      }
    }
```

### Pod Using ConfigMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.29
      ports:
        - containerPort: 80
      volumeMounts:
        - name: nginx-conf
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
  volumes:
    - name: nginx-conf
      configMap:
        name: nginx-conf
```

---

## 8. Secrets

* Store **sensitive data** (passwords, API keys).
* Base64 encoded by default, not encrypted.

**Best practice:** use external secret managers like HashiCorp Vault, AWS Secrets Manager, Google Secret Manager.

---

## 9 Kubernetes Security

* **RBAC**, **Pod Security**, **Admission Controllers**

### RoleBinding Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-pod-reader
  namespace: development
subjects:
  - kind: User
    name: alice
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRoleBinding Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-alice
subjects:
  - kind: User
    name: alice
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

### Pod Security Example

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: secure-ns
  labels:
    pod-security.kubernetes.io/enforce: restricted
```

---

## 10. Scheduling Objects

* **nodeSelector** → simple
* **Node Affinity** → advanced
* **nodeName** → manual
* **Topology Spread** → high availability
* **Taints & Tolerations** → control scheduling

### Example: NodeSelector

```yaml
nodeSelector:
  disktype: ssd
```

### Example: Node Affinity

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: topology.kubernetes.io/zone
              operator: In
              values:
                - us-east-1a
```

### Taints & Tolerations

```bash
kubectl taint nodes worker-1 region=useast2:NoSchedule
```

```yaml
tolerations:
  - key: "region"
    operator: "Equal"
    value: "useast2"
    effect: "NoSchedule"
```

---

## 11. Visualize Minikube Cluster – KubeView

```bash
kubectl create namespace kubeview
kubectl apply -f https://raw.githubusercontent.com/benc-uk/kubeview/master/deploy/kubeview-deployment.yaml
kubectl port-forward -n kubeview svc/kubeview 9090:80
```

* Open browser: `http://localhost:9090`
* Visualize Pods, ReplicaSets, and Deployments.

---
