# Kubernetes Lab Part 2: Namespaces, Nodes, Affinity, and Resources

## 1. How many Namespaces exist?

```bash
kubectl get namespaces
```

✅ Expect: `default`, `kube-system`, `kube-public`, `kube-node-lease`

## 2. How many Pods exist in the kube-system namespace?

```bash
kubectl get pods -n kube-system
```

## 3. Create Deployment in namespace `finance`

📄 YAML File:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: beta
  namespace: finance
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis-app
  template:
    metadata:
      labels:
        app: redis-app
    spec:
      containers:
      - name: redis
        image: redis
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "1"
            memory: "2Gi"
```

▶️ Commands:

```bash
kubectl create namespace finance
kubectl apply -f beta-deployment.yaml
# OR imperative
kubectl create deployment beta --image=redis --replicas=2 -n finance
# NOTE: Use `kubectl edit` to add resource requests/limits manually in imperative flow
```

## 4. How many Nodes exist?

```bash
kubectl get nodes
```

## 5. Check Taints on master node

```bash
kubectl describe node <master-node-name>
```

✅ Look for:

```bash
Taints: node-role.kubernetes.io/master:NoSchedule
```

## 6. Label master node with color=blue

```bash
kubectl label node <master-node-name> color=blue
kubectl get nodes --show-labels
```

## 7. Create Deployment with Node Affinity to color=blue

📄 YAML File:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-blue
  template:
    metadata:
      labels:
        app: nginx-blue
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
      containers:
      - name: nginx
        image: nginx
```

▶️ Command:

```bash
kubectl apply -f blue-affinity-deployment.yaml
```

---

## 🔍 Bonus: Docker-like exec

```bash
kubectl get pods -n finance
kubectl exec -it <pod-name> -n finance -- /bin/bash
# or /bin/sh depending on image (e.g. redis, nginx use sh)

# Check resource requests for a running pod:
kubectl describe pod <pod-name> -n finance
```

---
