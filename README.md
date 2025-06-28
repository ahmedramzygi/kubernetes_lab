# kubernetes_lab 
# Kubernetes Hands-On Lab (With Declarative YAML & Imperative Commands)

## Part 1: Pods, ReplicaSets, and Deployments

### 1. Create a Pod named `redis` using the image `redis`

**YAML (redis-pod.yaml):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
```

**Commands:**

```bash
kubectl apply -f redis-pod.yaml
kubectl run redis --image=redis --restart=Never
```

---

### 2. Create a Pod named `nginx` using an invalid image `nginx123`

**YAML (nginx-pod.yaml):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx123
```

**Commands:**

```bash
kubectl apply -f nginx-pod.yaml
kubectl run nginx --image=nginx123 --restart=Never
```

---

### 3. Check the nginx Pod Status

```bash
kubectl get pods
```

*Expected: ImagePullBackOff or ErrImagePull*

---

### 4. Fix the image and re-create the Pod

**YAML (nginx-fixed-pod.yaml):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

**Commands:**

```bash
kubectl delete pod nginx
kubectl apply -f nginx-fixed-pod.yaml
kubectl run nginx --image=nginx --restart=Never
```

---

### 5. Check how many ReplicaSets exist

```bash
kubectl get rs
```

*Expected: 0*

---

### 6. Create a ReplicaSet with 3 Pods using image busybox

**YAML (replicaset.yaml):**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-set-1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: busybox-app
  template:
    metadata:
      labels:
        app: busybox-app
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["sleep", "3600"]
```

**Command:**

```bash
kubectl apply -f replicaset.yaml
```

*Imperative `kubectl run` does not support ReplicaSet directly.*

---

### 7. Scale the ReplicaSet to 5 Pods

```bash
kubectl scale rs replica-set-1 --replicas=5
```

---

### 8. Check READY status in ReplicaSet

```bash
kubectl get rs replica-set-1
```

*Expected: READY 5/5*

---

### 9. Delete a Pod and observe ReplicaSet behavior

```bash
kubectl delete pod <pod-name>
kubectl get pods -l app=busybox-app
```

*Expected: Still 5 pods*

---

### 10. Check number of Deployments and ReplicaSets

```bash
kubectl get deploy
kubectl get rs
```

*Expected: 0 Deployments, 1 ReplicaSet*

---

### 11. Create a Deployment with busybox (3 replicas)

**YAML (deployment-1.yaml):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: busybox-deploy
  template:
    metadata:
      labels:
        app: busybox-deploy
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["sleep", "3600"]
```

**Commands:**

```bash
kubectl apply -f deployment-1.yaml
kubectl create deployment deployment-1 --image=busybox --replicas=3 --dry-run=client -o yaml > temp.yaml
# Then edit 'temp.yaml' to add sleep and apply
```

---

### 12. Check number of Deployments and ReplicaSets

```bash
kubectl get deploy
kubectl get rs
```

*Expected: 1 Deployment, 2 ReplicaSets (including replica-set-1)*

---

### 13. How many pods are ready for deployment-1?

```bash
kubectl get pods -l app=busybox-deploy
```

---

### 14. Update deployment-1 image to nginx

```bash
kubectl set image deployment deployment-1 busybox=nginx
kubectl rollout status deployment deployment-1
```

---

### 15. Check events and update strategy

```bash
kubectl describe deployment deployment-1
```

*Expected: Strategy is RollingUpdate*

---

### 16. Rollback deployment-1 to busybox

```bash
kubectl rollout undo deployment deployment-1
```

---

### 17. Create labeled Deployment using `nginx:latest`

**YAML (nginx-deployment.yaml):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

**Commands:**

```bash
kubectl apply -f nginx-deployment.yaml
kubectl create deployment nginx-deployment --image=nginx:latest --replicas=3 --dry-run=client -o yaml > nginx-deployment.yaml
# Then edit to add labels
```

---

*✅ The rest of the sections (Namespaces, Affinity, DaemonSets, Services, ConfigMaps, Secrets, StatefulSets, etc.) will follow the same format.*

Shall I continue with **Part 2: Namespaces, Nodes, and Affinity** in the same document?
