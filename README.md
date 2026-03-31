# Devops-repo
devops full course 


# Kubernetes: Zero to Advanced — The Professional DevOps Engineer's Guide

> *A battle-tested, production-grade learning path for mastering Kubernetes as a senior DevOps engineer would.*

---

## Table of Contents

1. [Foundation: What Is Kubernetes?](#1-foundation)
2. [Architecture Deep Dive](#2-architecture)
3. [Installing & Setting Up Clusters](#3-installation)
4. [Core Workload Objects](#4-core-workloads)
5. [Networking](#5-networking)
6. [Storage](#6-storage)
7. [Configuration & Secrets Management](#7-configuration)
8. [Security Hardening](#8-security)
9. [Observability: Logging, Metrics & Tracing](#9-observability)
10. [Helm: Package Management](#10-helm)
11. [CI/CD & GitOps](#11-cicd-gitops)
12. [Auto-Scaling & Resource Management](#12-scaling)
13. [Multi-Cluster & Federation](#13-multi-cluster)
14. [Disaster Recovery & Backups](#14-disaster-recovery)
15. [Service Mesh (Istio / Linkerd)](#15-service-mesh)
16. [Advanced Scheduling & Node Management](#16-advanced-scheduling)
17. [Operators & Custom Resources (CRDs)](#17-operators)
18. [Performance Tuning & Troubleshooting](#18-performance)
19. [Production Checklists](#19-production-checklists)
20. [Real-World Projects](#20-projects)

---

## 1. Foundation: What Is Kubernetes?

### Core Philosophy

Kubernetes (K8s) is a **container orchestration platform** that automates deploying, scaling, and managing containerized applications. It was originally designed by Google based on their internal system called Borg.

> **DevOps mindset**: Think of K8s not as a tool but as an *operating system for distributed systems*. Your job is to declare desired state — K8s reconciles reality to match it.

### Key Concepts

| Concept | Description |
|---|---|
| **Desired State** | You declare what you want (YAML manifests), K8s makes it happen |
| **Self-Healing** | K8s detects and automatically recovers from failures |
| **Declarative API** | Infrastructure as code, version controlled |
| **Horizontal Scaling** | Add/remove replicas automatically |
| **Service Discovery** | Pods find each other via DNS, not hardcoded IPs |

### When to Use Kubernetes

✅ Use K8s when you have:
- Microservices architecture (10+ services)
- Need for high availability & zero-downtime deployments
- Dynamic, autoscalable workloads
- Multi-team, multi-environment (dev/staging/prod) deployments

❌ Avoid K8s when:
- Running a single monolith application
- Small team with simple infrastructure
- Budget/ops complexity is a constraint

---

## 2. Architecture Deep Dive

```
┌─────────────────────────────────────────────────────┐
│                    CONTROL PLANE                     │
│  ┌──────────┐  ┌──────────┐  ┌────────────────────┐ │
│  │  API     │  │ etcd     │  │ Controller Manager │ │
│  │  Server  │  │ (state)  │  │ (reconcile loops)  │ │
│  └──────────┘  └──────────┘  └────────────────────┘ │
│  ┌──────────┐  ┌──────────┐                         │
│  │Scheduler │  │ Cloud    │                         │
│  │          │  │ Manager  │                         │
│  └──────────┘  └──────────┘                         │
└─────────────────────────────────────────────────────┘
          │
          │  (kubelet communicates up)
          │
┌─────────────────────────────────────────────────────┐
│                     WORKER NODES                     │
│  ┌──────────┐  ┌──────────┐  ┌────────────────────┐ │
│  │ kubelet  │  │ kube-    │  │ Container Runtime  │ │
│  │          │  │ proxy    │  │ (containerd/CRI-O) │ │
│  └──────────┘  └──────────┘  └────────────────────┘ │
│  ┌──────────────────────────────────────────────────┐│
│  │  Pod │ Pod │ Pod │ Pod │ Pod │ Pod │ Pod │ Pod   ││
│  └──────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────┘
```

### Control Plane Components

#### kube-apiserver
- The **single entry point** for all K8s management operations
- All `kubectl` commands hit the API server
- Stateless; horizontally scalable
- Validates and processes REST requests

#### etcd
- **Distributed key-value store** — the source of truth for all cluster state
- All cluster objects (Pods, Services, ConfigMaps) are stored here
- **Production rule**: Always run etcd with 3 or 5 nodes (odd number for quorum)
- Back up etcd daily: `etcdctl snapshot save backup.db`

#### kube-scheduler
- Watches for newly created pods with no assigned node
- Selects an optimal node based on: resource requests, affinity rules, taints/tolerations
- Scheduling algorithm: Filtering → Scoring → Binding

#### kube-controller-manager
- Runs reconciliation control loops:
  - **Node Controller**: monitors node health
  - **ReplicaSet Controller**: ensures desired pod count
  - **Deployment Controller**: manages rollouts
  - **Job Controller**: handles batch workloads

#### cloud-controller-manager
- Integrates with cloud provider APIs (AWS, GCP, Azure)
- Manages: LoadBalancers, Volumes, Node lifecycle

### Worker Node Components

#### kubelet
- The **node agent** — runs on every worker node
- Receives PodSpecs from API server and ensures containers are running
- Reports node and pod status back to API server
- Runs liveness/readiness probes

#### kube-proxy
- Maintains network rules (iptables/IPVS) on each node
- Implements the Service abstraction (routes traffic to correct pods)
- **Production tip**: Use IPVS mode for large clusters (better performance than iptables)

#### Container Runtime
- **containerd** (most common), CRI-O
- Implements the CRI (Container Runtime Interface)
- Pulls images, starts/stops containers

---

## 3. Installing & Setting Up Clusters

### Development Environments

#### minikube (local single-node)
```bash
# Install
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start cluster
minikube start --cpus=4 --memory=8192 --driver=docker

# Enable addons
minikube addons enable ingress
minikube addons enable metrics-server
minikube addons enable dashboard
```

#### kind (Kubernetes IN Docker)
```bash
# Install
go install sigs.k8s.io/kind@v0.22.0

# Multi-node cluster config
cat <<EOF > kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
EOF

kind create cluster --config kind-config.yaml --name dev-cluster
```

#### k3s (Lightweight Production)
```bash
# Single command install
curl -sfL https://get.k3s.io | sh -

# Join worker node
curl -sfL https://get.k3s.io | K3S_URL=https://MASTER_IP:6443 \
  K3S_TOKEN=NODE_TOKEN sh -
```

### Production Clusters

#### kubeadm (On-Premises)
```bash
# On all nodes: Install dependencies
apt-get update && apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
  gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Install kubeadm, kubelet, kubectl
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

# Initialize control plane
kubeadm init \
  --control-plane-endpoint="LOAD_BALANCER_DNS:6443" \
  --upload-certs \
  --pod-network-cidr=192.168.0.0/16

# Set up kubectl
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

# Install CNI (Calico)
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Join worker nodes
kubeadm join LOAD_BALANCER_DNS:6443 --token TOKEN \
  --discovery-token-ca-cert-hash sha256:HASH
```

#### Managed K8s (Recommended for production)

| Cloud | Service | Pros |
|---|---|---|
| AWS | EKS | Best IAM integration, Fargate support |
| Google | GKE | Most mature, Autopilot mode |
| Azure | AKS | Best for .NET workloads, Azure AD |
| DigitalOcean | DOKS | Cheapest for small teams |

```bash
# AWS EKS with eksctl
eksctl create cluster \
  --name production \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type m5.xlarge \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 10 \
  --managed
```

### kubectl Essentials

```bash
# Context management (critical for multi-cluster)
kubectl config get-contexts
kubectl config use-context production-cluster
kubectl config set-context --current --namespace=my-app

# Useful aliases (add to .bashrc/.zshrc)
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias ke='kubectl exec -it'
alias kaf='kubectl apply -f'

# Always check what you're working on
kubectl config current-context
kubectl get nodes
kubectl cluster-info
```

---

## 4. Core Workload Objects

### Pods

Pods are the **smallest deployable unit** in Kubernetes. A pod contains one or more containers that share network and storage.

```yaml
# pod.yaml — Understanding pod anatomy
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
    version: "1.0"
    env: production
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
spec:
  serviceAccountName: myapp-sa   # RBAC identity

  # Security context for the pod
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000

  containers:
    - name: myapp
      image: myregistry/myapp:1.0.0  # Always pin to digest in prod
      imagePullPolicy: IfNotPresent

      ports:
        - containerPort: 8080
          protocol: TCP

      # Resource limits — ALWAYS set in production
      resources:
        requests:
          memory: "128Mi"
          cpu: "100m"
        limits:
          memory: "512Mi"
          cpu: "500m"

      # Environment variables
      env:
        - name: ENV
          value: "production"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password

      # Health checks — critical for zero-downtime deployments
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
        initialDelaySeconds: 30
        periodSeconds: 10
        failureThreshold: 3

      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5

      startupProbe:
        httpGet:
          path: /healthz
          port: 8080
        failureThreshold: 30
        periodSeconds: 10

      # Volume mounts
      volumeMounts:
        - name: config
          mountPath: /etc/config
        - name: data
          mountPath: /data

  volumes:
    - name: config
      configMap:
        name: myapp-config
    - name: data
      persistentVolumeClaim:
        claimName: myapp-pvc

  # Graceful termination
  terminationGracePeriodSeconds: 60

  # Spread across availability zones
  topologySpreadConstraints:
    - maxSkew: 1
      topologyKey: topology.kubernetes.io/zone
      whenUnsatisfiable: DoNotSchedule
      labelSelector:
        matchLabels:
          app: myapp
```

### Deployments

Deployments manage **rolling updates and rollbacks** for stateless applications.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3

  selector:
    matchLabels:
      app: myapp

  # Rolling update strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Extra pods during update
      maxUnavailable: 0    # Never take pods down (zero-downtime)

  # Keep history for rollbacks
  revisionHistoryLimit: 10

  template:
    metadata:
      labels:
        app: myapp
        version: "1.0"
    spec:
      # Spread pods across nodes
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - myapp
                topologyKey: kubernetes.io/hostname
      containers:
        - name: myapp
          image: myregistry/myapp:1.0.0
          # ... (same as pod spec above)
```

```bash
# Deployment operations
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp
kubectl rollout undo deployment/myapp                    # Rollback to previous
kubectl rollout undo deployment/myapp --to-revision=3   # Rollback to specific version
kubectl set image deployment/myapp myapp=myapp:2.0.0    # Update image
kubectl scale deployment myapp --replicas=10            # Scale manually
```

### StatefulSets

For **stateful applications** (databases, message queues) that need stable network identities and persistent storage.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"   # Headless service name
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:15
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  # PVC template — each pod gets its own PVC
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: fast-ssd
        resources:
          requests:
            storage: 100Gi
```

> **Key difference from Deployment**: Pods get stable DNS names: `postgres-0.postgres`, `postgres-1.postgres`, `postgres-2.postgres`. Pods are created/deleted in order.

### DaemonSets

Run **one pod per node** — ideal for log collectors, monitoring agents, network plugins.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      tolerations:
        - key: node-role.kubernetes.io/control-plane
          effect: NoSchedule
      containers:
        - name: fluentd
          image: fluent/fluentd-kubernetes-daemonset:v1
          volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
        - name: varlibdockercontainers
          hostPath:
            path: /var/lib/docker/containers
```

### Jobs & CronJobs

```yaml
# One-time batch job
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 3
  template:
    spec:
      restartPolicy: OnFailure
      containers:
        - name: migrate
          image: myapp:latest
          command: ["python", "manage.py", "migrate"]

---
# Scheduled job
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"    # 2am daily
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: backup-tool:latest
              command: ["/scripts/backup.sh"]
```

---

## 5. Networking

### Services

Services provide **stable network endpoints** for pods (pods get new IPs when restarted).

```yaml
# ClusterIP — Internal only (default)
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080

---
# NodePort — External access via node IP
apiVersion: v1
kind: Service
metadata:
  name: myapp-nodeport
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080   # 30000-32767 range

---
# LoadBalancer — Cloud load balancer
apiVersion: v1
kind: Service
metadata:
  name: myapp-lb
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-internal: "false"
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - port: 443
      targetPort: 8080

---
# Headless Service — for StatefulSets, direct pod DNS
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
    - port: 5432
```

### Ingress

Ingress manages **HTTP/HTTPS routing** from outside the cluster.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/use-regex: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

```bash
# Install NGINX Ingress Controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.replicaCount=2 \
  --set controller.metrics.enabled=true

# Install cert-manager for automatic TLS
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true
```

### Network Policies

Control **pod-to-pod traffic** (default is allow-all — dangerous in production!).

```yaml
# Deny all ingress by default
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress

---
# Allow only specific traffic to backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
        - namespaceSelector:
            matchLabels:
              name: monitoring
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: postgres
      ports:
        - protocol: TCP
          port: 5432
    # Allow DNS
    - to: []
      ports:
        - protocol: UDP
          port: 53
```

### DNS in Kubernetes

```
# Pod DNS format:
pod-ip-address.namespace.pod.cluster.local

# Service DNS format:
service-name.namespace.svc.cluster.local

# Examples:
postgres.production.svc.cluster.local
myapp.default.svc.cluster.local

# Short forms (within same namespace):
postgres
myapp
```

### CNI Plugins Comparison

| CNI | Use Case | Features |
|---|---|---|
| **Calico** | Enterprise, production | NetworkPolicy, BGP, eBPF |
| **Flannel** | Simple setups | Overlay networking, easy setup |
| **Cilium** | High performance | eBPF-based, L7 policies, Hubble UI |
| **Weave** | Multi-cloud | Encryption, simple mesh |

---

## 6. Storage

### Storage Classes

```yaml
# AWS EBS StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

### PersistentVolumes & PersistentVolumeClaims

```yaml
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myapp-data
  namespace: production
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 50Gi
```

### Volume Types

| Type | Use Case |
|---|---|
| `emptyDir` | Temp data, cache (deleted with pod) |
| `hostPath` | Node-local data (DaemonSets) |
| `configMap/secret` | Config files, certificates |
| `persistentVolumeClaim` | Persistent app data |
| `nfs` | Shared storage across pods |
| `csi` | Cloud provider volumes |

---

## 7. Configuration & Secrets Management

### ConfigMaps

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  # Key-value pairs
  APP_ENV: "production"
  LOG_LEVEL: "info"

  # File content
  app.properties: |
    server.port=8080
    server.timeout=30s
    feature.dark_mode=true

  nginx.conf: |
    server {
        listen 80;
        location / {
            proxy_pass http://localhost:8080;
        }
    }
```

### Secrets

> ⚠️ **Production Warning**: Native K8s secrets are base64-encoded, not encrypted at rest. Use **Sealed Secrets**, **External Secrets Operator**, or **Vault** for production.

```yaml
# Create secret from literal
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=supersecret

# Create TLS secret
kubectl create secret tls myapp-tls \
  --cert=tls.crt \
  --key=tls.key
```

### External Secrets Operator (Production Pattern)

```yaml
# Store secrets in AWS Secrets Manager, sync to K8s
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secretsmanager
    kind: ClusterSecretStore
  target:
    name: db-secret
    creationPolicy: Owner
  data:
    - secretKey: password
      remoteRef:
        key: production/myapp/database
        property: password
```

### HashiCorp Vault Integration

```bash
# Install Vault Agent Sidecar
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault \
  --namespace vault \
  --create-namespace \
  --set server.ha.enabled=true \
  --set server.ha.replicas=3
```

---

## 8. Security Hardening

### RBAC (Role-Based Access Control)

```yaml
# ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
  namespace: production

---
# Role (namespace-scoped)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/logs"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: myapp-pod-reader
  namespace: production
subjects:
  - kind: ServiceAccount
    name: myapp-sa
    namespace: production
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

---
# ClusterRole — cluster-wide resources
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
```

### Pod Security Standards

```yaml
# Enforce restricted security for namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Security Context Best Practices

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault

  containers:
    - name: app
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
          add:
            - NET_BIND_SERVICE  # Only if binding port < 1024
```

### Image Security

```bash
# Use Trivy to scan images
trivy image myregistry/myapp:1.0.0

# Enforce image policy with OPA/Gatekeeper
# Only allow images from approved registries
```

```yaml
# ConstraintTemplate with OPA Gatekeeper
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: allowedrepos
spec:
  crd:
    spec:
      names:
        kind: AllowedRepos
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package allowedrepos
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not startswith(container.image, "myregistry.io/")
          msg := sprintf("Image %v not from approved registry", [container.image])
        }
```

---

## 9. Observability: Logging, Metrics & Tracing

### The Three Pillars

```
┌─────────────────────────────────────────────┐
│              OBSERVABILITY                   │
│                                             │
│  METRICS          LOGS           TRACES     │
│  (Prometheus)    (Loki/EFK)   (Jaeger)      │
│  "What's wrong"  "Why"         "Where"      │
└─────────────────────────────────────────────┘
```

### Prometheus + Grafana Stack

```bash
# Install kube-prometheus-stack (everything in one)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.enabled=true \
  --set prometheus.prometheusSpec.retention=30d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.storageClassName=fast-ssd \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=100Gi
```

```yaml
# ServiceMonitor — tell Prometheus to scrape your app
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
    - port: metrics
      path: /metrics
      interval: 30s
  namespaceSelector:
    matchNames:
      - production
```

```yaml
# PrometheusRule — alerting rules
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: myapp-alerts
spec:
  groups:
    - name: myapp
      rules:
        - alert: HighErrorRate
          expr: rate(http_requests_total{status="500"}[5m]) > 0.1
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "High error rate on {{ $labels.service }}"
            description: "Error rate is {{ $value }} req/s"

        - alert: PodRestarting
          expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
          for: 5m
          labels:
            severity: warning
```

### Logging with Loki + Promtail

```bash
helm install loki grafana/loki-stack \
  --namespace monitoring \
  --set loki.persistence.enabled=true \
  --set loki.persistence.size=50Gi \
  --set promtail.enabled=true
```

### EFK Stack (Elasticsearch + Fluentd + Kibana)

```bash
# Elasticsearch
helm install elasticsearch elastic/elasticsearch \
  --namespace logging \
  --create-namespace \
  --set replicas=3 \
  --set minimumMasterNodes=2

# Kibana
helm install kibana elastic/kibana --namespace logging

# Fluentd DaemonSet auto-collects all pod logs
kubectl apply -f https://raw.githubusercontent.com/fluent/fluentd-kubernetes-daemonset/master/fluentd-daemonset-elasticsearch-rbac.yaml
```

### Distributed Tracing with Jaeger

```bash
helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
helm install jaeger jaegertracing/jaeger \
  --namespace tracing \
  --create-namespace \
  --set collector.enabled=true \
  --set query.enabled=true
```

---

## 10. Helm: Package Management

### Helm Concepts

- **Chart**: A package of K8s manifests + templates
- **Release**: A deployed instance of a chart
- **Repository**: Chart registry (like npm/PyPI for K8s)
- **Values**: Configuration overrides

```bash
# Essential commands
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo bitnami/nginx
helm show values bitnami/nginx

helm install my-release bitnami/nginx \
  --namespace production \
  --create-namespace \
  --values my-values.yaml \
  --set service.type=LoadBalancer

helm list -A
helm upgrade my-release bitnami/nginx --values my-values.yaml
helm rollback my-release 1
helm uninstall my-release
```

### Creating Your Own Helm Chart

```bash
helm create myapp
# Creates structure:
# myapp/
# ├── Chart.yaml           # Chart metadata
# ├── values.yaml          # Default values
# ├── templates/
# │   ├── deployment.yaml
# │   ├── service.yaml
# │   ├── ingress.yaml
# │   ├── _helpers.tpl     # Reusable template helpers
# │   └── NOTES.txt
```

```yaml
# values.yaml
replicaCount: 3

image:
  repository: myregistry/myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent

resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70
```

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

---

## 11. CI/CD & GitOps

### GitOps Philosophy

> "The Git repository is the single source of truth for your infrastructure."

```
Developer → Git Push → CI Pipeline → Container Registry
                                            ↓
                                    GitOps Operator (ArgoCD/Flux)
                                            ↓
                                    Kubernetes Cluster
```

### ArgoCD

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get admin password
argocd admin initial-password -n argocd

# Port-forward UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

```yaml
# Application manifest
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp-gitops
    targetRevision: HEAD
    path: clusters/production/myapp

  destination:
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:
    automated:
      prune: true      # Delete resources removed from git
      selfHeal: true   # Auto-fix manual changes
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        maxDuration: 3m
```

### Flux CD

```bash
# Bootstrap Flux
flux bootstrap github \
  --owner=myorg \
  --repository=cluster-gitops \
  --branch=main \
  --path=./clusters/production \
  --personal
```

### GitHub Actions CI Pipeline

```yaml
# .github/workflows/deploy.yaml
name: Build and Deploy

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Log in to registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest

      - name: Run Trivy scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: "${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}"
          exit-code: "1"
          severity: CRITICAL

      - name: Update GitOps repo
        run: |
          git clone https://github.com/myorg/cluster-gitops
          cd cluster-gitops
          sed -i "s|image: .*|image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}|" \
            clusters/production/myapp/deployment.yaml
          git commit -am "Update image to ${{ github.sha }}"
          git push
```

---

## 12. Auto-Scaling & Resource Management

### Horizontal Pod Autoscaler (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 50

  metrics:
    # CPU-based
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70

    # Memory-based
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80

    # Custom metrics (from Prometheus via KEDA)
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "1000"

  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scaling down
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0    # Scale up immediately
      policies:
        - type: Pods
          value: 4
          periodSeconds: 15
```

### Vertical Pod Autoscaler (VPA)

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"  # Off | Initial | Recreate | Auto
  resourcePolicy:
    containerPolicies:
      - containerName: myapp
        minAllowed:
          cpu: 100m
          memory: 128Mi
        maxAllowed:
          cpu: 4
          memory: 4Gi
```

### Cluster Autoscaler

```bash
# AWS EKS with Cluster Autoscaler
helm install cluster-autoscaler autoscaler/cluster-autoscaler \
  --namespace kube-system \
  --set autoDiscovery.clusterName=production \
  --set awsRegion=us-east-1 \
  --set rbac.serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=arn:aws:iam::ACCOUNT:role/ClusterAutoscaler
```

### KEDA (Kubernetes Event-Driven Autoscaling)

```yaml
# Scale based on SQS queue depth
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: myapp-scaler
spec:
  scaleTargetRef:
    name: myapp
  minReplicaCount: 0    # Scale to zero!
  maxReplicaCount: 100
  triggers:
    - type: aws-sqs-queue
      metadata:
        queueURL: https://sqs.us-east-1.amazonaws.com/account/myqueue
        queueLength: "5"
        awsRegion: us-east-1
```

### Resource Quotas & LimitRanges

```yaml
# ResourceQuota per namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "20"
    requests.memory: "40Gi"
    limits.cpu: "40"
    limits.memory: "80Gi"
    pods: "100"
    services: "20"
    persistentvolumeclaims: "10"

---
# LimitRange — set defaults and min/max
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: production
spec:
  limits:
    - type: Container
      default:
        cpu: 200m
        memory: 256Mi
      defaultRequest:
        cpu: 100m
        memory: 128Mi
      max:
        cpu: "4"
        memory: 4Gi
      min:
        cpu: 50m
        memory: 64Mi
```

---

## 13. Multi-Cluster & Federation

### When to Use Multiple Clusters

- **Isolation**: Separate prod/staging/dev
- **Compliance**: Data sovereignty (EU data in EU clusters)
- **High Availability**: Active-active across regions
- **Scale**: Distribute load geographically

### kubectl Multi-Cluster Management

```bash
# Merge kubeconfigs
KUBECONFIG=~/.kube/cluster1:~/.kube/cluster2 kubectl config view --flatten > ~/.kube/config

# kubectx for easy switching
brew install kubectx
kubectx production   # Switch context
kubens my-namespace  # Switch namespace
```

### Cluster API (CAPI)

```yaml
# Declaratively manage clusters with K8s APIs
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: worker-cluster
spec:
  clusterNetwork:
    pods:
      cidrBlocks: ["192.168.0.0/16"]
  infrastructureRef:
    kind: AWSCluster
    name: worker-cluster
  controlPlaneRef:
    kind: KubeadmControlPlane
    name: worker-cluster
```

---

## 14. Disaster Recovery & Backups

### etcd Backup & Restore

```bash
# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%Y%m%d-%H%M%S).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify backup
ETCDCTL_API=3 etcdctl snapshot status backup.db --write-out=table

# Restore etcd
ETCDCTL_API=3 etcdctl snapshot restore backup.db \
  --data-dir=/var/lib/etcd-restored
```

### Velero (Application Backup)

```bash
# Install Velero with S3 backend
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.8.0 \
  --bucket my-velero-backups \
  --backup-location-config region=us-east-1 \
  --snapshot-location-config region=us-east-1 \
  --secret-file ./credentials-velero

# Create backup
velero backup create production-backup \
  --include-namespaces production \
  --storage-location default

# Schedule daily backups
velero schedule create daily-backup \
  --schedule="0 1 * * *" \
  --include-namespaces production \
  --ttl 720h0m0s   # Keep for 30 days

# Restore
velero restore create --from-backup production-backup
```

---

## 15. Service Mesh (Istio / Linkerd)

### Why Service Mesh?

- mTLS between all services (zero-trust networking)
- Traffic management (canary releases, A/B testing)
- Observability (traces, metrics) without code changes
- Circuit breaking, retries, timeouts

### Istio

```bash
# Install Istio
curl -L https://istio.io/downloadIstio | sh -
istioctl install --set profile=production -y

# Enable sidecar injection for namespace
kubectl label namespace production istio-injection=enabled
```

```yaml
# Traffic splitting — Canary deployment (10% new version)
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
    - myapp
  http:
    - route:
        - destination:
            host: myapp
            subset: stable
          weight: 90
        - destination:
            host: myapp
            subset: canary
          weight: 10

---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  subsets:
    - name: stable
      labels:
        version: stable
    - name: canary
      labels:
        version: canary
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
    outlierDetection:        # Circuit breaker
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 60s
```

---

## 16. Advanced Scheduling & Node Management

### Taints & Tolerations

```bash
# Taint a node (only specific pods can schedule here)
kubectl taint nodes gpu-node-1 hardware=gpu:NoSchedule
kubectl taint nodes spot-node-1 spot-instance=true:NoSchedule
```

```yaml
# Pod that tolerates the taint
spec:
  tolerations:
    - key: "hardware"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
  nodeSelector:
    hardware: gpu
```

### Node Affinity

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values:
                  - us-east-1a
                  - us-east-1b
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          preference:
            matchExpressions:
              - key: instance-type
                operator: In
                values:
                  - m5.2xlarge
```

### Priority Classes

```yaml
# High priority for critical workloads
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "Critical production workloads"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 1000
preemptionPolicy: Never
```

---

## 17. Operators & Custom Resources (CRDs)

### Custom Resource Definitions

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.mycompany.io
spec:
  group: mycompany.io
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                engine:
                  type: string
                  enum: [postgres, mysql, mongodb]
                version:
                  type: string
                replicas:
                  type: integer
                  minimum: 1
                  maximum: 5
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

### Building an Operator with Operator SDK

```bash
# Initialize operator project
operator-sdk init --domain mycompany.io --repo github.com/myorg/db-operator

# Create API
operator-sdk create api --group mycompany --version v1 --kind Database --resource --controller

# Implement reconcile logic in controllers/database_controller.go
# Build and deploy
make docker-build docker-push IMG=myregistry/db-operator:v1.0.0
make deploy IMG=myregistry/db-operator:v1.0.0
```

### Popular Operators to Know

- **Prometheus Operator**: Manages Prometheus + Alertmanager
- **PostgreSQL Operator (Zalando)**: Manages PostgreSQL clusters
- **Strimzi**: Apache Kafka on K8s
- **Cert-Manager**: TLS certificate automation
- **External Secrets Operator**: Secrets management

---

## 18. Performance Tuning & Troubleshooting

### Debugging Toolkit

```bash
# Pod debugging
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous              # Crashed container logs
kubectl logs <pod-name> -c <container-name>     # Multi-container pod
kubectl logs <pod-name> --tail=100 -f           # Follow logs

# Execute into a pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -c <container> -- sh

# Debug with ephemeral containers (K8s 1.23+)
kubectl debug -it <pod-name> --image=busybox --target=<container>

# Copy files
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file

# Port forwarding
kubectl port-forward pod/<pod-name> 8080:8080
kubectl port-forward svc/<service-name> 8080:80

# Node debugging
kubectl debug node/<node-name> -it --image=ubuntu

# Check events (often the first place to look)
kubectl get events --sort-by='.lastTimestamp' -n production
kubectl get events --field-selector type=Warning

# Resource usage
kubectl top pods -n production
kubectl top nodes
```

### Common Failure Patterns

| Symptom | Common Cause | Fix |
|---|---|---|
| `CrashLoopBackOff` | App crash on start | Check logs: `kubectl logs --previous` |
| `OOMKilled` | Memory limit too low | Increase memory limit or fix memory leak |
| `Pending` | No nodes can schedule | Check: affinity, taints, resources, PVCs |
| `ImagePullBackOff` | Can't pull image | Check registry access, image name/tag |
| `0/3 nodes available` | Resource exhaustion | Check node capacity, add nodes |
| Service unreachable | Wrong selector/port | Check service selector matches pod labels |

```bash
# Diagnose Pending pod
kubectl describe pod <pod-name> | grep -A 20 Events

# Check if service is routing correctly
kubectl get endpoints <service-name>

# Test DNS resolution
kubectl run debug --image=busybox --rm -it -- nslookup kubernetes

# Check network policies
kubectl get networkpolicies -n production
```

### Performance Optimization

```bash
# Use IPVS for kube-proxy (better than iptables at scale)
kubectl edit configmap kube-proxy -n kube-system
# Set: mode: "ipvs"

# Enable CPU Manager for CPU-intensive workloads
# In kubelet config: cpuManagerPolicy: static

# Use node local DNS cache
kubectl apply -f https://k8s.io/examples/admin/dns/nodelocaldns.yaml
```

---

## 19. Production Checklists

### Deployment Readiness Checklist

- [ ] Health checks configured (liveness, readiness, startup probes)
- [ ] Resource requests AND limits set
- [ ] Anti-affinity rules (spread pods across nodes/zones)
- [ ] PodDisruptionBudget configured
- [ ] Rolling update strategy with `maxUnavailable: 0`
- [ ] Image pinned to specific digest or immutable tag
- [ ] Secrets stored in Vault/External Secrets (not raw K8s secrets)
- [ ] NetworkPolicies in place (default deny)
- [ ] RBAC: service account with minimum permissions
- [ ] Security context: non-root, read-only filesystem, no privilege escalation
- [ ] Horizontal Pod Autoscaler configured
- [ ] Monitoring: ServiceMonitor + alerts defined
- [ ] Graceful termination: SIGTERM handler + terminationGracePeriodSeconds

### PodDisruptionBudget

```yaml
# Ensure at least 2 replicas always available during maintenance
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

### Cluster Production Checklist

- [ ] HA control plane (3 master nodes minimum)
- [ ] etcd backups automated (at minimum daily)
- [ ] Node OS auto-updates managed (not automatic)
- [ ] Cluster autoscaler configured
- [ ] Admission controllers: OPA Gatekeeper / Kyverno policies
- [ ] Audit logging enabled
- [ ] Image scanning in CI/CD pipeline
- [ ] Network policies: default deny per namespace
- [ ] Secrets encryption at rest enabled in etcd
- [ ] Certificates rotated automatically (cert-manager)
- [ ] Velero backups for application data

---

## 20. Real-World Projects

### Project 1: Deploy a Microservices App (Beginner)
Deploy a 3-tier app (frontend + backend + database) with:
- Deployments for stateless services
- StatefulSet for PostgreSQL
- Services + Ingress with TLS
- ConfigMaps and Secrets

### Project 2: GitOps Pipeline (Intermediate)
- GitHub Actions builds Docker image + pushes to registry
- ArgoCD auto-deploys on git push
- Separate repos for app code and K8s manifests
- Helm chart for the application

### Project 3: Production-Grade Platform (Advanced)
- Multi-cluster setup (staging + production)
- Full observability: Prometheus, Grafana, Loki, Jaeger
- Service mesh with Istio (mTLS + traffic management)
- KEDA autoscaling from message queues
- External Secrets from HashiCorp Vault
- OPA Gatekeeper policies
- Velero automated backups

---

## Essential Tools Ecosystem

| Category | Tool |
|---|---|
| **Package Management** | Helm, Kustomize |
| **GitOps** | ArgoCD, Flux CD |
| **Secrets** | HashiCorp Vault, External Secrets Operator |
| **Service Mesh** | Istio, Linkerd |
| **Observability** | Prometheus, Grafana, Loki, Jaeger |
| **Security** | OPA/Gatekeeper, Kyverno, Trivy, Falco |
| **Networking** | Calico, Cilium, MetalLB |
| **Autoscaling** | KEDA, Cluster Autoscaler, VPA |
| **Backup** | Velero, Kasten |
| **Multi-cluster** | Rancher, ArgoCD, Cluster API |
| **CLI Tools** | kubectx, k9s, Lens, Stern |

---

## Learning Path Summary

```
Week 1-2:   Architecture, kubectl, Pods, Deployments, Services
Week 3-4:   Storage, ConfigMaps, Secrets, RBAC, Namespaces
Week 5-6:   Ingress, Network Policies, Helm charts
Week 7-8:   Monitoring (Prometheus/Grafana), Logging (Loki)
Week 9-10:  CI/CD with ArgoCD, GitOps patterns
Week 11-12: Auto-scaling, Resource management, Production hardening
Week 13-16: Service Mesh, Operators, CRDs, Multi-cluster
Ongoing:    Build real projects, contribute to open source
```

---

*Built with 20 years of DevOps experience — by engineers who've run K8s in production at scale.*
