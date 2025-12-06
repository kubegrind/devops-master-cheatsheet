# GitOps Complete Guide

## Table of Contents
- [Introduction](#introduction)
- [GitOps Principles](#gitops-principles)
- [GitOps Tools](#gitops-tools)
- [ArgoCD](#argocd)
- [Flux CD](#flux-cd)
- [Jenkins X](#jenkins-x)
- [GitOps Workflows](#gitops-workflows)
- [Multi-Cluster Management](#multi-cluster-management)
- [Security & Best Practices](#security--best-practices)
- [Troubleshooting](#troubleshooting)
- [Advanced Patterns](#advanced-patterns)

---

## Introduction

GitOps is a modern approach to continuous deployment where Git serves as the single source of truth for declarative infrastructure and applications. The desired state of the system is versioned in Git, and automated processes ensure the actual state matches the desired state.

### What is GitOps?

GitOps is an operational framework that applies DevOps best practices (version control, collaboration, compliance, CI/CD) to infrastructure automation. It uses Git repositories as the source of truth for defining infrastructure and application state.

### Key Benefits

- **Version Control**: All changes tracked in Git
- **Rollback**: Easy rollback to previous states
- **Audit Trail**: Complete history of who changed what and when
- **Collaboration**: Standard Git workflows (PRs, reviews)
- **Consistency**: Declarative configs ensure consistent deployments
- **Security**: Git as authentication and authorization mechanism
- **Disaster Recovery**: Git repo can restore entire system state

### GitOps vs Traditional CI/CD

| Aspect | Traditional CI/CD | GitOps |
|--------|------------------|--------|
| Deployment Trigger | Push-based | Pull-based |
| Source of Truth | CI/CD Pipeline | Git Repository |
| Deployment Tool | CI/CD Server | GitOps Agent |
| State Management | Manual | Automated Sync |
| Rollback | Complex | Git revert |
| Audit | CI/CD logs | Git history |

---

## GitOps Principles

### 1. Declarative Configuration

Everything must be described declaratively (YAML, JSON, HCL):

```yaml
# Good - Declarative
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: myapp:v1.0.0
```

```bash
# Bad - Imperative
kubectl scale deployment myapp --replicas=3
kubectl set image deployment/myapp app=myapp:v1.0.0
```

### 2. Git as Single Source of Truth

All desired state stored in Git:

```
gitops-repo/
├── apps/
│   ├── production/
│   │   ├── frontend/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── ingress.yaml
│   │   └── backend/
│   │       ├── deployment.yaml
│   │       └── service.yaml
│   └── staging/
│       └── ...
├── infrastructure/
│   ├── namespaces/
│   ├── rbac/
│   └── network-policies/
└── cluster-config/
    ├── ingress-controller/
    ├── cert-manager/
    └── monitoring/
```

### 3. Automated Delivery

GitOps operators continuously reconcile desired vs actual state:

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  source:
    repoURL: https://github.com/org/gitops-repo
    targetRevision: HEAD
    path: apps/production/myapp
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### 4. Continuous Reconciliation

Agents continuously monitor and sync state:

```bash
# GitOps operator workflow
while true; do
  desired_state = fetch_from_git()
  actual_state = fetch_from_cluster()

  if desired_state != actual_state:
    apply_changes(desired_state)

  sleep(sync_interval)
done
```

---

## GitOps Tools

### Tool Comparison

| Tool | Type | Language | Platform | Strengths |
|------|------|----------|----------|-----------|
| ArgoCD | Pull | Go | Kubernetes | UI, Multi-cluster, App management |
| Flux | Pull | Go | Kubernetes | Native K8s, Helm support, Lightweight |
| Jenkins X | Hybrid | Go | Kubernetes | Full CI/CD, Preview envs |
| Spinnaker | Push | Java | Multi-cloud | Multi-cloud, Complex pipelines |
| Tekton | Push | Go | Kubernetes | Cloud-native, K8s native |
| GitLab CI/CD | Push | Ruby/Go | Multi | Integrated platform |

### ArgoCD vs Flux

```yaml
# ArgoCD - Application-centric
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  source:
    repoURL: https://github.com/org/repo
    path: apps/myapp
  destination:
    namespace: default
  syncPolicy:
    automated: {}

---
# Flux - Resource-centric
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: myapp-repo
spec:
  interval: 1m
  url: https://github.com/org/repo
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: myapp
spec:
  interval: 5m
  sourceRef:
    kind: GitRepository
    name: myapp-repo
  path: ./apps/myapp
```

---

## ArgoCD

### Installation

#### Quick Installation

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=available --timeout=600s deployment/argocd-server -n argocd

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Access UI at https://localhost:8080
# Username: admin
# Password: (from above command)
```

#### Production Installation with HA

```yaml
# argocd-ha-install.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: argocd
---
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  namespace: argocd
spec:
  server:
    replicas: 3
    ingress:
      enabled: true
      hosts:
        - argocd.example.com
      tls:
        - secretName: argocd-tls
          hosts:
            - argocd.example.com
  redis:
    enabled: true
  controller:
    replicas: 3
  repo:
    replicas: 3
  ha:
    enabled: true
```

```bash
# Install with Helm
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  --set server.replicas=3 \
  --set controller.replicas=3 \
  --set repoServer.replicas=3 \
  --set redis-ha.enabled=true
```

### ArgoCD CLI

```bash
# Install ArgoCD CLI
# macOS
brew install argocd

# Linux
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/

# Windows
choco install argocd-cli

# Login
argocd login localhost:8080 --username admin --password <password> --insecure

# Change admin password
argocd account update-password

# List clusters
argocd cluster list

# Add cluster
argocd cluster add my-cluster-context

# List applications
argocd app list

# Get application details
argocd app get myapp

# Create application
argocd app create myapp \
  --repo https://github.com/org/repo \
  --path apps/myapp \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Sync application
argocd app sync myapp

# Delete application
argocd app delete myapp

# Set auto-sync
argocd app set myapp --sync-policy automated

# Rollback
argocd app rollback myapp
```

### Application Deployment

#### Basic Application

```yaml
# application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps
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
kubectl apply -f application.yaml

# Monitor sync status
argocd app get guestbook --watch

# Sync manually
argocd app sync guestbook

# Diff between desired and live state
argocd app diff guestbook
```

#### Helm Application

```yaml
# helm-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.bitnami.com/bitnami
    chart: nginx
    targetRevision: 15.0.0
    helm:
      releaseName: nginx
      parameters:
      - name: service.type
        value: LoadBalancer
      - name: replicaCount
        value: "3"
      values: |
        ingress:
          enabled: true
          hostname: nginx.example.com
  destination:
    server: https://kubernetes.default.svc
    namespace: nginx
  syncPolicy:
    automated: {}
    syncOptions:
    - CreateNamespace=true
```

#### Kustomize Application

```yaml
# kustomize-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/repo
    targetRevision: HEAD
    path: overlays/production
    kustomize:
      images:
      - myapp:v1.2.3
      namePrefix: prod-
      commonLabels:
        environment: production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

### AppProject (Multi-Tenancy)

```yaml
# project.yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-a
  namespace: argocd
spec:
  description: Team A's applications

  # Source repositories
  sourceRepos:
  - https://github.com/org/team-a-apps
  - https://github.com/org/shared-charts

  # Allowed destinations
  destinations:
  - namespace: team-a-*
    server: https://kubernetes.default.svc
  - namespace: team-a-staging
    server: https://staging-cluster

  # Cluster resource whitelist
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  - group: 'rbac.authorization.k8s.io'
    kind: ClusterRole

  # Namespace resource blacklist
  namespaceResourceBlacklist:
  - group: ''
    kind: ResourceQuota
  - group: ''
    kind: LimitRange

  # RBAC roles
  roles:
  - name: developer
    description: Developer role
    policies:
    - p, proj:team-a:developer, applications, get, team-a/*, allow
    - p, proj:team-a:developer, applications, sync, team-a/*, allow
    groups:
    - team-a-developers

  - name: admin
    description: Admin role
    policies:
    - p, proj:team-a:admin, applications, *, team-a/*, allow
    groups:
    - team-a-admins
```

### ApplicationSet (Multiple Environments)

```yaml
# applicationset.yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp
  namespace: argocd
spec:
  generators:
  - list:
      elements:
      - cluster: in-cluster
        url: https://kubernetes.default.svc
        environment: dev
        namespace: dev
      - cluster: staging
        url: https://staging.k8s.local
        environment: staging
        namespace: staging
      - cluster: production
        url: https://prod.k8s.local
        environment: production
        namespace: production

  template:
    metadata:
      name: 'myapp-{{environment}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/repo
        targetRevision: HEAD
        path: 'envs/{{environment}}'
      destination:
        server: '{{url}}'
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### Git Generator Pattern

```yaml
# git-generator-directories.yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cluster-addons
  namespace: argocd
spec:
  generators:
  - git:
      repoURL: https://github.com/org/cluster-addons
      revision: HEAD
      directories:
      - path: addons/*

  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/cluster-addons
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
      syncPolicy:
        automated: {}
```

### ArgoCD Notifications

```yaml
# notifications-cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  # Slack notification template
  template.app-deployed: |
    message: |
      Application {{.app.metadata.name}} is now running new version.
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
          ]
        }]

  # Trigger configuration
  trigger.on-deployed: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [app-deployed]

  # Slack service
  service.slack: |
    token: $slack-token

---
# notifications-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: argocd-notifications-secret
  namespace: argocd
stringData:
  slack-token: xoxb-your-slack-token
```

```yaml
# Add notification annotation to application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  annotations:
    notifications.argoproj.io/subscribe.on-deployed.slack: my-channel
```

### ArgoCD Image Updater

```bash
# Install ArgoCD Image Updater
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```

```yaml
# Annotate application for auto image updates
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  annotations:
    argocd-image-updater.argoproj.io/image-list: myapp=docker.io/org/myapp
    argocd-image-updater.argoproj.io/myapp.update-strategy: semver
    argocd-image-updater.argoproj.io/myapp.semver.constraint: ~1.0
    argocd-image-updater.argoproj.io/write-back-method: git
spec:
  source:
    repoURL: https://github.com/org/repo
    path: apps/myapp
```

---

## Flux CD

### Installation

```bash
# Install Flux CLI
# macOS
brew install fluxcd/tap/flux

# Linux
curl -s https://fluxcd.io/install.sh | sudo bash

# Windows
choco install flux

# Verify installation
flux --version

# Check prerequisites
flux check --pre

# Bootstrap Flux on cluster
export GITHUB_TOKEN=<your-token>

flux bootstrap github \
  --owner=my-org \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/production \
  --personal

# Bootstrap with different Git provider
flux bootstrap gitlab \
  --owner=my-org \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/production \
  --token-auth

# Check Flux installation
flux check
```

### GitRepository Source

```yaml
# gitrepository.yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: myapp
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/org/myapp
  ref:
    branch: main
  secretRef:
    name: git-credentials
  ignore: |
    # exclude all
    /*
    # include charts directory
    !/charts/
```

### Kustomization Controller

```yaml
# kustomization.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: myapp
  namespace: flux-system
spec:
  interval: 5m
  sourceRef:
    kind: GitRepository
    name: myapp
  path: ./deploy/production
  prune: true
  wait: true
  timeout: 2m
  healthChecks:
  - apiVersion: apps/v1
    kind: Deployment
    name: myapp
    namespace: default
```

### HelmRepository & HelmRelease

```yaml
# helmrepository.yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: bitnami
  namespace: flux-system
spec:
  interval: 10m
  url: https://charts.bitnami.com/bitnami

---
# helmrelease.yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: nginx
  namespace: default
spec:
  interval: 5m
  chart:
    spec:
      chart: nginx
      version: '15.x'
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
  values:
    service:
      type: LoadBalancer
    replicaCount: 3
  install:
    remediation:
      retries: 3
  upgrade:
    remediation:
      retries: 3
```

### Image Automation

```yaml
# imagerepository.yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: myapp
  namespace: flux-system
spec:
  image: docker.io/org/myapp
  interval: 1m

---
# imagepolicy.yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: myapp
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: myapp
  policy:
    semver:
      range: '>=1.0.0 <2.0.0'

---
# imageupdateautomation.yaml
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: myapp
  namespace: flux-system
spec:
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: myapp
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        email: fluxcdbot@example.com
        name: fluxcdbot
      messageTemplate: |
        Update image to {{range .Updated.Images}}{{println .}}{{end}}
    push:
      branch: main
  update:
    path: ./deploy/production
    strategy: Setters
```

```yaml
# deployment.yaml with image marker
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: app
        image: docker.io/org/myapp:v1.0.0 # {"$imagepolicy": "flux-system:myapp"}
```

### Flux CLI Commands

```bash
# Install/Update Flux
flux install
flux upgrade

# Uninstall Flux
flux uninstall

# Create source
flux create source git myapp \
  --url=https://github.com/org/myapp \
  --branch=main \
  --interval=1m

# Create kustomization
flux create kustomization myapp \
  --source=GitRepository/myapp \
  --path="./deploy/production" \
  --prune=true \
  --interval=5m

# Create HelmRelease
flux create helmrelease nginx \
  --source=HelmRepository/bitnami \
  --chart=nginx \
  --values=./values.yaml

# Reconcile resources
flux reconcile source git myapp
flux reconcile kustomization myapp
flux reconcile helmrelease nginx

# Suspend/Resume automation
flux suspend kustomization myapp
flux resume kustomization myapp

# Export resources
flux export source git myapp > gitrepository.yaml
flux export kustomization myapp > kustomization.yaml

# Get all Flux resources
flux get sources git
flux get kustomizations
flux get helmreleases

# Logs
flux logs --all-namespaces --follow
flux logs --kind=Kustomization --name=myapp

# Tree view
flux tree kustomization myapp

# Diff
flux diff kustomization myapp --path ./deploy/production
```

### Multi-Cluster with Flux

```yaml
# clusters/production/flux-system/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- gotk-components.yaml
- gotk-sync.yaml

---
# clusters/production/infrastructure.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: infrastructure
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/production
  prune: true

---
# clusters/production/apps.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 5m
  dependsOn:
  - name: infrastructure
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./apps/production
  prune: true
```

---

## Jenkins X

### Installation

```bash
# Install jx CLI
# macOS
brew install jenkins-x/jx/jx

# Linux
curl -L https://github.com/jenkins-x/jx/releases/download/$(curl --silent https://api.github.com/repos/jenkins-x/jx/releases/latest | jq -r '.tag_name')/jx-linux-amd64.tar.gz | tar xzv
sudo mv jx /usr/local/bin

# Verify
jx version

# Install Jenkins X on cluster
jx admin operator install

# Create cluster with Jenkins X
jx create cluster gke \
  --cluster-name=my-cluster \
  --project-id=my-project \
  --zone=us-central1-a
```

### Quickstart Project

```bash
# Create new project
jx create quickstart \
  --name=myapp \
  --language=go

# Import existing project
jx import --url https://github.com/org/myapp

# View pipelines
jx get pipelines

# View activities
jx get activities

# View logs
jx logs

# Get build logs
jx get build logs
```

### Preview Environments

```yaml
# .lighthouse/jenkins-x/pullrequest.yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: pullrequest
spec:
  pipelineSpec:
    tasks:
    - name: from-build-pack
      taskSpec:
        steps:
        - image: uses:jenkins-x/jx3-pipeline-catalog/tasks/go/pullrequest.yaml@versionStream
          name: ""
```

```bash
# Create preview environment
jx preview create

# List preview environments
jx get preview

# Delete preview
jx delete preview
```

---

## GitOps Workflows

### Environment Promotion

#### Directory Structure

```
gitops-repo/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── environments/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── patches.yaml
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   └── patches.yaml
│   └── production/
│       ├── kustomization.yaml
│       └── patches.yaml
└── apps/
    └── myapp/
        ├── dev/
        ├── staging/
        └── production/
```

#### Kustomize Overlays

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml

---
# environments/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev
namePrefix: dev-
commonLabels:
  environment: dev
resources:
- ../../base
images:
- name: myapp
  newTag: dev-latest
replicas:
- name: myapp
  count: 1

---
# environments/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: production
namePrefix: prod-
commonLabels:
  environment: production
resources:
- ../../base
images:
- name: myapp
  newTag: v1.2.3
replicas:
- name: myapp
  count: 3
patches:
- path: patches.yaml
```

```yaml
# environments/production/patches.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: app
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 1Gi
```

### Branch-Based Promotion

```bash
# Development branch
git checkout develop
# Update image tag
kustomize edit set image myapp:dev-abc123
git commit -am "Update dev image"
git push

# Promote to staging
git checkout staging
git merge develop
git push

# Promote to production (via PR)
git checkout -b release/v1.2.3
git merge staging
# Create PR to main branch
gh pr create --base main --title "Release v1.2.3"
```

### Tag-Based Promotion

```yaml
# ArgoCD with tag tracking
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
spec:
  source:
    repoURL: https://github.com/org/gitops-repo
    targetRevision: v1.2.3  # Track specific tag
    path: apps/myapp/production
```

```bash
# Create release tag
git tag -a v1.2.3 -m "Release version 1.2.3"
git push origin v1.2.3

# Update ArgoCD application
argocd app set myapp-prod --revision v1.2.3
```

### Pull Request Workflow

```yaml
# .github/workflows/gitops-pr.yaml
name: GitOps PR Validation
on:
  pull_request:
    paths:
    - 'apps/**'
    - 'infrastructure/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Validate Kubernetes manifests
      run: |
        kubectl apply --dry-run=client -f apps/

    - name: Kustomize build
      run: |
        kustomize build environments/production

    - name: Run kubeval
      uses: instrumenta/kubeval-action@master
      with:
        files: apps/

    - name: ArgoCD diff
      run: |
        argocd app diff myapp --local environments/production
```

### Automated Image Updates

```yaml
# GitHub Actions - Update image tag
name: Update Image Tag
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment'
        required: true
        default: 'dev'
      image_tag:
        description: 'Image tag'
        required: true

jobs:
  update-tag:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Update kustomization
      run: |
        cd environments/${{ github.event.inputs.environment }}
        kustomize edit set image myapp:${{ github.event.inputs.image_tag }}

    - name: Commit changes
      run: |
        git config user.name "GitHub Actions"
        git config user.email "actions@github.com"
        git add .
        git commit -m "Update ${{ github.event.inputs.environment }} to ${{ github.event.inputs.image_tag }}"
        git push
```

---

## Multi-Cluster Management

### Cluster Registration (ArgoCD)

```bash
# List contexts
kubectl config get-contexts

# Add cluster to ArgoCD
argocd cluster add staging-cluster \
  --name staging \
  --kubeconfig ~/.kube/config

argocd cluster add production-cluster \
  --name production \
  --kubeconfig ~/.kube/config

# List registered clusters
argocd cluster list
```

```yaml
# Deploy to multiple clusters
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp-multi-cluster
spec:
  generators:
  - clusters:
      selector:
        matchLabels:
          environment: production
  template:
    metadata:
      name: 'myapp-{{name}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/repo
        path: apps/myapp
      destination:
        server: '{{server}}'
        namespace: myapp
```

### Multi-Cluster with Flux

```bash
# Bootstrap multiple clusters
# Cluster 1 (Production)
flux bootstrap github \
  --context=production-cluster \
  --owner=org \
  --repository=fleet-infra \
  --path=clusters/production

# Cluster 2 (Staging)
flux bootstrap github \
  --context=staging-cluster \
  --owner=org \
  --repository=fleet-infra \
  --path=clusters/staging
```

```
fleet-infra/
├── clusters/
│   ├── production/
│   │   ├── flux-system/
│   │   ├── infrastructure.yaml
│   │   └── apps.yaml
│   └── staging/
│       ├── flux-system/
│       ├── infrastructure.yaml
│       └── apps.yaml
├── infrastructure/
│   ├── base/
│   ├── production/
│   └── staging/
└── apps/
    ├── base/
    ├── production/
    └── staging/
```

### Hub-Spoke Pattern

```yaml
# Management cluster deploys to workload clusters
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cluster-config
  namespace: argocd
spec:
  generators:
  - git:
      repoURL: https://github.com/org/clusters
      revision: HEAD
      files:
      - path: "clusters/*/config.json"

  template:
    metadata:
      name: '{{cluster.name}}-config'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/cluster-config
        path: '{{cluster.path}}'
      destination:
        name: '{{cluster.name}}'
        namespace: kube-system
```

```json
# clusters/prod-us-east/config.json
{
  "cluster": {
    "name": "prod-us-east",
    "path": "manifests/production",
    "region": "us-east-1"
  }
}
```

---

## Security & Best Practices

### Repository Structure

```
gitops-repo/
├── README.md
├── .gitignore
├── clusters/
│   ├── production/
│   └── staging/
├── infrastructure/
│   ├── base/
│   ├── namespaces/
│   ├── rbac/
│   ├── network-policies/
│   ├── ingress/
│   ├── cert-manager/
│   └── monitoring/
├── apps/
│   ├── base/
│   ├── overlays/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   └── team-a/
│       ├── frontend/
│       └── backend/
└── scripts/
    ├── validate.sh
    └── sync-check.sh
```

### Secret Management

#### Sealed Secrets

```bash
# Install Sealed Secrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# Install kubeseal CLI
# macOS
brew install kubeseal

# Linux
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/kubeseal-linux-amd64
chmod +x kubeseal-linux-amd64
sudo mv kubeseal-linux-amd64 /usr/local/bin/kubeseal

# Create secret
kubectl create secret generic mysecret \
  --from-literal=password=mypassword \
  --dry-run=client -o yaml > secret.yaml

# Seal the secret
kubeseal --format=yaml < secret.yaml > sealedsecret.yaml

# Commit sealed secret to Git
git add sealedsecret.yaml
git commit -m "Add sealed secret"
```

```yaml
# sealedsecret.yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: mysecret
  namespace: default
spec:
  encryptedData:
    password: AgBQ7H5K9...encrypted...
```

#### SOPS (Secrets OPerationS)

```bash
# Install SOPS
# macOS
brew install sops

# Install age for encryption
brew install age

# Generate age key
age-keygen -o key.txt

# Export public key
export SOPS_AGE_RECIPIENTS=$(cat key.txt | grep public | cut -d: -f2 | tr -d ' ')

# Create .sops.yaml
cat <<EOF > .sops.yaml
creation_rules:
  - path_regex: .*.yaml
    encrypted_regex: ^(data|stringData)$
    age: $SOPS_AGE_RECIPIENTS
EOF

# Encrypt secret
sops --encrypt secret.yaml > secret.enc.yaml

# Decrypt
sops --decrypt secret.enc.yaml

# Edit in place
sops secret.enc.yaml
```

```yaml
# Configure Flux to decrypt SOPS
# Create secret with age key
cat key.txt | kubectl create secret generic sops-age \
  --namespace=flux-system \
  --from-file=age.agekey=/dev/stdin

# Create Kustomization with decryption
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: myapp
  namespace: flux-system
spec:
  interval: 5m
  path: ./apps/myapp
  sourceRef:
    kind: GitRepository
    name: flux-system
  decryption:
    provider: sops
    secretRef:
      name: sops-age
```

#### External Secrets Operator

```bash
# Install External Secrets Operator
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
            name: external-secrets-sa

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
    creationPolicy: Owner
  data:
  - secretKey: password
    remoteRef:
      key: myapp/production
      property: password
```

### RBAC Best Practices

```yaml
# ArgoCD RBAC
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.default: role:readonly
  policy.csv: |
    # Developers can view and sync applications
    p, role:developer, applications, get, */*, allow
    p, role:developer, applications, sync, */*, allow
    p, role:developer, applications, override, */*, deny

    # DevOps can do everything except delete projects
    p, role:devops, applications, *, */*, allow
    p, role:devops, clusters, *, *, allow
    p, role:devops, repositories, *, *, allow
    p, role:devops, projects, delete, *, deny

    # Admin can do everything
    p, role:admin, *, *, *, allow

    # Group bindings
    g, developers, role:developer
    g, devops-team, role:devops
    g, platform-team, role:admin
```

### Validation & Testing

```bash
# validate.sh
#!/bin/bash

echo "Validating Kubernetes manifests..."

# Validate with kubectl
kubectl apply --dry-run=client -f apps/

# Validate with kubeval
kubeval apps/**/*.yaml

# Validate Kustomize builds
for env in dev staging production; do
  echo "Building $env..."
  kustomize build environments/$env > /dev/null
done

# Validate ArgoCD applications
argocd app list -o yaml | kubectl apply --dry-run=client -f -

# Check for secrets
echo "Checking for plain secrets..."
grep -r "kind: Secret" apps/ && echo "ERROR: Plain secrets found!" && exit 1

echo "Validation complete!"
```

```yaml
# Pre-commit hook configuration
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files

  - repo: https://github.com/instrumenta/kubeval
    rev: v0.16.1
    hooks:
      - id: kubeval
        files: \.yaml$
```

---

## Troubleshooting

### ArgoCD Issues

```bash
# Application not syncing
argocd app get myapp
argocd app diff myapp
argocd app sync myapp --force
argocd app sync myapp --prune

# Check controller logs
kubectl logs -n argocd deployment/argocd-application-controller

# Check server logs
kubectl logs -n argocd deployment/argocd-server

# Check repo server logs
kubectl logs -n argocd deployment/argocd-repo-server

# Hard refresh
argocd app get myapp --hard-refresh

# Restart components
kubectl rollout restart -n argocd deployment/argocd-application-controller
kubectl rollout restart -n argocd deployment/argocd-server
kubectl rollout restart -n argocd deployment/argocd-repo-server

# Fix stuck application
kubectl patch app myapp -n argocd --type=merge -p '{"metadata":{"finalizers":null}}'
```

### Flux Issues

```bash
# Check Flux status
flux check

# Reconcile stuck resource
flux reconcile source git myapp
flux reconcile kustomization myapp --with-source

# Check events
flux events --for Kustomization/myapp

# Suspend and resume
flux suspend kustomization myapp
flux resume kustomization myapp

# Check logs
flux logs --all-namespaces --follow
kubectl logs -n flux-system deployment/source-controller
kubectl logs -n flux-system deployment/kustomize-controller
kubectl logs -n flux-system deployment/helm-controller

# Debug Kustomization
flux tree kustomization myapp
flux get kustomizations myapp
kubectl describe kustomization myapp -n flux-system

# Uninstall and reinstall
flux uninstall
flux bootstrap github ...
```

### Common Issues

#### Image Pull Errors

```yaml
# Add image pull secret
apiVersion: v1
kind: Secret
metadata:
  name: regcred
  namespace: default
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-config>

---
# Reference in deployment
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      imagePullSecrets:
      - name: regcred
```

#### Git Authentication

```bash
# Create SSH key
ssh-keygen -t ed25519 -C "gitops@example.com" -f gitops-key

# Add to Kubernetes
kubectl create secret generic git-credentials \
  --from-file=identity=gitops-key \
  --from-file=known_hosts=~/.ssh/known_hosts \
  -n argocd

# For HTTPS with token
kubectl create secret generic git-credentials \
  --from-literal=username=git \
  --from-literal=password=<token> \
  -n argocd
```

#### Sync Failures

```yaml
# Ignore differences
apiVersion: argoproj.io/v1alpha1
kind: Application
spec:
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas
  - group: apps
    kind: StatefulSet
    jsonPointers:
    - /spec/volumeClaimTemplates
```

---

## Advanced Patterns

### Progressive Delivery with Argo Rollouts

```bash
# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Install kubectl plugin
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x kubectl-argo-rollouts-linux-amd64
sudo mv kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
```

```yaml
# Canary deployment
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 10
  strategy:
    canary:
      steps:
      - setWeight: 10
      - pause: {duration: 1m}
      - setWeight: 20
      - pause: {duration: 1m}
      - setWeight: 50
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
      - name: app
        image: myapp:v2.0.0
```

```yaml
# Blue-Green deployment
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    blueGreen:
      activeService: myapp-active
      previewService: myapp-preview
      autoPromotionEnabled: false
      prePromotionAnalysis:
        templates:
        - templateName: success-rate
      postPromotionAnalysis:
        templates:
        - templateName: error-rate
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:v2.0.0
```

```bash
# Rollout commands
kubectl argo rollouts list rollouts
kubectl argo rollouts get rollout myapp
kubectl argo rollouts status myapp
kubectl argo rollouts promote myapp
kubectl argo rollouts abort myapp
kubectl argo rollouts retry myapp
kubectl argo rollouts undo myapp
```

### Notification Systems

```yaml
# ArgoCD notifications for Slack, Teams, Email
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
data:
  service.slack: |
    token: $slack-token

  service.webhook.teams: |
    url: $teams-webhook-url

  service.email.gmail: |
    username: $email-username
    password: $email-password
    host: smtp.gmail.com
    port: 587

  template.app-sync-succeeded: |
    message: |
      Application {{.app.metadata.name}} sync succeeded
    email:
      subject: Application {{.app.metadata.name}} sync succeeded
    slack:
      attachments: |
        [{
          "title": "{{ .app.metadata.name}}",
          "color": "good"
        }]

  trigger.on-sync-succeeded: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [app-sync-succeeded]
```

### Config Management Plugins

```yaml
# ArgoCD ConfigManagementPlugin
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
data:
  configManagementPlugins: |
    - name: helmfile
      init:
        command: ["helmfile", "repos"]
      generate:
        command: ["helmfile", "template"]
```

### Monorepo vs Polyrepo

**Monorepo Structure:**
```
gitops-monorepo/
├── apps/
│   ├── team-a/
│   ├── team-b/
│   └── shared/
├── infrastructure/
├── clusters/
└── policies/
```

**Polyrepo Structure:**
```
gitops-infrastructure/      (separate repo)
gitops-team-a-apps/        (separate repo)
gitops-team-b-apps/        (separate repo)
gitops-policies/           (separate repo)
```

---

## Conclusion

GitOps provides a declarative, Git-centric approach to continuous deployment with benefits including:

- **Reliability**: Git history provides audit trail and rollback capability
- **Consistency**: Declarative configs ensure reproducible deployments
- **Velocity**: Automated sync enables rapid deployment
- **Security**: Git-based access control and secret management
- **Collaboration**: Standard Git workflows for infrastructure changes

### Key Takeaways

1. **Choose the Right Tool**: ArgoCD for UI/visibility, Flux for lightweight/native
2. **Structure Repos Properly**: Separate environments, use overlays
3. **Manage Secrets Securely**: Never commit plain secrets, use Sealed Secrets/SOPS
4. **Implement Progressive Delivery**: Use Argo Rollouts for safe deployments
5. **Monitor and Alert**: Set up notifications for sync failures
6. **Test Changes**: Validate manifests before merging
7. **Document Workflows**: Clear promotion and rollback procedures

### Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Flux Documentation](https://fluxcd.io/docs/)
- [GitOps Working Group](https://github.com/gitops-working-group)
- [OpenGitOps Principles](https://opengitops.dev/)
- [CNCF GitOps](https://www.cncf.io/blog/2021/08/11/gitops/)

---

**GitOps is not just a tool, it's a methodology for modern cloud-native deployments.**
