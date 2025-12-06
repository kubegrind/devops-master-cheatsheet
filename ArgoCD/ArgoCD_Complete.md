# Argo CD Complete Guide

## Table of Contents
- [Introduction](#introduction)
- [Installation](#installation)
- [Getting Started](#getting-started)
- [Applications](#applications)
- [Projects](#projects)
- [Repositories](#repositories)
- [Clusters](#clusters)
- [Sync Strategies](#sync-strategies)
- [ApplicationSet](#applicationset)
- [Helm Integration](#helm-integration)
- [Kustomize Integration](#kustomize-integration)
- [Multi-Tenancy](#multi-tenancy)
- [Security](#security)
- [SSO and RBAC](#sso-and-rbac)
- [Notifications](#notifications)
- [Image Updater](#image-updater)
- [Secrets Management](#secrets-management)
- [High Availability](#high-availability)
- [Monitoring and Metrics](#monitoring-and-metrics)
- [CLI Commands](#cli-commands)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Advanced Patterns](#advanced-patterns)
- [Practical Examples](#practical-examples)

---

## Introduction

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes. It follows the GitOps pattern of using Git repositories as the source of truth for defining the desired application state.

### Key Features

- **Declarative GitOps**: Git as single source of truth
- **Automated Deployment**: Continuous monitoring and sync
- **Multi-Cluster**: Manage multiple Kubernetes clusters
- **Multi-Tenancy**: Project-based access control
- **Rollback**: Easy rollback to any Git commit
- **Health Assessment**: Application health monitoring
- **Web UI & CLI**: Multiple interaction methods
- **SSO Integration**: OIDC, SAML, LDAP support
- **Webhook Support**: GitHub, GitLab, Bitbucket
- **PreSync/PostSync Hooks**: Lifecycle hooks

### Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Git Repository                     │
│              (Source of Truth)                       │
└────────────────────┬────────────────────────────────┘
                     │
                     │ Pull
                     ▼
┌─────────────────────────────────────────────────────┐
│                  Argo CD Server                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │
│  │   API    │  │   Web    │  │   Application    │ │
│  │  Server  │  │    UI    │  │   Controller     │ │
│  └──────────┘  └──────────┘  └──────────────────┘ │
└────────────────────┬────────────────────────────────┘
                     │
                     │ Apply
                     ▼
┌─────────────────────────────────────────────────────┐
│              Kubernetes Cluster(s)                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │   Pod    │  │  Service │  │  Ingress │         │
│  └──────────┘  └──────────┘  └──────────┘         │
└─────────────────────────────────────────────────────┘
```

---

## Installation

### Prerequisites

```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Verify Kubernetes cluster access
kubectl cluster-info
kubectl get nodes
```

### Install Argo CD (Standard)

```bash
# Create namespace
kubectl create namespace argocd

# Install Argo CD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=available --timeout=600s deployment/argocd-server -n argocd

# Get all resources
kubectl get all -n argocd
```

### Install Argo CD (HA)

```bash
# Install HA version
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/ha/install.yaml
```

### Install with Helm

```bash
# Add Helm repository
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Install Argo CD
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  --version 5.51.0

# Custom values
cat > argocd-values.yaml <<EOF
global:
  image:
    tag: "v2.9.0"

server:
  replicas: 3
  ingress:
    enabled: true
    hosts:
      - argocd.example.com
    annotations:
      kubernetes.io/ingress.class: nginx
      cert-manager.io/cluster-issuer: letsencrypt-prod
    tls:
      - secretName: argocd-tls
        hosts:
          - argocd.example.com

  config:
    url: https://argocd.example.com
    application.instanceLabelKey: argocd.argoproj.io/instance

redis:
  enabled: true

controller:
  replicas: 3

repoServer:
  replicas: 3

applicationSet:
  enabled: true

notifications:
  enabled: true
EOF

# Install with custom values
helm install argocd argo/argo-cd \
  -n argocd \
  --create-namespace \
  -f argocd-values.yaml

# Upgrade
helm upgrade argocd argo/argo-cd \
  -n argocd \
  -f argocd-values.yaml
```

### Access Argo CD UI

```bash
# Port forwarding (development)
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Access at: https://localhost:8080

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo

# Change admin password
argocd account update-password

# Expose via LoadBalancer
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Expose via Ingress
cat > argocd-ingress.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  rules:
    - host: argocd.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  number: 443
  tls:
    - hosts:
        - argocd.example.com
      secretName: argocd-tls
EOF

kubectl apply -f argocd-ingress.yaml
```

### Install Argo CD CLI

```bash
# Linux
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/

# macOS
brew install argocd

# Windows (using Chocolatey)
choco install argocd-cli

# Verify installation
argocd version

# Login to Argo CD
argocd login argocd.example.com
argocd login localhost:8080 --insecure

# Login with token
argocd login argocd.example.com --username admin --password PASSWORD

# Context management
argocd context
argocd context argocd.example.com
```

---

## Getting Started

### First Application (CLI)

```bash
# Create application from Git repository
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Sync application
argocd app sync guestbook

# Get application status
argocd app get guestbook

# Watch application sync
argocd app sync guestbook --watch

# List applications
argocd app list

# Delete application
argocd app delete guestbook
```

### First Application (YAML)

```yaml
# guestbook-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - Validate=true
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

```bash
# Apply application
kubectl apply -f guestbook-app.yaml

# Or using argocd CLI
argocd app create -f guestbook-app.yaml
```

---

## Applications

### Application Manifest Structure

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
  # Finalizer that ensures cascaded deletion
  finalizers:
    - resources-finalizer.argocd.argoproj.io
  # Labels
  labels:
    environment: production
    team: platform
  # Annotations
  annotations:
    notifications.argoproj.io/subscribe.on-sync-succeeded.slack: my-channel
spec:
  # Project name
  project: my-project

  # Source repository
  source:
    repoURL: https://github.com/org/repo.git
    targetRevision: main
    path: k8s/overlays/production

    # Helm specific
    helm:
      releaseName: myapp
      valueFiles:
        - values.yaml
        - values-production.yaml
      parameters:
        - name: image.tag
          value: v1.0.0
      values: |
        replicaCount: 3
        service:
          type: LoadBalancer

    # Kustomize specific
    kustomize:
      namePrefix: prod-
      nameSuffix: -v1
      images:
        - myapp=myapp:v1.0.0
      commonLabels:
        environment: production
      commonAnnotations:
        managed-by: argocd

    # Directory (plain YAML)
    directory:
      recurse: true
      jsonnet:
        extVars:
          - name: environment
            value: production

  # Destination cluster and namespace
  destination:
    server: https://kubernetes.default.svc
    namespace: production

  # Sync policy
  syncPolicy:
    automated:
      prune: true        # Delete resources not in Git
      selfHeal: true     # Sync when cluster state changes
      allowEmpty: false  # Don't delete all resources
    syncOptions:
      - Validate=true
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
      - ApplyOutOfSyncOnly=true
      - RespectIgnoreDifferences=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

  # Ignore differences
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
    - group: apps
      kind: StatefulSet
      jsonPointers:
        - /spec/volumeClaimTemplates

  # Resource tracking
  revisionHistoryLimit: 10

  # Info
  info:
    - name: 'Project'
      value: 'My Application'
    - name: 'Owner'
      value: 'platform-team@example.com'
```

### Application Operations

```bash
# Create application
argocd app create myapp \
  --repo https://github.com/org/repo.git \
  --path k8s/overlays/production \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production \
  --sync-policy automated \
  --auto-prune \
  --self-heal

# Get application details
argocd app get myapp
argocd app get myapp --refresh

# List applications
argocd app list
argocd app list -o wide
argocd app list -o json

# Sync application
argocd app sync myapp
argocd app sync myapp --force
argocd app sync myapp --dry-run
argocd app sync myapp --prune
argocd app sync myapp --resource Deployment:myapp

# Check diff
argocd app diff myapp

# Rollback application
argocd app rollback myapp
argocd app rollback myapp <history-id>

# Set application parameters
argocd app set myapp --repo https://github.com/org/new-repo.git
argocd app set myapp --path k8s/new-path
argocd app set myapp --revision v2.0.0
argocd app set myapp --sync-policy automated

# Patch application
argocd app patch myapp --patch '{"spec":{"source":{"targetRevision":"v2.0.0"}}}'

# Unset auto-sync
argocd app unset myapp --sync-policy

# Terminate operation
argocd app terminate-op myapp

# Delete application
argocd app delete myapp
argocd app delete myapp --cascade  # Delete app and resources
argocd app delete myapp --cascade=false  # Delete only app

# Wait for sync
argocd app wait myapp
argocd app wait myapp --timeout 300

# Manifest generation
argocd app manifests myapp
argocd app manifests myapp --source live
```

### Resource Hooks

```yaml
# PreSync hook
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
        - name: migration
          image: myapp:migrations
          command: ["./migrate.sh"]
      restartPolicy: Never

---
# PostSync hook
apiVersion: batch/v1
kind: Job
metadata:
  name: cache-warmup
  annotations:
    argocd.argoproj.io/hook: PostSync
    argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
spec:
  template:
    spec:
      containers:
        - name: warmup
          image: myapp:latest
          command: ["./warmup.sh"]
      restartPolicy: Never

---
# SyncFail hook
apiVersion: batch/v1
kind: Job
metadata:
  name: rollback-notification
  annotations:
    argocd.argoproj.io/hook: SyncFail
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
        - name: notify
          image: curlimages/curl
          command:
            - sh
            - -c
            - 'curl -X POST https://hooks.slack.com/services/XXX'
      restartPolicy: Never
```

### Hook Policies

```yaml
# Available hooks:
# - PreSync: Before sync operation
# - Sync: During sync operation
# - PostSync: After sync operation
# - SyncFail: On sync failure
# - Skip: Skip resource from sync

# Hook delete policies:
# - HookSucceeded: Delete after successful execution
# - HookFailed: Delete after failed execution
# - BeforeHookCreation: Delete before new hook is created
```

---

## Projects

### Create Project

```yaml
# project.yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
  namespace: argocd
spec:
  description: My Application Project

  # Source repositories
  sourceRepos:
    - https://github.com/org/repo1.git
    - https://github.com/org/repo2.git
    - '*'  # Allow all

  # Destination clusters and namespaces
  destinations:
    - namespace: '*'
      server: https://kubernetes.default.svc
    - namespace: production
      server: https://prod-cluster
    - namespace: 'team-*'
      server: '*'

  # Cluster resource whitelist
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
    - group: 'rbac.authorization.k8s.io'
      kind: ClusterRole
    - group: 'rbac.authorization.k8s.io'
      kind: ClusterRoleBinding

  # Namespace resource blacklist
  namespaceResourceBlacklist:
    - group: ''
      kind: ResourceQuota
    - group: ''
      kind: LimitRange

  # Namespace resource whitelist
  namespaceResourceWhitelist:
    - group: 'apps'
      kind: Deployment
    - group: 'apps'
      kind: StatefulSet
    - group: ''
      kind: Service

  # Sync windows
  syncWindows:
    - kind: allow
      schedule: '0 9 * * *'
      duration: 8h
      applications:
        - '*'
      namespaces:
        - production
    - kind: deny
      schedule: '0 22 * * *'
      duration: 2h
      applications:
        - '*'
      manualSync: true

  # Roles
  roles:
    - name: developer
      description: Developer role
      policies:
        - p, proj:my-project:developer, applications, get, my-project/*, allow
        - p, proj:my-project:developer, applications, sync, my-project/*, allow
      groups:
        - developers

    - name: admin
      description: Admin role
      policies:
        - p, proj:my-project:admin, applications, *, my-project/*, allow
      groups:
        - platform-team

  # Orphaned resources
  orphanedResources:
    warn: true

  # Signature keys
  signatureKeys:
    - keyID: 4AEE18F83AFDEB23
```

```bash
# Apply project
kubectl apply -f project.yaml

# Create project via CLI
argocd proj create my-project \
  --description "My Project" \
  --src https://github.com/org/repo.git \
  --dest https://kubernetes.default.svc,production

# List projects
argocd proj list

# Get project details
argocd proj get my-project

# Add source repository
argocd proj add-source my-project https://github.com/org/new-repo.git

# Add destination
argocd proj add-destination my-project https://kubernetes.default.svc staging

# Set project role
argocd proj role create my-project developer
argocd proj role add-policy my-project developer \
  --action get --permission allow --object "my-project/*"

# Delete project
argocd proj delete my-project
```

---

## Repositories

### Add Repository

```bash
# HTTPS repository
argocd repo add https://github.com/org/repo.git

# HTTPS with credentials
argocd repo add https://github.com/org/repo.git \
  --username myuser \
  --password mypassword

# SSH repository
argocd repo add git@github.com:org/repo.git \
  --ssh-private-key-path ~/.ssh/id_rsa

# Helm repository
argocd repo add https://charts.bitnami.com/bitnami \
  --type helm \
  --name bitnami

# Helm with credentials
argocd repo add https://charts.example.com \
  --type helm \
  --username myuser \
  --password mypassword

# List repositories
argocd repo list

# Remove repository
argocd repo rm https://github.com/org/repo.git
```

### Repository Credentials Template

```yaml
# repo-creds.yaml
apiVersion: v1
kind: Secret
metadata:
  name: repo-credentials
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repo-creds
stringData:
  type: git
  url: https://github.com/org
  username: myuser
  password: mytoken

---
# SSH credentials
apiVersion: v1
kind: Secret
metadata:
  name: repo-ssh-credentials
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repo-creds
stringData:
  type: git
  url: git@github.com:org
  sshPrivateKey: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    ...
    -----END OPENSSH PRIVATE KEY-----

---
# Helm credentials
apiVersion: v1
kind: Secret
metadata:
  name: helm-credentials
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repo-creds
stringData:
  type: helm
  url: https://charts.example.com
  username: myuser
  password: mypassword
```

```bash
# Apply credentials
kubectl apply -f repo-creds.yaml
```

---

## Clusters

### Add Cluster

```bash
# List available contexts
kubectl config get-contexts

# Add cluster from kubeconfig
argocd cluster add my-cluster-context

# Add cluster with name
argocd cluster add my-cluster-context --name production

# Add cluster with specific kubeconfig
argocd cluster add my-cluster --kubeconfig ~/.kube/config-prod

# List clusters
argocd cluster list

# Get cluster info
argocd cluster get https://prod-cluster

# Remove cluster
argocd cluster rm https://prod-cluster
```

### Cluster Secret

```yaml
# cluster-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-cluster
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: cluster
type: Opaque
stringData:
  name: production
  server: https://prod-cluster
  config: |
    {
      "bearerToken": "TOKEN",
      "tlsClientConfig": {
        "insecure": false,
        "caData": "BASE64_ENCODED_CA"
      }
    }
```

---

## Sync Strategies

### Manual Sync

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  syncPolicy: {}  # No automated sync
```

### Automated Sync

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  syncPolicy:
    automated:
      prune: true      # Auto delete resources not in Git
      selfHeal: true   # Auto sync when cluster state changes
      allowEmpty: false
```

### Sync Options

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  syncPolicy:
    syncOptions:
      # Validate resources before applying
      - Validate=true

      # Create namespace if doesn't exist
      - CreateNamespace=true

      # Prune propagation policy
      - PrunePropagationPolicy=foreground

      # Prune resources last
      - PruneLast=true

      # Replace instead of apply
      - Replace=true

      # Server-side apply
      - ServerSideApply=true

      # Skip schema validation
      - SkipSchemaValidation=true

      # Apply out of sync resources only
      - ApplyOutOfSyncOnly=true

      # Respect ignore differences
      - RespectIgnoreDifferences=true
```

### Sync Waves

```yaml
# Database (wave 0)
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
  annotations:
    argocd.argoproj.io/sync-wave: "0"
spec:
  # ...

---
# Application (wave 1)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  # ...

---
# Frontend (wave 2)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  annotations:
    argocd.argoproj.io/sync-wave: "2"
spec:
  # ...
```

---

## ApplicationSet

### List Generator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapps
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - cluster: production
            url: https://prod-cluster
            environment: prod
          - cluster: staging
            url: https://staging-cluster
            environment: staging
          - cluster: development
            url: https://dev-cluster
            environment: dev

  template:
    metadata:
      name: 'myapp-{{environment}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/repo.git
        targetRevision: HEAD
        path: 'k8s/{{environment}}'
      destination:
        server: '{{url}}'
        namespace: myapp
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### Git Directory Generator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cluster-addons
  namespace: argocd
spec:
  generators:
    - git:
        repoURL: https://github.com/org/cluster-config.git
        revision: HEAD
        directories:
          - path: addons/*
          - path: addons/*/overlays/production

  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/cluster-config.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
      syncPolicy:
        automated: {}
```

### Git File Generator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: multi-env-apps
  namespace: argocd
spec:
  generators:
    - git:
        repoURL: https://github.com/org/apps.git
        revision: HEAD
        files:
          - path: "apps/*/config.json"

  template:
    metadata:
      name: '{{app.name}}-{{app.environment}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/apps.git
        targetRevision: HEAD
        path: 'apps/{{app.name}}/{{app.environment}}'
      destination:
        server: '{{app.cluster}}'
        namespace: '{{app.namespace}}'
      syncPolicy:
        automated:
          prune: true
```

### Cluster Generator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: prometheus-multi-cluster
  namespace: argocd
spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            environment: production

  template:
    metadata:
      name: '{{name}}-prometheus'
    spec:
      project: monitoring
      source:
        repoURL: https://github.com/prometheus-community/helm-charts.git
        targetRevision: HEAD
        path: charts/kube-prometheus-stack
      destination:
        server: '{{server}}'
        namespace: monitoring
      syncPolicy:
        automated: {}
```

### Matrix Generator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: matrix-example
  namespace: argocd
spec:
  generators:
    - matrix:
        generators:
          - git:
              repoURL: https://github.com/org/apps.git
              revision: HEAD
              directories:
                - path: apps/*
          - list:
              elements:
                - cluster: staging
                  url: https://staging-cluster
                - cluster: production
                  url: https://prod-cluster

  template:
    metadata:
      name: '{{path.basename}}-{{cluster}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/apps.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: '{{url}}'
        namespace: '{{path.basename}}'
```

---

## Helm Integration

### Helm Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx
  namespace: argocd
spec:
  project: default

  source:
    # Helm chart from Git repo
    repoURL: https://github.com/org/charts.git
    targetRevision: HEAD
    path: charts/nginx

    # Or Helm repository
    # repoURL: https://charts.bitnami.com/bitnami
    # chart: nginx
    # targetRevision: 15.0.0

    helm:
      releaseName: nginx

      # Values files
      valueFiles:
        - values.yaml
        - values-production.yaml

      # Parameters (override values)
      parameters:
        - name: replicaCount
          value: "3"
        - name: service.type
          value: LoadBalancer
        - name: image.tag
          value: "1.25.0"

      # Inline values
      values: |
        ingress:
          enabled: true
          hosts:
            - host: nginx.example.com
              paths:
                - path: /
                  pathType: Prefix
          tls:
            - secretName: nginx-tls
              hosts:
                - nginx.example.com

      # Pass credentials to private Helm repos
      passCredentials: false

      # Skip CRDs
      skipCrds: false

  destination:
    server: https://kubernetes.default.svc
    namespace: nginx

  syncPolicy:
    automated: {}
    syncOptions:
      - CreateNamespace=true
```

### Helm Secrets

```yaml
# Using helm-secrets plugin
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  source:
    repoURL: https://github.com/org/charts.git
    path: charts/myapp
    helm:
      valueFiles:
        - values.yaml
        - secrets://secrets.yaml  # Encrypted with sops
```

---

## Kustomize Integration

### Kustomize Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/org/repo.git
    targetRevision: HEAD
    path: k8s/overlays/production

    kustomize:
      # Name prefix
      namePrefix: prod-

      # Name suffix
      nameSuffix: -v1

      # Images (kustomize edit set image)
      images:
        - myapp=myregistry/myapp:v1.0.0
        - postgres=postgres:14

      # Common labels
      commonLabels:
        environment: production
        managed-by: argocd

      # Common annotations
      commonAnnotations:
        deployed-by: argocd
        version: v1.0.0

      # Version
      version: v4.5.7

      # Namespace
      namespace: production

      # Force common labels
      forceCommonLabels: false

      # Force common annotations
      forceCommonAnnotations: false

      # Common annotations from environment
      commonAnnotationsEnvsubst: true

  destination:
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:
    automated: {}
```

---

## Multi-Tenancy

### App of Apps Pattern

```yaml
# root-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/org/root-apps.git
    targetRevision: HEAD
    path: apps

  destination:
    server: https://kubernetes.default.svc
    namespace: argocd

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

```
# Repository structure
apps/
├── app1.yaml
├── app2.yaml
├── app3.yaml
└── infrastructure/
    ├── ingress-nginx.yaml
    ├── cert-manager.yaml
    └── prometheus.yaml
```

### Team-Based Projects

```yaml
# team-a-project.yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-a
  namespace: argocd
spec:
  description: Team A Applications

  sourceRepos:
    - https://github.com/org/team-a-*

  destinations:
    - namespace: 'team-a-*'
      server: '*'

  clusterResourceWhitelist:
    - group: ''
      kind: Namespace

  roles:
    - name: developer
      policies:
        - p, proj:team-a:developer, applications, *, team-a/*, allow
      groups:
        - team-a-developers
```

---

## Security

### Enable TLS

```bash
# Generate TLS certificate
kubectl create secret tls argocd-server-tls \
  --cert=path/to/cert.crt \
  --key=path/to/cert.key \
  -n argocd

# Or use cert-manager
cat > argocd-certificate.yaml <<EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: argocd-server
  namespace: argocd
spec:
  secretName: argocd-server-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
    - argocd.example.com
EOF

kubectl apply -f argocd-certificate.yaml
```

### Disable Admin User

```bash
# Edit argocd-cm ConfigMap
kubectl edit configmap argocd-cm -n argocd
```

```yaml
data:
  admin.enabled: "false"
```

### API Rate Limiting

```yaml
# argocd-cmd-params-cm
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cmd-params-cm
  namespace: argocd
data:
  # Rate limit for API requests
  server.api.rate.limit: "1000"
```

---

## SSO and RBAC

### SSO with DEX (OIDC)

```yaml
# argocd-cm ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  url: https://argocd.example.com

  dex.config: |
    connectors:
      # GitHub
      - type: github
        id: github
        name: GitHub
        config:
          clientID: $github-client-id
          clientSecret: $github-client-secret
          orgs:
            - name: my-org

      # Google
      - type: oidc
        id: google
        name: Google
        config:
          issuer: https://accounts.google.com
          clientID: $google-client-id
          clientSecret: $google-client-secret

      # GitLab
      - type: gitlab
        id: gitlab
        name: GitLab
        config:
          baseURL: https://gitlab.com
          clientID: $gitlab-client-id
          clientSecret: $gitlab-client-secret
          groups:
            - my-group

      # LDAP
      - type: ldap
        name: LDAP
        id: ldap
        config:
          host: ldap.example.com:636
          insecureSkipVerify: false
          bindDN: cn=admin,dc=example,dc=com
          bindPW: $ldap-password
          userSearch:
            baseDN: ou=users,dc=example,dc=com
            filter: "(objectClass=person)"
            username: uid
            idAttr: uid
            emailAttr: mail
            nameAttr: cn
          groupSearch:
            baseDN: ou=groups,dc=example,dc=com
            filter: "(objectClass=groupOfNames)"
            userAttr: DN
            groupAttr: member
            nameAttr: cn
```

### RBAC Configuration

```yaml
# argocd-rbac-cm ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  # Default role for unauthenticated users
  policy.default: role:readonly

  # CSV RBAC policies
  policy.csv: |
    # Format: p, subject, resource, action, object, effect

    # Admins - full access
    p, role:admin, applications, *, */*, allow
    p, role:admin, clusters, *, *, allow
    p, role:admin, repositories, *, *, allow
    p, role:admin, projects, *, *, allow
    p, role:admin, accounts, *, *, allow
    p, role:admin, certificates, *, *, allow
    p, role:admin, gpgkeys, *, *, allow

    # Developers - app management
    p, role:developer, applications, get, */*, allow
    p, role:developer, applications, create, */*, allow
    p, role:developer, applications, update, */*, allow
    p, role:developer, applications, sync, */*, allow
    p, role:developer, applications, delete, */*, deny
    p, role:developer, repositories, get, *, allow
    p, role:developer, clusters, get, *, allow

    # Read-only
    p, role:readonly, applications, get, */*, allow
    p, role:readonly, repositories, get, *, allow
    p, role:readonly, clusters, get, *, allow
    p, role:readonly, projects, get, *, allow

    # Project-specific roles
    p, role:team-a-admin, applications, *, team-a/*, allow
    p, role:team-a-developer, applications, get, team-a/*, allow
    p, role:team-a-developer, applications, sync, team-a/*, allow

    # Group bindings
    g, platform-team, role:admin
    g, developers, role:developer
    g, team-a-admins, role:team-a-admin
    g, team-a-devs, role:team-a-developer

    # User bindings
    g, alice@example.com, role:admin
    g, bob@example.com, role:developer

  # Scopes for JWT tokens
  scopes: '[groups, email]'
```

### Local Users

```bash
# Create local user
argocd account create alice

# Set password
argocd account update-password --account alice

# List accounts
argocd account list

# Disable account
argocd account update-password --account alice --current-password OLD --new-password NEW
```

---

## Notifications

### Install Notifications

```bash
# Install notifications controller
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-notifications/release-1.0/manifests/install.yaml

# Or via Helm
helm upgrade --install argocd-notifications \
  argo/argocd-notifications \
  -n argocd
```

### Configure Notifications

```yaml
# argocd-notifications-cm ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  # Slack service
  service.slack: |
    token: $slack-token

  # Webhook service
  service.webhook.teams: |
    url: $teams-webhook-url

  # Email service
  service.email.gmail: |
    username: $email-username
    password: $email-password
    host: smtp.gmail.com
    port: 587
    from: argocd@example.com

  # Templates
  template.app-deployed: |
    message: |
      Application {{.app.metadata.name}} is now running new version.
    email:
      subject: New version of an application {{.app.metadata.name}} is up and running.
    slack:
      attachments: |
        [{
          "title": "{{ .app.metadata.name}}",
          "title_link":"{{.context.argocdUrl}}/applications/{{.app.metadata.name}}",
          "color": "#18be52",
          "fields": [
          {
            "title": "Sync Status",
            "value": "{{.app.status.sync.status}}",
            "short": true
          },
          {
            "title": "Repository",
            "value": "{{.app.spec.source.repoURL}}",
            "short": true
          }
          {{range $index, $c := .app.status.conditions}}
          {{if not $index}},{{end}}
          {{if $index}},{{end}}
          {
            "title": "{{$c.type}}",
            "value": "{{$c.message}}",
            "short": true
          }
          {{end}}
          ]
        }]

  template.app-health-degraded: |
    message: |
      Application {{.app.metadata.name}} has degraded.
    slack:
      attachments: |
        [{
          "title": "{{ .app.metadata.name}}",
          "color": "#f4c030",
          "fields": [
          {
            "title": "Health Status",
            "value": "{{.app.status.health.status}}",
            "short": true
          }
          ]
        }]

  template.app-sync-failed: |
    message: |
      Application {{.app.metadata.name}} sync failed.
    slack:
      attachments: |
        [{
          "title": "{{ .app.metadata.name}}",
          "color": "#E96D76",
          "fields": [
          {
            "title": "Sync Status",
            "value": "{{.app.status.sync.status}}",
            "short": true
          }
          ]
        }]

  # Triggers
  trigger.on-deployed: |
    - when: app.status.operationState.phase in ['Succeeded'] and app.status.health.status == 'Healthy'
      send: [app-deployed]

  trigger.on-health-degraded: |
    - when: app.status.health.status == 'Degraded'
      send: [app-health-degraded]

  trigger.on-sync-failed: |
    - when: app.status.operationState.phase in ['Error', 'Failed']
      send: [app-sync-failed]
```

```yaml
# argocd-notifications-secret
apiVersion: v1
kind: Secret
metadata:
  name: argocd-notifications-secret
  namespace: argocd
stringData:
  slack-token: xoxb-your-slack-token
  teams-webhook-url: https://outlook.office.com/webhook/...
  email-username: user@gmail.com
  email-password: app-password
```

### Subscribe to Notifications

```yaml
# Subscribe via annotations
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  annotations:
    notifications.argoproj.io/subscribe.on-deployed.slack: my-channel
    notifications.argoproj.io/subscribe.on-sync-failed.slack: alerts-channel
    notifications.argoproj.io/subscribe.on-health-degraded.email: team@example.com
```

---

## Image Updater

### Install Image Updater

```bash
# Install via kubectl
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml

# Or via Helm
helm install argocd-image-updater \
  argo/argocd-image-updater \
  -n argocd
```

### Configure Image Updater

```yaml
# Annotate application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  annotations:
    # Image list to update
    argocd-image-updater.argoproj.io/image-list: myapp=myregistry/myapp

    # Update strategy
    argocd-image-updater.argoproj.io/myapp.update-strategy: semver

    # Semver constraint
    argocd-image-updater.argoproj.io/myapp.semver.constraint: ~1.0

    # Write back method
    argocd-image-updater.argoproj.io/write-back-method: git

    # Git branch
    argocd-image-updater.argoproj.io/git-branch: main

    # Kustomize image
    argocd-image-updater.argoproj.io/myapp.kustomize.image-name: myregistry/myapp

    # Helm parameter
    argocd-image-updater.argoproj.io/myapp.helm.image-name: image.repository
    argocd-image-updater.argoproj.io/myapp.helm.image-tag: image.tag

    # Pull secret
    argocd-image-updater.argoproj.io/myapp.pull-secret: pullsecret:argocd/myapp-pull-secret
spec:
  # ...
```

### Update Strategies

- **semver**: Semantic versioning (e.g., `~1.0`, `^2.3`)
- **latest**: Always use latest tag
- **name**: Use specific tag name
- **digest**: Use image digest

---

## Secrets Management

### Sealed Secrets

```bash
# Install Sealed Secrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# Install kubeseal CLI
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/kubeseal-linux-amd64
chmod +x kubeseal-linux-amd64
sudo mv kubeseal-linux-amd64 /usr/local/bin/kubeseal

# Create sealed secret
kubectl create secret generic mysecret \
  --from-literal=password=mypassword \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > sealedsecret.yaml

# Commit sealedsecret.yaml to Git
git add sealedsecret.yaml
git commit -m "Add sealed secret"
```

### External Secrets Operator

```bash
# Install External Secrets
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets \
  external-secrets/external-secrets \
  -n external-secrets-system \
  --create-namespace
```

```yaml
# SecretStore
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secretsmanager
  namespace: default
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets

---
# ExternalSecret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-secret
  namespace: default
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secretsmanager
    kind: SecretStore
  target:
    name: myapp-secret
  data:
    - secretKey: password
      remoteRef:
        key: myapp/production
        property: password
```

### SOPS

```yaml
# .sops.yaml
creation_rules:
  - path_regex: .*.yaml
    encrypted_regex: ^(data|stringData)$
    pgp: YOUR_PGP_FINGERPRINT

# Use with Argo CD
# Install argocd-vault-plugin or ksops
```

---

## High Availability

### HA Installation

```bash
# Install HA version
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/ha/install.yaml
```

### HA Configuration

```yaml
# Redis HA
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: argocd-redis-ha-server
spec:
  replicas: 3
  # ...

---
# Application Controller HA
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: argocd-application-controller
spec:
  replicas: 3
  # ...

---
# Repo Server HA
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd-repo-server
spec:
  replicas: 3
  # ...

---
# Server HA
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd-server
spec:
  replicas: 3
  # ...
```

---

## Monitoring and Metrics

### Prometheus Metrics

```bash
# Port forward to metrics endpoint
kubectl port-forward svc/argocd-metrics -n argocd 8082:8082

# Access metrics
curl http://localhost:8082/metrics
```

### Service Monitors

```yaml
# argocd-metrics ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-metrics
  namespace: argocd
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-metrics
  endpoints:
    - port: metrics

---
# argocd-server-metrics ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-server-metrics
  namespace: argocd
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-server-metrics
  endpoints:
    - port: metrics

---
# argocd-repo-server-metrics ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-repo-server-metrics
  namespace: argocd
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-repo-server
  endpoints:
    - port: metrics
```

### Grafana Dashboards

```bash
# Import Argo CD dashboards
# Dashboard ID: 14584 (Argo CD)
# Dashboard ID: 16104 (Argo CD Application Insights)
```

---

## CLI Commands

### Application Commands

```bash
# Create
argocd app create APP_NAME --repo REPO --path PATH --dest-server SERVER --dest-namespace NAMESPACE

# List
argocd app list
argocd app list -o wide
argocd app list -o json

# Get
argocd app get APP_NAME
argocd app get APP_NAME --refresh

# Sync
argocd app sync APP_NAME
argocd app sync APP_NAME --force
argocd app sync APP_NAME --dry-run
argocd app sync APP_NAME --prune

# Diff
argocd app diff APP_NAME

# Rollback
argocd app rollback APP_NAME HISTORY_ID

# Delete
argocd app delete APP_NAME

# Wait
argocd app wait APP_NAME --timeout 300

# Manifests
argocd app manifests APP_NAME

# Logs
argocd app logs APP_NAME
```

### Project Commands

```bash
# Create
argocd proj create PROJECT_NAME

# List
argocd proj list

# Get
argocd proj get PROJECT_NAME

# Add source
argocd proj add-source PROJECT_NAME REPO_URL

# Add destination
argocd proj add-destination PROJECT_NAME SERVER NAMESPACE

# Create role
argocd proj role create PROJECT_NAME ROLE_NAME

# Delete
argocd proj delete PROJECT_NAME
```

### Repository Commands

```bash
# Add
argocd repo add REPO_URL
argocd repo add REPO_URL --username USER --password PASS

# List
argocd repo list

# Remove
argocd repo rm REPO_URL
```

### Cluster Commands

```bash
# Add
argocd cluster add CONTEXT_NAME

# List
argocd cluster list

# Get
argocd cluster get SERVER_URL

# Remove
argocd cluster rm SERVER_URL
```

---

## Best Practices

### Repository Structure

```
gitops-repo/
├── apps/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── kustomization.yaml
│   └── overlays/
│       ├── dev/
│       │   ├── kustomization.yaml
│       │   └── patches.yaml
│       ├── staging/
│       │   ├── kustomization.yaml
│       │   └── patches.yaml
│       └── production/
│           ├── kustomization.yaml
│           └── patches.yaml
├── argocd/
│   ├── applications/
│   │   ├── app1.yaml
│   │   ├── app2.yaml
│   │   └── app3.yaml
│   └── projects/
│       ├── team-a.yaml
│       └── team-b.yaml
└── infrastructure/
    ├── cert-manager/
    ├── ingress-nginx/
    └── prometheus/
```

### Application Best Practices

1. **Use Projects** for multi-tenancy
2. **Enable Auto-Sync** with caution
3. **Use Sync Waves** for ordered deployments
4. **Implement Health Checks** properly
5. **Use Resource Hooks** for migrations
6. **Set Resource Limits** on Argo CD components
7. **Enable Notifications** for failures
8. **Use Sealed Secrets** or External Secrets
9. **Implement RBAC** properly
10. **Monitor Metrics** in Prometheus

### Security Best Practices

```yaml
# 1. Disable admin user (use SSO)
# 2. Enable TLS
# 3. Use RBAC
# 4. Encrypt secrets
# 5. Use private endpoints
# 6. Enable audit logging
# 7. Scan images
# 8. Use network policies
# 9. Regular updates
# 10. Backup configuration
```

---

## Troubleshooting

### Common Issues

```bash
# Application stuck in progressing
argocd app get APP_NAME
argocd app sync APP_NAME --force
argocd app sync APP_NAME --prune

# Refresh application
argocd app get APP_NAME --refresh
argocd app get APP_NAME --hard-refresh

# Check logs
kubectl logs -n argocd deployment/argocd-server
kubectl logs -n argocd deployment/argocd-repo-server
kubectl logs -n argocd statefulset/argocd-application-controller

# Check events
kubectl get events -n argocd --sort-by='.lastTimestamp'

# Sync fails
argocd app diff APP_NAME
kubectl get events -n NAMESPACE

# Out of sync resources
argocd app diff APP_NAME
argocd app sync APP_NAME --force

# Authentication issues
argocd account list
argocd relogin

# Repository connection issues
argocd repo list
argocd repo add REPO_URL --insecure-skip-server-verification

# Delete stuck application
kubectl patch app APP_NAME -n argocd \
  --type json \
  -p='[{"op": "remove", "path": "/metadata/finalizers"}]'
```

### Debug Mode

```bash
# Enable debug logging
kubectl edit configmap argocd-cmd-params-cm -n argocd

# Add:
data:
  server.log.level: debug
  controller.log.level: debug
  reposerver.log.level: debug

# Restart components
kubectl rollout restart deployment argocd-server -n argocd
kubectl rollout restart statefulset argocd-application-controller -n argocd
kubectl rollout restart deployment argocd-repo-server -n argocd
```

---

## Advanced Patterns

### Progressive Delivery with Argo Rollouts

```bash
# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

```yaml
# Canary deployment
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 5
  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: {duration: 1m}
        - setWeight: 40
        - pause: {duration: 1m}
        - setWeight: 60
        - pause: {duration: 1m}
        - setWeight: 80
        - pause: {duration: 1m}
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
          image: myapp:v2.0.0
```

### Multi-Cluster Management

```yaml
# ApplicationSet for multi-cluster
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: multi-cluster-app
spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            environment: production
  template:
    metadata:
      name: '{{name}}-myapp'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/repo.git
        path: k8s
      destination:
        server: '{{server}}'
        namespace: myapp
```

---

## Practical Examples

### Complete GitOps Workflow

```yaml
# 1. Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: production

  source:
    repoURL: https://github.com/org/myapp.git
    targetRevision: main
    path: k8s/overlays/production
    kustomize:
      images:
        - myapp=registry.example.com/myapp:v1.0.0

  destination:
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas

---
# 2. Project
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production Applications
  sourceRepos:
    - '*'
  destinations:
    - namespace: 'production'
      server: '*'
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
```

```bash
# Deploy
kubectl apply -f application.yaml
kubectl apply -f project.yaml

# Monitor
argocd app get myapp-prod --watch

# Sync
argocd app sync myapp-prod
```

---

## Conclusion

This comprehensive Argo CD guide covers:

1. **Installation**: Multiple installation methods
2. **Applications**: Complete application lifecycle
3. **Projects**: Multi-tenancy and access control
4. **Sync Strategies**: Automated and manual sync
5. **ApplicationSet**: Multi-cluster and multi-environment
6. **Integrations**: Helm, Kustomize, and more
7. **Security**: SSO, RBAC, secrets management
8. **HA**: High availability setup
9. **Monitoring**: Metrics and dashboards
10. **Best Practices**: Production-ready patterns

### Additional Resources

- [Argo CD Documentation](https://argo-cd.readthedocs.io/)
- [Argo CD GitHub](https://github.com/argoproj/argo-cd)
- [GitOps Principles](https://opengitops.dev/)
- [Argo Project](https://argoproj.github.io/)

---

**Remember**: GitOps is a methodology, not just a tool. Keep Git as your single source of truth and let Argo CD handle the rest!
