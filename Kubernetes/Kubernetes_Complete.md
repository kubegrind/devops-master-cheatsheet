# Kubernetes Complete Reference Guide

**Version:** 1.0  
**Last Updated:** October 2025  
**Audience:** DevOps Engineers & Kubernetes Administrators

---

## Table of Contents

1. [Kubernetes Basics & Architecture](#kubernetes-basics--architecture)
2. [kubectl Installation & Setup](#kubectl-installation--setup)
3. [Cluster Management](#cluster-management)
4. [Nodes Management](#nodes-management)
5. [Namespace Management](#namespace-management)
6. [Pod Management](#pod-management)
7. [Deployment Management](#deployment-management)
8. [Service Management](#service-management)
9. [ConfigMaps & Secrets](#configmaps--secrets)
10. [Persistent Volumes & Storage](#persistent-volumes--storage)
11. [StatefulSets](#statefulsets)
12. [DaemonSets & Jobs](#daemonsets--jobs)
13. [Ingress Management](#ingress-management)
14. [RBAC & Security](#rbac--security)
15. [Monitoring & Logging](#monitoring--logging)
16. [Networking](#networking)
17. [Helm Package Manager](#helm-package-manager)
18. [Common Workflows](#common-workflows)
19. [Troubleshooting](#troubleshooting)
20. [Best Practices](#best-practices)

---

## Kubernetes Basics & Architecture

### Kubernetes Components Overview

**Master/Control Plane Components:**
- **API Server**: Exposes Kubernetes API, manages cluster state
- **etcd**: Distributed key-value store for cluster data
- **Scheduler**: Assigns pods to nodes based on resources and constraints
- **Controller Manager**: Runs various controllers (deployment, statefulset, daemonset)
- **Cloud Controller Manager**: Manages cloud provider-specific controllers

**Worker Node Components:**
- **kubelet**: Agent running on each node, ensures containers run in pods
- **Container Runtime**: Docker, containerd, CRI-O
- **kube-proxy**: Handles networking and service discovery
- **Container Network Interface (CNI)**: Network plugin (Calico, Flannel, Weave)

### Kubernetes Objects

- **Pod**: Smallest deployable unit, can contain one or more containers
- **Service**: Exposes pods as network service
- **Deployment**: Manages replicated applications
- **StatefulSet**: Manages stateful applications with persistent identity
- **DaemonSet**: Ensures pod runs on every node
- **Job**: Runs to completion once
- **CronJob**: Scheduled jobs
- **ConfigMap**: Non-confidential configuration data
- **Secret**: Confidential data like passwords and tokens
- **PersistentVolume**: Storage resource
- **PersistentVolumeClaim**: Request for storage
- **Ingress**: Manages external HTTP/HTTPS access
- **NetworkPolicy**: Controls pod-to-pod communication

---

## kubectl Installation & Setup

### Install kubectl (macOS)

```bash
brew install kubectl
# or
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

Installs kubectl on macOS using Homebrew or direct download.

### Install kubectl (Linux)

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

Downloads and installs kubectl binary on Linux.

### Install kubectl (Windows)

```bash
choco install kubernetes-cli
# or
scoop install kubectl
```

Installs kubectl using package managers on Windows.

### Verify kubectl Installation

```bash
kubectl version
kubectl version --client
```

Confirms kubectl is installed and displays version.

### Configure kubectl Context

```bash
kubectl config view
kubectl config get-clusters
kubectl config get-contexts
```

Views current kubeconfig configuration and available clusters.

### Set Current Context

```bash
kubectl config use-context my-cluster
kubectl config set-context docker-desktop
```

Switches to specified cluster context.

### Add Cluster Credentials

```bash
kubectl config set-cluster my-cluster --server=https://cluster.example.com:6443
kubectl config set-credentials admin --client-certificate=/path/cert --client-key=/path/key
kubectl config set-context admin@my-cluster --cluster=my-cluster --user=admin
```

Manually configures cluster access credentials.

### Merge kubeconfig Files

```bash
export KUBECONFIG=~/.kube/config:~/.kube/cluster1:~/.kube/cluster2
kubectl config view --flatten > ~/.kube/config-merged
```

Combines multiple kubeconfig files for multi-cluster access.

### Create Namespace-Scoped Context

```bash
kubectl config set-context dev-ns --cluster=my-cluster --namespace=development
kubectl config use-context dev-ns
```

Creates context with default namespace scope.

### Enable Shell Autocompletion

```bash
# Bash
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc

# Zsh
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
source ~/.zshrc
```

Enables kubectl command autocompletion in shell.

### kubectl Alias

```bash
alias k=kubectl
source <(kubectl completion bash | sed 's/kubectl/k/g')
```

Creates shorthand alias for kubectl with autocomplete support.

---

## Cluster Management

### Get Cluster Information

```bash
kubectl cluster-info
kubectl cluster-info dump
```

Displays cluster endpoints and status information.

### Get Cluster Events

```bash
kubectl get events --all-namespaces
kubectl get events -n kube-system
```

Shows all cluster events for debugging and auditing.

### Check Cluster Health

```bash
kubectl get componentstatuses
kubectl get nodes -o wide
kubectl describe node
```

Verifies health of cluster components and nodes.

### Get API Resources

```bash
kubectl api-resources
kubectl api-resources --namespaced=true
```

Lists available API resources and their types.

### Get API Versions

```bash
kubectl api-versions
```

Shows all API groups and versions supported by cluster.

### Dry Run Commands

```bash
kubectl create deployment myapp --image=nginx --dry-run=client -o yaml
kubectl apply -f deployment.yaml --dry-run=client
```

Previews changes without applying them.

### Set Default Namespace

```bash
kubectl config set-context --current --namespace=production
```

Changes default namespace for all commands in current context.

### Get Cluster Metrics

```bash
kubectl top nodes
kubectl top pods --all-namespaces
kubectl top pods -n default
```

Displays CPU and memory usage across cluster.

### Describe Cluster

```bash
kubectl describe node master-node
kubectl get node -o json
```

Provides detailed information about cluster nodes.

### Export Cluster Configuration

```bash
kubectl get all --all-namespaces -o yaml > cluster-backup.yaml
```

Backs up entire cluster configuration.

### Access Kubernetes Dashboard

```bash
kubectl proxy
# Then visit http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

Starts proxy to access Kubernetes Dashboard web UI.

---

## Nodes Management

### List Nodes

```bash
kubectl get nodes
kubectl get nodes -o wide
kubectl get nodes --show-labels
```

Displays all cluster nodes with detailed information.

### Node Status

```bash
kubectl describe node node-name
kubectl get node node-name -o yaml
```

Shows comprehensive information about specific node.

### Add Node Label

```bash
kubectl label nodes node-name disktype=ssd
kubectl label nodes node-name tier=frontend
```

Assigns labels to nodes for pod scheduling.

### Remove Node Label

```bash
kubectl label nodes node-name disktype-
```

Removes label from node.

### Update Node Label

```bash
kubectl label nodes node-name disktype=hdd --overwrite
```

Modifies existing node label.

### Cordon Node

```bash
kubectl cordon node-name
```

Prevents new pods from being scheduled on node. Existing pods remain.

### Uncordon Node

```bash
kubectl uncordon node-name
```

Re-enables pod scheduling on previously cordoned node.

### Drain Node

```bash
kubectl drain node-name
kubectl drain node-name --ignore-daemonsets
kubectl drain node-name --delete-emptydir-data
```

Safely removes all pods from node before maintenance.

### Node Resource Requests

```bash
kubectl top node node-name
kubectl describe node node-name | grep -A 5 "Allocated resources"
```

Shows resource consumption and allocation on node.

### Taint Node

```bash
kubectl taint nodes node-name key=value:NoSchedule
kubectl taint nodes node-name key=value:NoExecute
```

Adds taint to prevent pod scheduling (NoSchedule) or evict existing pods (NoExecute).

### Remove Taint

```bash
kubectl taint nodes node-name key:NoSchedule-
```

Removes taint from node.

### Get Node Capacity

```bash
kubectl describe node node-name | grep -E "Capacity|Allocatable"
```

Shows total and allocatable resources on node.

### Check Node Ready Status

```bash
kubectl get nodes
# Look for "Ready" status
```

Verifies node is ready to accept pods.

### Node Logs

```bash
kubectl logs --tail=50 -f pod-name -c container-name
# On node itself
journalctl -u kubelet -n 50
```

Accesses kubelet logs on node.

---

## Namespace Management

### Create Namespace

```bash
kubectl create namespace production
kubectl create namespace staging
```

Creates new namespace for resource isolation.

### List Namespaces

```bash
kubectl get namespace
kubectl get ns
```

Shows all namespaces in cluster.

### Get Default Namespace

```bash
kubectl config view --minify | grep namespace
```

Displays current default namespace.

### Create Namespace Declaratively

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    environment: dev
```

Defines namespace in YAML for version control.

### Apply Namespace from File

```bash
kubectl apply -f namespace.yaml
```

Creates namespace from YAML configuration file.

### Get Namespace Details

```bash
kubectl describe namespace production
```

Shows detailed information about namespace.

### Set Namespace Quota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: production
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    pods: "100"
```

Limits resource consumption in namespace.

### Set Network Policy for Namespace

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

Restricts network traffic within namespace.

### Delete Namespace

```bash
kubectl delete namespace production
```

Removes namespace and all its resources.

### Get Namespace Resources

```bash
kubectl get all -n production
kubectl get pods,svc,deploy -n production
```

Lists all resources in specific namespace.

### Set Resource Limit for Namespace

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-memory-limit
  namespace: production
spec:
  limits:
  - max:
      cpu: "2"
      memory: "1Gi"
    min:
      cpu: "100m"
      memory: "128Mi"
    type: Pod
```

Defines resource limits for pods in namespace.

---

## Pod Management

### Create Pod from Image

```bash
kubectl run nginx --image=nginx:latest
kubectl run nginx --image=nginx:latest --port=80
```

Creates a pod from container image.

### Create Pod with Port Mapping

```bash
kubectl run myapp --image=myapp:1.0 --port=8080
```

Exposes container port on pod.

### Create Pod with Environment Variables

```bash
kubectl run myapp --image=myapp:1.0 \
  --env="DATABASE_URL=postgres://db:5432/mydb" \
  --env="NODE_ENV=production"
```

Sets environment variables for pod.

### Create Pod with Resource Limits

```bash
kubectl run myapp --image=myapp:1.0 \
  --limits=cpu=1,memory=256Mi \
  --requests=cpu=100m,memory=128Mi
```

Specifies resource requests and limits.

### List All Pods

```bash
kubectl get pods
kubectl get pods -o wide
kubectl get pods --all-namespaces
```

Shows all pods with various detail levels.

### List Pods with Labels

```bash
kubectl get pods --show-labels
kubectl get pods -L app,tier
```

Displays pod labels for organization.

### Get Pod Details

```bash
kubectl describe pod pod-name
kubectl get pod pod-name -o yaml
```

Shows comprehensive pod information.

### Get Pod Logs

```bash
kubectl logs pod-name
kubectl logs pod-name -c container-name
kubectl logs pod-name --previous
```

Displays pod logs from current or previous instance.

### Stream Pod Logs

```bash
kubectl logs -f pod-name
kubectl logs -f pod-name -c container-name
```

Continuously streams pod logs.

### Execute Command in Pod

```bash
kubectl exec -it pod-name -- /bin/bash
kubectl exec pod-name -- ls -la /app
```

Runs command inside running pod.

### Copy Files to/from Pod

```bash
kubectl cp pod-name:/app/file.txt ./local-file.txt
kubectl cp ./local-file.txt pod-name:/app/file.txt
```

Transfers files between host and pod.

### Port Forward to Pod

```bash
kubectl port-forward pod-name 8080:8080
kubectl port-forward pod-name 9000:8080
```

Forwards local port to pod for debugging.

### Delete Pod

```bash
kubectl delete pod pod-name
kubectl delete pod pod-name1 pod-name2 pod-name3
```

Removes one or more pods.

### Force Delete Pod

```bash
kubectl delete pod pod-name --grace-period=0 --force
```

Immediately terminates pod without graceful shutdown.

### Delete All Pods in Namespace

```bash
kubectl delete pods --all -n production
```

Removes all pods from specified namespace.

### Attach to Pod

```bash
kubectl attach pod-name -i -t
```

Attaches to pod's stdin/stdout for interaction.

### Describe Pod Events

```bash
kubectl describe pod pod-name | grep -A 10 "Events"
```

Shows events related to pod creation and status changes.

### Check Pod Readiness

```bash
kubectl get pod pod-name -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'
```

Verifies pod is ready to serve traffic.

### Wait for Pod Status

```bash
kubectl wait --for=condition=ready pod/pod-name --timeout=300s
```

Waits for pod to reach ready state or timeout.

---

## Deployment Management

### Create Deployment

```bash
kubectl create deployment myapp --image=myapp:1.0
kubectl create deployment myapp --image=myapp:1.0 --replicas=3
```

Creates deployment with specified image and replicas.

### Create Deployment Declaratively

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
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
      - name: myapp
        image: myapp:1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

Complete deployment configuration with health checks.

### Apply Deployment

```bash
kubectl apply -f deployment.yaml
```

Creates or updates deployment from file.

### List Deployments

```bash
kubectl get deployments
kubectl get deploy -o wide
```

Shows all deployments in current namespace.

### Get Deployment Status

```bash
kubectl describe deployment myapp
kubectl get deployment myapp -o yaml
```

Displays deployment details and status.

### Scale Deployment

```bash
kubectl scale deployment myapp --replicas=5
```

Changes number of replicas.

### Update Deployment Image

```bash
kubectl set image deployment/myapp myapp=myapp:2.0
kubectl set image deployment/myapp myapp=myapp:2.0 --record
```

Updates container image to new version with optional rollout history.

### Update Deployment Resource Limits

```bash
kubectl set resources deployment myapp -c=myapp --limits=cpu=500m,memory=512Mi --requests=cpu=200m,memory=256Mi
```

Changes resource requests and limits.

### Update Deployment Environment

```bash
kubectl set env deployment/myapp NODE_ENV=production
```

Updates environment variables in deployment.

### Rollout History

```bash
kubectl rollout history deployment/myapp
kubectl rollout history deployment/myapp --revision=2
```

Shows deployment update history with revision details.

### Rollback Deployment

```bash
kubectl rollout undo deployment/myapp
kubectl rollout undo deployment/myapp --to-revision=2
```

Reverts to previous or specific deployment revision.

### Pause Rollout

```bash
kubectl rollout pause deployment/myapp
```

Stops rolling update process.

### Resume Rollout

```bash
kubectl rollout resume deployment/myapp
```

Continues paused rolling update.

### Check Rollout Status

```bash
kubectl rollout status deployment/myapp
```

Monitors rolling update progress.

### Delete Deployment

```bash
kubectl delete deployment myapp
```

Removes deployment and associated pods.

### Set Deployment Strategy

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

Configures rolling update strategy with surge and unavailable pod limits.

### Rolling Update Configuration

```bash
kubectl patch deployment myapp -p '{"spec":{"minReadySeconds":10}}'
```

Updates deployment strategy parameters.

### Deployment with Init Containers

```yaml
spec:
  template:
    spec:
      initContainers:
      - name: init-db
        image: init:1.0
        command: ['sh', '-c', 'wait-for-db.sh']
      containers:
      - name: app
        image: myapp:1.0
```

Defines initialization containers that run before main container.

---

## Service Management

### Create Service

```bash
kubectl create service clusterip myapp --tcp=80:8080
kubectl expose deployment myapp --port=80 --target-port=8080 --type=ClusterIP
```

Creates service exposing deployment.

### Service Types

```yaml
# ClusterIP - Internal access only
apiVersion: v1
kind: Service
metadata:
  name: internal-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080

---
# NodePort - Access via node IP:port
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30000

---
# LoadBalancer - Cloud provider load balancer
apiVersion: v1
kind: Service
metadata:
  name: lb-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080

---
# ExternalName - External service access
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: ExternalName
  externalName: example.com
  ports:
  - port: 80
```

Different service types for various access patterns.

### List Services

```bash
kubectl get services
kubectl get svc -o wide
kubectl get endpoints
```

Shows all services and endpoint details.

### Get Service Details

```bash
kubectl describe service myapp
kubectl get service myapp -o yaml
```

Displays comprehensive service information.

### Create Service from Deployment

```bash
kubectl expose deployment myapp --port=80 --target-port=8080
```

Exposes deployment as service.

### Delete Service

```bash
kubectl delete service myapp
```

Removes service.

### Service Discovery

```bash
# Inside cluster, access service
curl http://service-name:port
curl http://service-name.namespace.svc.cluster.local:port
```

Accessing services within cluster.

### Port Forward to Service

```bash
kubectl port-forward svc/myapp 8080:80
```

Forwards local port to service.

### Endpoint Controller

```bash
kubectl get endpoints
kubectl get endpoints myapp -o yaml
```

Views endpoints automatically created for service.

### Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-headless
spec:
  clusterIP: None
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

Service without cluster IP for direct pod access.

### Session Affinity

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: ClusterIP
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

Ensures client requests route to same pod.

---

## ConfigMaps & Secrets

### Create ConfigMap from Literal

```bash
kubectl create configmap app-config --from-literal=key=value
kubectl create configmap app-config \
  --from-literal=NODE_ENV=production \
  --from-literal=LOG_LEVEL=debug
```

Creates ConfigMap from key-value pairs.

### Create ConfigMap from File

```bash
kubectl create configmap app-config --from-file=app.properties
kubectl create configmap app-config --from-file=config/
```

Loads configuration from file or directory.

### Create ConfigMap Declaratively

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  NODE_ENV: "production"
  LOG_LEVEL: "debug"
  app.properties: |
    key1=value1
    key2=value2
```

Defines ConfigMap in YAML.

### List ConfigMaps

```bash
kubectl get configmap
kubectl describe configmap app-config
```

Shows all ConfigMaps in namespace.

### Update ConfigMap

```bash
kubectl create configmap app-config --from-literal=key=newvalue --dry-run=client -o yaml | kubectl apply -f -
```

Updates ConfigMap configuration.

### Use ConfigMap in Deployment

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    envFrom:
    - configMapRef:
        name: app-config
    # Or specific keys
    env:
    - name: NODE_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: NODE_ENV
    # Or mount as file
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

Consumes ConfigMap in pod.

### Create Secret

```bash
kubectl create secret generic db-credentials --from-literal=username=admin --from-literal=password=secret123
```

Creates secret for sensitive data.

### Create Secret from File

```bash
kubectl create secret generic db-credentials --from-file=credentials.txt
```

Loads secret from file.

### Create Docker Registry Secret

```bash
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.azurecr.io \
  --docker-username=username \
  --docker-password=password \
  --docker-email=email@example.com
```

Creates credential for private image registry.

### Create TLS Secret

```bash
kubectl create secret tls tls-secret --cert=path/to/cert --key=path/to/key
```

Creates TLS certificate secret.

### List Secrets

```bash
kubectl get secrets
kubectl describe secret db-credentials
```

Shows all secrets in namespace.

### View Secret Data

```bash
kubectl get secret db-credentials -o yaml
# Decode base64
echo 'encoded-value' | base64 -d
```

Retrieves secret data (base64 encoded).

### Use Secret in Deployment

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    envFrom:
    - secretRef:
        name: db-credentials
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

Consumes secret in pod.

### Use Registry Secret

```yaml
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: myregistry.azurecr.io/myapp:1.0
```

Uses registry credentials for private images.

### Delete Secret

```bash
kubectl delete secret db-credentials
```

Removes secret.

### Sealed Secrets (Encryption at Rest)

```bash
# Install sealed-secrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Create sealed secret
kubectl create secret generic mysecret --from-literal=key=value --dry-run=client -o yaml | kubeseal -f - > mysealedsecret.yaml

# Apply sealed secret
kubectl apply -f mysealedsecret.yaml
```

Encrypts secrets in repository using sealed-secrets.

---

## Persistent Volumes & Storage

### Create PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /data/pv-storage
```

Defines storage resource in cluster.

### Create PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-storage
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 5Gi
```

Requests storage resource.

### List PersistentVolumes

```bash
kubectl get pv
kubectl describe pv pv-storage
```

Shows all storage volumes in cluster.

### List PersistentVolumeClaims

```bash
kubectl get pvc
kubectl get pvc -n production
```

Shows all storage claims in namespace.

### Use PVC in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-storage
```

Mounts PVC in pod.

### StorageClass Definition

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  size: 100Gi
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

Defines automatic volume provisioning.

### Dynamic Volume Provisioning

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 20Gi
```

Automatically provisions volume using StorageClass.

### Expand PVC

```bash
kubectl patch pvc pvc-storage -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'
```

Increases PVC size.

### Delete PVC

```bash
kubectl delete pvc pvc-storage
```

Removes PVC and associated volume.

### Local Storage

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 50Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  local:
    path: /mnt/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node-1
```

Defines local storage on specific node.

### NFS Volume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteMany
  nfs:
    server: nfs.example.com
    path: "/exports/data"
```

Uses NFS for shared storage.

### Snapshot Storage

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: pvc-snapshot
spec:
  volumeSnapshotClassName: csi-hostpath-snapclass
  source:
    persistentVolumeClaimName: pvc-storage
```

Creates storage snapshot for backup.

---

## StatefulSets

### Create StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql-headless
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

Manages stateful application with persistent identity.

### List StatefulSets

```bash
kubectl get statefulset
kubectl get sts -o wide
```

Shows all StatefulSets in namespace.

### Scale StatefulSet

```bash
kubectl scale statefulset mysql --replicas=5
```

Changes StatefulSet replicas.

### Delete StatefulSet

```bash
kubectl delete statefulset mysql
# Keep data
kubectl delete statefulset mysql --cascade=orphan
```

Removes StatefulSet, optionally keeping data.

### StatefulSet Pod Identity

```bash
# Pods get stable names: mysql-0, mysql-1, mysql-2
kubectl get pods -l app=mysql
```

Each pod maintains persistent identity.

### Headless Service for StatefulSet

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
```

Required for StatefulSet DNS discovery.

### UpdateStrategy for StatefulSet

```yaml
spec:
  updateStrategy:
    type: OnDelete  # Manual update
    # or
    type: RollingUpdate
    rollingUpdate:
      partition: 2  # Update only pods >= 2
```

Defines how StatefulSet updates are applied.

---

## DaemonSets & Jobs

### Create DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-monitor
spec:
  selector:
    matchLabels:
      app: node-monitor
  template:
    metadata:
      labels:
        app: node-monitor
    spec:
      containers:
      - name: monitor
        image: monitor:1.0
      tolerations:
      - effect: NoSchedule
        operator: Exists
```

Ensures pod runs on every node.

### Create Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-backup
spec:
  template:
    spec:
      containers:
      - name: backup
        image: backup:1.0
        command: ["./backup.sh"]
      restartPolicy: OnFailure
  backoffLimit: 3
```

Runs task to completion.

### Create CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup:1.0
            command: ["./backup.sh"]
          restartPolicy: OnFailure
```

Schedules job to run periodically.

### List DaemonSets

```bash
kubectl get daemonset
kubectl describe daemonset node-monitor
```

Shows all DaemonSets in namespace.

### List Jobs

```bash
kubectl get jobs
kubectl get jobs --all-namespaces
```

Shows all jobs in namespace.

### List CronJobs

```bash
kubectl get cronjobs
kubectl describe cronjob daily-backup
```

Shows all scheduled jobs.

### Get Job Logs

```bash
kubectl logs job/data-backup
kubectl logs -l job-name=data-backup
```

Views job execution logs.

### Delete Job

```bash
kubectl delete job data-backup
```

Removes completed job.

### Suspend CronJob

```bash
kubectl patch cronjob daily-backup -p '{"spec":{"suspend":true}}'
```

Pauses scheduled job execution.

### Resume CronJob

```bash
kubectl patch cronjob daily-backup -p '{"spec":{"suspend":false}}'
```

Resumes suspended scheduled job.

### Parallel Jobs

```yaml
spec:
  parallelism: 4
  completions: 10
  template:
    spec:
      containers:
      - name: worker
        image: worker:1.0
      restartPolicy: OnFailure
```

Runs multiple job instances in parallel.

---

## Ingress Management

### Create Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp
            port:
              number: 80
```

Exposes HTTP/HTTPS routes externally.

### List Ingresses

```bash
kubectl get ingress
kubectl describe ingress myapp-ingress
```

Shows all ingresses in namespace.

### Ingress with Multiple Routes

```yaml
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 3000
```

Routes different paths to different services.

### Ingress with Host Routing

```yaml
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        backend:
          service:
            name: api-service
            port:
              number: 8080
  - host: web.example.com
    http:
      paths:
      - path: /
        backend:
          service:
            name: web-service
            port:
              number: 3000
```

Routes different hosts to different services.

### Install Ingress Controller

```bash
# NGINX Ingress Controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace

# Or using kubectl
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/baremetal/deploy.yaml
```

Sets up ingress controller for routing rules.

### Get Ingress IP/Hostname

```bash
kubectl get ingress
kubectl get ingress -o jsonpath='{.items[0].status.loadBalancer.ingress[0]}'
```

Retrieves external IP or hostname for ingress.

### Delete Ingress

```bash
kubectl delete ingress myapp-ingress
```

Removes ingress routing rule.

### Ingress Annotation

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
```

Configures ingress behavior with annotations.

### SSL/TLS Configuration

```bash
# Create TLS secret
kubectl create secret tls myapp-tls \
  --cert=path/to/cert.crt \
  --key=path/to/key.key
```

Sets up HTTPS for ingress.

---

## RBAC & Security

### Create ServiceAccount

```bash
kubectl create serviceaccount myapp-sa -n production
```

Creates service account for pod authentication.

### Create Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/logs"]
  verbs: ["get"]
```

Defines namespace-scoped permissions.

### Create ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
```

Defines cluster-wide permissions.

### Create RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- kind: ServiceAccount
  name: myapp-sa
  namespace: production
```

Binds role to service account in namespace.

### Create ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes-all
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: node-reader
subjects:
- kind: ServiceAccount
  name: myapp-sa
  namespace: production
```

Binds cluster role to service account.

### List Roles

```bash
kubectl get roles
kubectl get roles -n production
```

Shows all roles in namespace.

### List ClusterRoles

```bash
kubectl get clusterroles
```

Shows all cluster-wide roles.

### Describe Role

```bash
kubectl describe role pod-reader -n production
```

Shows role permissions.

### Use ServiceAccount in Pod

```yaml
spec:
  serviceAccountName: myapp-sa
  containers:
  - name: app
    image: myapp:1.0
```

Assigns service account to pod.

### Pod Security Policy

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
  - ALL
  volumes:
  - 'configMap'
  - 'emptyDir'
  - 'projected'
  - 'secret'
  - 'downwardAPI'
  - 'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'MustRunAs'
  supplementalGroups:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
```

Enforces security constraints for pods.

### Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

Restricts network traffic between pods.

---

## Monitoring & Logging

### Install Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Enables CPU and memory metrics collection.

### View Cluster Metrics

```bash
kubectl top nodes
kubectl top nodes --sort-by=memory
```

Shows node resource usage.

### View Pod Metrics

```bash
kubectl top pods
kubectl top pods -n production
kubectl top pods --sort-by=memory
```

Shows pod resource consumption.

### Install Prometheus

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
```

Sets up Prometheus monitoring.

### Install Grafana

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana
```

Sets up Grafana dashboards.

### Install ELK Stack

```bash
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch
helm install kibana elastic/kibana
helm install beats elastic/filebeat
```

Sets up centralized logging.

### View Events

```bash
kubectl get events
kubectl get events --all-namespaces
kubectl get events -n production --sort-by='.lastTimestamp'
```

Shows cluster events.

### Tail Logs

```bash
kubectl logs -f deployment/myapp
kubectl logs -f pod/myapp-123 -c container-name
```

Streams logs in real-time.

### View Previous Logs

```bash
kubectl logs pod-name --previous
```

Accesses logs from crashed container.

### Export Logs

```bash
kubectl logs pod-name > pod-logs.txt
kubectl logs -l app=myapp > all-logs.txt
```

Saves logs to file.

### Debug Pod

```bash
kubectl debug pod-name -it --image=ubuntu
```

Creates debug pod for troubleshooting.

### Check Resource Usage Patterns

```bash
kubectl describe node node-name | grep -A 20 "Allocated resources"
```

Reviews resource allocation on node.

---

## Networking

### Check Network Policies

```bash
kubectl get networkpolicies
kubectl describe networkpolicy allow-frontend
```

Shows network traffic rules.

### Allow All Traffic

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - {}
  egress:
  - {}
```

Permits all pod-to-pod communication.

### Deny All Traffic

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

Blocks all pod-to-pod communication.

### Test Connectivity

```bash
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
# Inside container
wget -O- http://service-name:port
nc -zv service-name port
```

Tests pod-to-pod connectivity.

### DNS Resolution

```bash
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup service-name
```

Checks DNS resolution within cluster.

### Service DNS

```bash
# Short name
service-name
# Fully qualified
service-name.namespace.svc.cluster.local
# With port
service-name:port
```

Service naming conventions in cluster.

### Check kube-dns

```bash
kubectl get pods -n kube-system | grep dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

Verifies DNS service is running.

### IP Policies

```yaml
spec:
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.1.0/24
        except:
        - 192.168.1.10/32
```

Restricts traffic from IP ranges.

---

## Helm Package Manager

### Install Helm

```bash
brew install helm
# or
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Installs Helm package manager.

### Add Helm Repository

```bash
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

Adds chart repository and updates local index.

### Search Helm Charts

```bash
helm search repo nginx
helm search repo nginx --versions
```

Searches available charts in repositories.

### Install Chart

```bash
helm install myrelease nginx-stable/nginx
helm install myrelease nginx-stable/nginx -n production --create-namespace
```

Deploys chart to cluster.

### Install with Values

```bash
helm install myrelease nginx-stable/nginx -f values.yaml
helm install myrelease nginx-stable/nginx --set image.tag=1.20
```

Customizes chart installation.

### List Releases

```bash
helm list
helm list -n production
helm list --all-namespaces
```

Shows installed releases.

### Get Release Values

```bash
helm get values myrelease
helm get all myrelease
```

Displays release configuration.

### Upgrade Release

```bash
helm upgrade myrelease nginx-stable/nginx
helm upgrade myrelease nginx-stable/nginx -f values.yaml
```

Updates installed release.

### Rollback Release

```bash
helm rollback myrelease
helm rollback myrelease 1
```

Reverts to previous release version.

### Uninstall Release

```bash
helm uninstall myrelease
helm uninstall myrelease -n production
```

Removes installed release.

### Create Helm Chart

```bash
helm create mychart
cd mychart
# Edit Chart.yaml, values.yaml, templates/
```

Scaffolds new Helm chart structure.

### Package Chart

```bash
helm package mychart
```

Creates chart tarball for distribution.

### Template Chart

```bash
helm template myrelease ./mychart
helm template myrelease ./mychart -f values.yaml
```

Renders chart templates without installing.

### Helm Hooks

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
spec:
  template:
    spec:
      containers:
      - name: pre-install
        image: setup:1.0
      restartPolicy: Never
```

Runs scripts at helm lifecycle events.

### Helm Values

```yaml
# values.yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.20"
  pullPolicy: IfNotPresent
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

Default configuration values for charts.

---

## Common Workflows

### Deploy Application

```bash
# Create namespace
kubectl create namespace production

# Create ConfigMap
kubectl create configmap app-config -n production \
  --from-literal=LOG_LEVEL=info

# Create Secret
kubectl create secret generic db-credentials -n production \
  --from-literal=username=admin \
  --from-literal=password=secret

# Deploy application
kubectl apply -f deployment.yaml -n production

# Expose service
kubectl expose deployment myapp -n production --port=80 --target-port=8080 --type=ClusterIP

# Create ingress
kubectl apply -f ingress.yaml -n production
```

Complete application deployment workflow.

### Blue-Green Deployment

```bash
# Deploy blue version
kubectl set image deployment/myapp myapp=myapp:blue

# Deploy green version
kubectl create deployment myapp-green --image=myapp:green

# Test green version
kubectl port-forward svc/myapp-green 8080:8080

# Switch traffic to green
kubectl set image deployment/myapp myapp=myapp:green

# Keep blue for quick rollback
```

Zero-downtime deployment with easy rollback.

### Rolling Update

```bash
# Update image
kubectl set image deployment/myapp myapp=myapp:2.0

# Watch rollout
kubectl rollout status deployment/myapp

# Pause if needed
kubectl rollout pause deployment/myapp

# Resume or undo
kubectl rollout resume deployment/myapp
# or
kubectl rollout undo deployment/myapp
```

Gradual deployment update.

### Database Migration

```bash
# Create migration pod
kubectl run db-migrate --image=myapp:latest \
  -n production \
  --command -- ./migrate.sh

# Wait for completion
kubectl wait --for=condition=Ready pod/db-migrate -n production

# Deploy application
kubectl apply -f deployment.yaml -n production
```

Runs database migrations before app deployment.

### Scheduled Backup

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-db
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:1.0
            env:
            - name: BACKUP_DIR
              value: "/backups"
            volumeMounts:
            - name: backup-storage
              mountPath: /backups
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
```

Automatic scheduled backups.

### Multi-Tier Application

```yaml
# Database tier
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  # Database configuration

---
# Cache tier
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cache
spec:
  # Cache configuration

---
# Application tier
apiVersion: apps/v1
kind: Deployment
metadata:
  name: application
spec:
  # Application configuration
```

Deploying multi-tier microservices architecture.

---

## Troubleshooting

### Pod Not Starting

```bash
# Check pod status
kubectl describe pod pod-name

# Check pod events
kubectl get events --field-selector involvedObject.name=pod-name

# Check logs
kubectl logs pod-name
kubectl logs pod-name --previous

# Check node resources
kubectl describe node
```

Diagnosing pod startup failures.

### Insufficient Resources

```bash
# Check node resources
kubectl describe node | grep -A 10 "Allocated resources"

# Scale down other workloads
kubectl scale deployment myapp --replicas=1

# Add node to cluster
# Or increase node capacity
```

Resolving resource constraints.

### Crashlooping Pod

```bash
# Check logs
kubectl logs pod-name

# Check events
kubectl describe pod pod-name

# Increase init timeout
kubectl set env deployment/myapp STARTUP_PROBE_TIMEOUT=60

# Check health probe configuration
kubectl get pod pod-name -o yaml | grep -A 10 "probe"
```

Fixing continually restarting pods.

### Stuck Terminating Pod

```bash
# Force delete pod
kubectl delete pod pod-name --grace-period=0 --force

# Check for finalizers
kubectl get pod pod-name -o yaml | grep finalizers

# Remove finalizer if stuck
kubectl patch pod pod-name -p '{"metadata":{"finalizers":null}}'
```

Removing pods stuck in terminating state.

### Network Connectivity Issues

```bash
# Test DNS
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup service-name

# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://service:port

# Check service endpoints
kubectl get endpoints service-name

# Check network policies
kubectl get networkpolicies
```

Diagnosing network problems.

### PVC Not Bound

```bash
# Check PVC status
kubectl describe pvc pvc-name

# Check PV status
kubectl get pv

# Verify StorageClass exists
kubectl get storageclass

# Check for errors
kubectl get events | grep pvc-name
```

Fixing unbound persistent volume claims.

### Image Pull Errors

```bash
# Check image exists in registry
docker pull myregistry.azurecr.io/myapp:tag

# Verify credentials
kubectl get secrets
kubectl describe secret registry-secret

# Check pod status
kubectl describe pod pod-name | grep "Image"
```

Resolving image pull failures.

### High Resource Usage

```bash
# Check pod metrics
kubectl top pods --sort-by=memory

# Check node metrics
kubectl top nodes --sort-by=memory

# Identify culprit
kubectl describe pod high-memory-pod

# Set resource limits
kubectl set resources deployment/myapp --limits=memory=512Mi,cpu=500m
```

Finding and limiting resource-hungry pods.

### Debugging Application

```bash
# Port forward for direct testing
kubectl port-forward pod/myapp 8080:8080

# Execute shell in pod
kubectl exec -it pod-name -- /bin/bash

# Copy files for inspection
kubectl cp pod-name:/app/config.json ./local-config.json

# Check environment variables
kubectl exec pod-name -- env
```

Direct application debugging techniques.

### API Server Issues

```bash
# Check API server status
kubectl get componentstatuses

# Check kube-system namespace
kubectl get pods -n kube-system

# Restart API server if needed
kubectl delete pod -n kube-system kube-apiserver-master

# Check logs
kubectl logs -n kube-system kube-apiserver-master
```

Troubleshooting control plane issues.

### etcd Issues

```bash
# Check etcd health
kubectl -n kube-system exec etcd-master -- etcdctl endpoint health

# Backup etcd
kubectl -n kube-system exec etcd-master -- etcdctl snapshot save /tmp/etcd-backup.db

# Check etcd members
kubectl -n kube-system exec etcd-master -- etcdctl member list
```

Managing and troubleshooting etcd store.

---

## Best Practices

Use namespaces to organize and isolate workloads. Implement namespace quotas to prevent resource exhaustion.

Define resource requests and limits for all containers. Use Horizontal Pod Autoscaling for dynamic workload scaling.

Implement RBAC with least privilege principle. Use service accounts for pod authentication.

Use health checks (liveness, readiness, startup probes) to ensure reliability.

Separate configuration from code using ConfigMaps and Secrets.

Implement network policies to restrict traffic and improve security.

Use Helm charts for consistent, reusable deployments.

Monitor cluster health with Prometheus and display metrics in Grafana.

Centralize logging with ELK or similar solutions.

Implement backup and disaster recovery procedures.

Use GitOps tools (ArgoCD, Flux) for declarative infrastructure.

Regularly update Kubernetes and dependencies.

Implement Pod Disruption Budgets for high-availability applications.

Use persistent volumes appropriately for stateful workloads.

Version all configurations and keep them in version control.

---

## Resources

- Official Kubernetes Documentation: https://kubernetes.io/docs/
- kubectl Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- Kubernetes API Reference: https://kubernetes.io/docs/reference/
- Helm Documentation: https://helm.sh/docs/
- CNCF Kubernetes Training: https://www.cncf.io/training/
- Kubernetes Network Policies: https://kubernetes.io/docs/concepts/services-networking/network-policies/
- Kubernetes Security: https://kubernetes.io/docs/concepts/security/
- Kubernetes RBAC: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

