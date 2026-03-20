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

## 3. Optional: Port Forward (Another Easy Way)✅ Best Option

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

more information about pod
```
kubectl describe pod nginx-with-sidecar
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

## 🚀 How to Test Locally (Minikube)

Apply the Pod:
```
kubectl apply -f init-myservice.yaml
```

Check Pod status:

```
kubectl get pods
```
more information about pod
```
kubectl describe pod myapp-pod
```

View logs
```
kubectl logs myapp-pod -c myapp-container
```
```
kubectl logs myapp-pod -c init-myservice
```


---

# demo   4 ( Workload Objects)

1. Pod Sample


vim pod.yaml

```
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

---
Apply the Pod:
```
kubectl apply -f pod.yaml
```

Check Pod status:

```
kubectl get pods
```

---

2. replicaset sample

vim replicaset.yaml

```
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
---

Apply the Pod:
```
kubectl apply -f replicaset.yaml
```
Verify Pod status replicated 

```
kubectl get pods
```

```
kubectl get pods -o wide
```

kubectl scale rs <replicaset-name> --replicas=<number-of-replicas>

```
kubectl scale rs nginx --replicas=5
```
or 
```
kubectl scale  --replicas=5 rs/nginx
```

verify 
```
kubectl get pods -o wide
```

3. deployment sample

vim replicaset.yaml

```
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
Apply the Pod:
```
kubectl apply -f replicaset.yaml
```

update image of your container using
```
kubectl set image deployment/nginx nginx=nginx:1.20
```

4. PersistentVolume (NFS)
vim PersistentVolume.yaml

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

> **Pod is temporary, but PV is permanent storage**

---

## 🔹 Explanation of This Example

* **20Gi storage** available
* **RWX** → multiple Pods/nodes can use it
* Uses **NFS server** at `192.168.1.10`
* Data is **retained even after release**

---

  * **PV = supply (storage provided)**
  * **PVC = demand (storage requested)**
    
* Show that:

  * Pod dies ❌ → data still exists ✅
* Use **Retain policy** for safe demos



# 🟢 Persistent Volumes (PV) – Summary

A **PersistentVolume (PV)** is a **cluster-wide storage resource** in Kubernetes.

* It is **provisioned by an admin** (static) or automatically via a **StorageClass** (dynamic).
* It **abstracts the underlying storage** (NFS, cloud disk, local disk, etc.).
* Its lifecycle is **independent of Pods** → data **persists even if Pods are deleted**.

---

## 🔹 Key Concepts

### ✅ Capacity

* Defines storage size
* Example: `10Gi`, `20Gi`

---

### ✅ Access Modes

* **ReadWriteOnce (RWO)** → One node can read/write
* **ReadOnlyMany (ROX)** → Many nodes read-only
* **ReadWriteMany (RWX)** → Many nodes read/write

---

### ✅ Volume Mode

* `Filesystem` → mounted as normal file system (default)
* `Block` → raw block device

---

### ✅ Reclaim Policy

What happens when a PVC releases the PV:

* **Retain** → keeps data (manual cleanup) ✅ (safe for demos)
* **Delete** → deletes storage automatically
* **Recycle** → deprecated

---

### ✅ Storage Backend

* Defines **where data lives**

  * NFS
  * AWS EBS / Azure Disk
  * iSCSI
  * Ceph

---

### ✅ Node Affinity

* Controls **which nodes can use the volume**
* Important for **cloud zones / local disks**

---







---

## Local Setup of kubeView Options

If you’re using Minikube locally, you can run KubeView either as:

Run as a Pod in your cluster

```
kubectl create namespace kubeview
```
```
kubectl apply -f https://raw.githubusercontent.com/benc-uk/kubeview/master/deploy/kubeview-deployment.yaml
```
Then port-forward to access locally:

```
kubectl port-forward -n kubeview svc/kubeview 9090:80
```

Open in browser:

http://localhost:9090

✅ You will see your Minikube Pods, ReplicaSets, Deployments visually.

---

