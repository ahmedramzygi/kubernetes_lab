Kubernetes Lab Part 3: DaemonSets, Services, Static Pods, Networking
This lab will take you from DaemonSets to static pods with YAMLs, kubectl commands, and full explanations.
________________________________________
1. How many DaemonSets are created in the cluster in all namespaces?
▶️ Command:
kubectl get daemonsets --all-namespaces
✅ Explanation: This shows all DaemonSets running across the cluster.
________________________________________
2. What DaemonSets exist in the kube-system namespace?
▶️ Command:
kubectl get daemonsets -n kube-system
________________________________________
3. What is the image used by the pod deployed by the kube-proxy DaemonSet?
▶️ Command:
kubectl get daemonset kube-proxy -n kube-system -o jsonpath='{.spec.template.spec.containers[0].image}'
✅ Output: Will show something like: k8s.gcr.io/kube-proxy:v1.29.0
________________________________________
4. Deploy a DaemonSet for Fluentd Logging
📄 YAML File:
# fluentd-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: elasticsearch
  template:
    metadata:
      labels:
        name: elasticsearch
    spec:
      containers:
      - name: fluentd
        image: k8s.gcr.io/fluentd-elasticsearch:1.20
▶️ Command:
kubectl apply -f fluentd-daemonset.yaml
________________________________________
5. Deploy a Pod named nginx-pod using the nginx:alpine image with label tier=backend
📄 YAML File:
# nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    tier: backend
spec:
  containers:
  - name: nginx
    image: nginx:alpine
▶️ Command:
kubectl apply -f nginx-pod.yaml
________________________________________
6. Deploy a test pod using nginx:alpine
📄 YAML File:
# test-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    command: ["sleep"]
    args: ["3600"]
▶️ Command:
kubectl apply -f test-pod.yaml
________________________________________
7. Create a service backend-service to expose the backend pod on port 80
▶️ Command:
kubectl expose pod nginx-pod --port=80 --name=backend-service
________________________________________
8. Curl the backend-service from the test pod
▶️ Command:
kubectl exec -it test-pod -- curl backend-service
✅ Explanation: Should return the default NGINX welcome page HTML.
________________________________________
9. Create a deployment named web-app using the image nginx with 2 replicas
📄 YAML File:
# web-app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
▶️ Command:
kubectl apply -f web-app-deployment.yaml
________________________________________
10. Expose the web-app deployment as web-app-service on port 80, NodePort 30082
▶️ Command:
kubectl expose deployment web-app --name=web-app-service --port=80 --target-port=80 --type=NodePort
kubectl patch svc web-app-service -p '{"spec": {"ports": [{"port": 80, "targetPort": 80, "nodePort": 30082}]}}'
________________________________________
11. Access the web app from the node
▶️ Command:
curl http://<node-ip>:30082
✅ Expected: Default NGINX page
________________________________________
12. How many static pods exist in this cluster in all namespaces?
▶️ Command:
kubectl get pods --all-namespaces -o wide | grep static
📌 Or manually inspect:
sudo ls /etc/kubernetes/manifests/
✅ Explanation: Static pods are defined directly on node in the above folder.
________________________________________
13. On which nodes are the static pods currently created?
▶️ Command:
kubectl get pods -A -o wide | grep static
✅ Explanation: Column NODE will show which node they're on.
________________________________________
✅ This completes your advanced DaemonSet + Networking + Static Pod Lab.
Let me know when you're ready for:
•	ConfigMap/Secret Volumes
•	Ingress with domain routing
•	Health checks & Probes
•	StatefulSets
•	Persistent Volumes & Claims

1. How many ConfigMaps exist in the environment?
▶️ Command:
kubectl get configmaps --all-namespaces
________________________________________
2. Create a ConfigMap webapp-config-map
📄 YAML:
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config-map
  namespace: default
data:
  APP_COLOR: darkblue
▶️ Command:
kubectl apply -f webapp-config-map.yaml
________________________________________
3. Create webapp-color pod using the ConfigMap
📄 YAML:
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
  - name: nginx
    image: nginx
    envFrom:
    - configMapRef:
        name: webapp-config-map
▶️ Command:
kubectl apply -f webapp-color.yaml
________________________________________
4. How many Secrets exist?
▶️ Command:
kubectl get secrets --all-namespaces
________________________________________
5. How many secrets are defined in the default token?
▶️ Command:
kubectl describe secret $(kubectl get secret | grep default-token | awk '{print $1}')
________________________________________
6. Create a pod db-pod using MySQL 5.7
📄 YAML:
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  containers:
  - name: mysql
    image: mysql:5.7
▶️ Command:
kubectl apply -f db-pod.yaml
________________________________________
7. Why is db-pod not ready?
✅ Because MySQL requires environment variables such as MYSQL_ROOT_PASSWORD to be defined, otherwise it fails readiness checks.
________________________________________
8. Create secret db-secret
▶️ Command:
kubectl create secret generic db-secret \
  --from-literal=MYSQL_DATABASE=sql01 \
  --from-literal=MYSQL_USER=user1 \
  --from-literal=MYSQL_PASSWORD=password \
  --from-literal=MYSQL_ROOT_PASSWORD=password123
________________________________________
9. Recreate db-pod with secret envs
📄 YAML:
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  containers:
  - name: mysql
    image: mysql:5.7
    envFrom:
    - secretRef:
        name: db-secret
▶️ Command:
kubectl delete pod db-pod
kubectl apply -f db-pod.yaml
________________________________________
10. Create multi-container pod yellow
📄 YAML:
apiVersion: v1
kind: Pod
metadata:
  name: yellow
spec:
  containers:
  - name: lemon
    image: busybox
    command: ["sleep"]
    args: ["3600"]
  - name: gold
    image: redis
▶️ Command:
kubectl apply -f yellow.yaml
________________________________________
11. Create pod red with initContainer
📄 YAML:
apiVersion: v1
kind: Pod
metadata:
  name: red
spec:
  initContainers:
  - name: init-sleep
    image: busybox
    command: ['sh', '-c', 'sleep 20']
  containers:
  - name: redis
    image: redis
▶️ Command:
kubectl apply -f red.yaml
________________________________________
12. Create pod print-envars-greeting
📄 YAML:
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  containers:
  - name: print-env-container
    image: bash
    command: ["sh", "-c"]
    args: ["echo $GREETING $COMPANY $GROUP && sleep 10"]
    env:
    - name: GREETING
      value: "Welcome to"
    - name: COMPANY
      value: "DevOps"
    - name: GROUP
      value: "Industries"
▶️ Command:
kubectl apply -f print-envars.yaml
kubectl logs -f print-envars-greeting
________________________________________
13. Where is the default kubeconfig located?
📍 ~/.kube/config
________________________________________
14. How many clusters are defined in kubeconfig?
▶️ Command:
kubectl config view -o jsonpath='{.clusters[*].name}'
________________________________________
15. What is the user in the current context?
▶️ Command:
kubectl config view --minify -o jsonpath='{.users[0].name}'
________________________________________
16. Create a PersistentVolume pv-log
📄 YAML:
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/log
▶️ Command:
kubectl apply -f pv-log.yaml
________________________________________
17. Create PersistentVolumeClaim claim-log-1
📄 YAML:
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
▶️ Command:
kubectl apply -f pvc-claim-log.yaml
________________________________________
18. Create webapp pod using PVC
📄 YAML:
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/nginx
  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: claim-log-1
▶️ Command:
kubectl apply -f webapp.yaml

































