
# 🌐 Kubernetes Notes

Kubernetes (a.k.a **K8s**) is a container orchestration platform used to manage, scale, and maintain containerized applications.

---

## ⚙️ Kubernetes Architecture

Kubernetes follows a **master-worker node architecture**.

### 🖥️ Master Node (Control Plane)
Responsible for managing the cluster.

- **kube-apiserver** → Acts as the entry point for all REST commands. Responsible for every communication in the cluster.
- **etcd** → A distributed key-value store that stores **all cluster data** (nodes, pods, configs, secrets, etc.).
- **kube-scheduler** → Assigns Pods to nodes based on resource availability and constraints.
- **Controllers** → Ensure the desired state of the system. Examples: Node Controller, Replication Controller.

### 💻 Worker Node
Responsible for running workloads.

- **kubelet** → An agent that runs on each node. It ensures containers are running in a Pod and reports node status back to the master.
- **Container Runtime** → The software that runs containers (e.g., Docker, containerd, CRI-O).
- **kube-proxy** → Handles networking on worker nodes, enabling service discovery and load balancing.

---

## 🔑 Basic Commands

```bash
kubectl run hello-minikube      # Run a container in the cluster
kubectl cluster-info            # Get cluster information
kubectl get nodes               # List all nodes in the cluster
kubectl edit pod <pod-name>     # Edit an existing pod manifest
```

---

## 🚀 Minikube

Minikube provides a **single-node cluster** for learning and testing Kubernetes locally.

```bash
minikube start
kubectl get nodes
```

---

## 📦 Pods in Kubernetes

A **Pod** is the smallest deployable unit in Kubernetes. It can run one or more tightly coupled containers.

### Example Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    type: front-end  
spec:
  containers:
    - name: nginx-container
      image: nginx
```

### Anatomy of a Pod Manifest
1. **apiVersion** → Version of Kubernetes API (v1 for Pods).
2. **kind** → Resource type (Pod, Deployment, Service, etc.).
3. **metadata** → Name, labels, annotations.
4. **spec** → Specification of the Pod (containers, volumes, etc.).

---

## 🌍 Exposing Ports in Pods

```yaml
containers:
  - name: web
    image: nginx
    ports:
      - containerPort: 80
```

---

## ⚡ Resource Requests & Limits

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "250m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

---

## 🔑 Environment Variables

```yaml
env:
  - name: DB_HOST
    value: mysql
  - name: DB_PASS
    value: secret123
```

---

## 📂 Volumes & Mounts

```yaml
volumes:
  - name: app-storage
    emptyDir: {}

containers:
  - name: app
    image: busybox
    volumeMounts:
      - mountPath: /data
        name: app-storage
```

---

## ❤️ Liveness & Readiness Probes

- **Liveness Probe** → Checks if the container is alive. If it fails, container is restarted.
- **Readiness Probe** → Checks if the container is ready to accept traffic.

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5

readinessProbe:
  tcpSocket:
    port: 3306
```

---

## 🧩 ReplicationController & ReplicaSet

- **ReplicationController (RC)** → Ensures a specified number of Pods are running at all times.
- **ReplicaSet (RS)** → The newer version of RC, with support for **selectors**.

### ReplicationController Example

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
spec:
  replicas: 3
  template:
    metadata:
      name: redis
    spec:
      containers:
      - name: redis
        image: redis
```

### ReplicaSet Example

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template:
    metadata:
      labels:
        type: front-end
    spec:
      containers:
      - name: redis
        image: redis
```

### 📌 Labels & Selectors

- **Labels** → Key-value pairs attached to objects (pods, services, etc.). Example: `type: front-end`.
- **Selectors** → Used to group and select Pods by labels. Example: A Service or ReplicaSet will select Pods with label `type=front-end`.

```bash
kubectl scale rs myapp-rs --replicas=6   # Scale pods
```

---

## 📦 Deployments

Deployments manage ReplicaSets and Pods. They support **rolling updates** and **rollbacks**.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx
```

Commands:
```bash
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment
```

---

## 🌐 Networking in Kubernetes

Every Pod gets:
- Its own IP address inside the cluster.
- Communication between Pods happens via these IPs.
- **Services** are used to expose Pods reliably (because Pod IPs can change).

### Types of Services

1. **ClusterIP (default)**  
   - Exposes service **within the cluster only**.
   - Gets a stable IP accessible to other Pods.

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: myapp-clusterip
   spec:
     type: ClusterIP
     ports:
       - port: 80
         targetPort: 80
     selector:
       app: myapp
   ```

2. **NodePort**  
   - Exposes service on a static port (`30000–32767`) on each node.  
   - Accessible externally via `<NodeIP>:NodePort`.

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: myapp-nodeport
   spec:
     type: NodePort
     ports:
       - port: 80
         targetPort: 80
         nodePort: 30008
     selector:
       app: myapp
   ```

3. **LoadBalancer**  
   - Used in cloud environments (AWS, GCP, Azure).  
   - Creates an external load balancer that routes to the Service.

---

## 🔗 About `--link`

The `--link` flag was used in **Docker** to connect containers together by sharing environment variables and updating `/etc/hosts`.  
⚠️ In **Kubernetes, you don’t need `--link`** because Pods and Services handle networking and service discovery.

---

# ✅ Summary

- **Master Node** → API server, etcd, scheduler, controllers.  
- **Worker Node** → kubelet, kube-proxy, container runtime.  
- **Pods** → Smallest deployable unit.  
- **ReplicationController/ReplicaSet** → Ensure availability.  
- **Deployment** → Manage Pods/ReplicaSets with rolling updates.  
- **Services** → Expose Pods (ClusterIP, NodePort, LoadBalancer).  
- **Networking** → Flat network, each Pod has its own IP.  
- **Probes, Volumes, Resources** → Enhance Pod reliability & persistence.

---

📌 With this, you have a **complete beginner-friendly Kubernetes guide** 🚀
