# Helm Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Helm Basics](#helm-basics)
4. [Chart Structure](#chart-structure)
5. [Chart Management](#chart-management)
6. [Repositories](#repositories)
7. [Templating](#templating)
8. [Values and Configuration](#values-and-configuration)
9. [Release Management](#release-management)
10. [Helm Hooks](#helm-hooks)
11. [Dependencies](#dependencies)
12. [Chart Testing](#chart-testing)
13. [Security](#security)
14. [Advanced Topics](#advanced-topics)
15. [Troubleshooting](#troubleshooting)
16. [Best Practices](#best-practices)
17. [Practical Examples](#practical-examples)

---

## Introduction

### What is Helm?

Helm is the package manager for Kubernetes, often referred to as "apt/yum for Kubernetes". It simplifies the deployment and management of Kubernetes applications.

### Key Concepts

- **Chart**: A Helm package containing Kubernetes resource definitions
- **Repository**: A place where charts are stored and shared
- **Release**: An instance of a chart running in a Kubernetes cluster
- **Values**: Configuration parameters for customizing chart deployments
- **Template**: Go template files that generate Kubernetes manifests

### Helm Architecture (v3)

```
┌─────────────┐
│   Helm CLI  │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│  Kubernetes API │
└─────────────────┘
       │
       ▼
┌─────────────────┐
│  Release Data   │
│  (Secrets/CM)   │
└─────────────────┘
```

**Note**: Helm v3 removed Tiller (server-side component), making it more secure.

---

## Installation

### Linux/macOS

```bash
# Using script
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Using Homebrew (macOS)
brew install helm

# Using Snap (Linux)
sudo snap install helm --classic

# Manual installation
curl -fsSL https://get.helm.sh/helm-v3.13.0-linux-amd64.tar.gz -o helm.tar.gz
tar -zxvf helm.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
```

### Windows

```powershell
# Using Chocolatey
choco install kubernetes-helm

# Using Scoop
scoop install helm

# Using winget
winget install Helm.Helm
```

### Verify Installation

```bash
helm version
helm version --short

# Output:
# v3.13.0+g8084d88
```

### Initialize Helm

```bash
# Add stable repo
helm repo add stable https://charts.helm.sh/stable

# Add bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update repositories
helm repo update
```

---

## Helm Basics

### Essential Commands

```bash
# Search for charts
helm search hub nginx                    # Search Artifact Hub
helm search repo nginx                   # Search local repos

# Show chart information
helm show chart bitnami/nginx
helm show values bitnami/nginx
helm show readme bitnami/nginx
helm show all bitnami/nginx

# Install a chart
helm install my-nginx bitnami/nginx
helm install my-nginx bitnami/nginx --namespace web --create-namespace

# List releases
helm list
helm list -A                             # All namespaces
helm list -n web                         # Specific namespace

# Get release status
helm status my-nginx
helm status my-nginx -n web

# Uninstall a release
helm uninstall my-nginx
helm uninstall my-nginx -n web --keep-history
```

### Quick Start Example

```bash
# 1. Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# 2. Search for chart
helm search repo wordpress

# 3. Install WordPress
helm install my-wordpress bitnami/wordpress \
  --set wordpressUsername=admin \
  --set wordpressPassword=password \
  --set mariadb.auth.rootPassword=secretpassword

# 4. Get release info
helm list
helm status my-wordpress

# 5. Access WordPress
kubectl get svc my-wordpress

# 6. Uninstall
helm uninstall my-wordpress
```

---

## Chart Structure

### Directory Layout

```
mychart/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Default configuration values
├── charts/                 # Dependent charts
├── templates/              # Kubernetes manifest templates
│   ├── NOTES.txt          # Post-install notes
│   ├── _helpers.tpl       # Template helpers
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── hpa.yaml
│   ├── serviceaccount.yaml
│   └── tests/
│       └── test-connection.yaml
├── .helmignore            # Files to ignore
└── README.md              # Chart documentation
```

### Chart.yaml

```yaml
apiVersion: v2
name: mychart
description: A Helm chart for my application
type: application           # or 'library'
version: 0.1.0             # Chart version (SemVer)
appVersion: "1.16.0"       # Application version

keywords:
  - web
  - nginx
  - kubernetes

home: https://example.com
sources:
  - https://github.com/example/mychart

maintainers:
  - name: John Doe
    email: john@example.com
    url: https://example.com

icon: https://example.com/icon.png

dependencies:
  - name: redis
    version: "17.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
    tags:
      - cache
  - name: postgresql
    version: "12.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled

annotations:
  category: Infrastructure
  licenses: Apache-2.0
```

### values.yaml

```yaml
# Default values for mychart

replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: ""  # Overrides the image tag (default: Chart.appVersion)

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  annotations: {}
  name: ""

podAnnotations: {}

podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000

securityContext:
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: "nginx"
  annotations: {}
    # cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}

# Application-specific values
config:
  logLevel: info
  feature1Enabled: true

database:
  host: postgresql
  port: 5432
  name: myapp
  username: myapp
  # password: set via secret

redis:
  enabled: true
  auth:
    enabled: false

postgresql:
  enabled: true
  auth:
    username: myapp
    password: changeme
    database: myapp
```

### .helmignore

```
# Patterns to ignore when building packages
.git/
.gitignore
.bzr/
.bzrignore
.hg/
.hgignore
.svn/
*.swp
*.bak
*.tmp
*.orig
*~
.DS_Store

# Development files
.vscode/
.idea/
*.iml

# CI/CD files
.gitlab-ci.yml
.github/
Jenkinsfile

# Documentation
docs/
*.md
!README.md
```

---

## Chart Management

### Creating Charts

```bash
# Create new chart
helm create mychart

# Validate chart
helm lint mychart

# Package chart
helm package mychart
# Output: mychart-0.1.0.tgz

# Package with specific version
helm package mychart --version 1.0.0 --app-version 2.0.0

# Create chart with dependencies
cd mychart
helm dependency update
helm dependency build
helm dependency list
```

### Installing Charts

```bash
# Install from repository
helm install myrelease bitnami/nginx

# Install from local chart
helm install myrelease ./mychart

# Install from packaged chart
helm install myrelease mychart-0.1.0.tgz

# Install with custom values
helm install myrelease ./mychart -f custom-values.yaml

# Install with set values
helm install myrelease ./mychart \
  --set replicaCount=3 \
  --set image.tag=1.16.0

# Install with values from multiple files
helm install myrelease ./mychart \
  -f values-base.yaml \
  -f values-prod.yaml

# Dry run (test without installing)
helm install myrelease ./mychart --dry-run --debug

# Generate manifests without installing
helm template myrelease ./mychart

# Install with timeout
helm install myrelease ./mychart --timeout 10m

# Install and wait for resources
helm install myrelease ./mychart --wait --wait-for-jobs

# Install with custom release name
helm install --generate-name ./mychart
# Or with prefix
helm install --generate-name ./mychart --name-template "myapp-{{randAlpha 8}}"
```

### Upgrading Charts

```bash
# Upgrade release
helm upgrade myrelease ./mychart

# Upgrade with new values
helm upgrade myrelease ./mychart -f new-values.yaml

# Upgrade or install (install if not exists)
helm upgrade --install myrelease ./mychart

# Upgrade with reuse of values
helm upgrade myrelease ./mychart --reuse-values

# Upgrade and reset values
helm upgrade myrelease ./mychart --reset-values

# Upgrade specific values only
helm upgrade myrelease ./mychart --set image.tag=1.17.0

# Upgrade with cleanup on fail
helm upgrade myrelease ./mychart --cleanup-on-fail

# Force upgrade (replace resources)
helm upgrade myrelease ./mychart --force

# Atomic upgrade (rollback on failure)
helm upgrade myrelease ./mychart --atomic

# Upgrade with wait
helm upgrade myrelease ./mychart --wait --timeout 5m
```

### Rolling Back

```bash
# View release history
helm history myrelease

# Rollback to previous revision
helm rollback myrelease

# Rollback to specific revision
helm rollback myrelease 2

# Rollback with cleanup
helm rollback myrelease 2 --cleanup-on-fail

# Rollback and wait
helm rollback myrelease --wait
```

### Uninstalling Charts

```bash
# Uninstall release
helm uninstall myrelease

# Uninstall and keep history
helm uninstall myrelease --keep-history

# Uninstall with timeout
helm uninstall myrelease --timeout 5m

# View deleted releases
helm list --uninstalled
helm list -a  # All releases
```

---

## Repositories

### Managing Repositories

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable
helm repo add jetstack https://charts.jetstack.io

# List repositories
helm repo list

# Update repositories
helm repo update

# Remove repository
helm repo remove stable

# Search repositories
helm search repo nginx
helm search repo nginx --versions
helm search repo nginx --version "~9"

# Show repository index
helm repo index .
```

### Popular Helm Repositories

```bash
# Bitnami
helm repo add bitnami https://charts.bitnami.com/bitnami

# Prometheus Community
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Grafana
helm repo add grafana https://grafana.github.io/helm-charts

# Ingress NGINX
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Jetstack (cert-manager)
helm repo add jetstack https://charts.jetstack.io

# ArgoCD
helm repo add argo https://argoproj.github.io/argo-helm

# Elastic
helm repo add elastic https://helm.elastic.co

# Harbor
helm repo add harbor https://helm.goharbor.io

# Hashicorp
helm repo add hashicorp https://helm.releases.hashicorp.com

# Kubernetes Dashboard
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
```

### Creating a Chart Repository

```bash
# Using GitHub Pages
# 1. Create charts directory
mkdir -p charts
helm package mychart -d charts/

# 2. Generate index
helm repo index charts/ --url https://username.github.io/helm-charts/

# 3. Commit and push
git add charts/
git commit -m "Add mychart"
git push

# 4. Add to Helm
helm repo add myrepo https://username.github.io/helm-charts/
```

### Using OCI Registries (Helm v3.8+)

```bash
# Login to registry
helm registry login registry-1.docker.io -u username

# Push chart to OCI registry
helm push mychart-0.1.0.tgz oci://registry-1.docker.io/username

# Pull chart from OCI registry
helm pull oci://registry-1.docker.io/username/mychart --version 0.1.0

# Install from OCI registry
helm install myrelease oci://registry-1.docker.io/username/mychart --version 0.1.0

# Logout
helm registry logout registry-1.docker.io
```

---

## Templating

### Template Basics

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
        env:
        - name: LOG_LEVEL
          value: {{ .Values.config.logLevel | quote }}
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

### Built-in Objects

```yaml
# Chart metadata
{{ .Chart.Name }}
{{ .Chart.Version }}
{{ .Chart.AppVersion }}

# Release information
{{ .Release.Name }}
{{ .Release.Namespace }}
{{ .Release.Service }}
{{ .Release.IsUpgrade }}
{{ .Release.IsInstall }}

# Kubernetes capabilities
{{ .Capabilities.KubeVersion }}
{{ .Capabilities.APIVersions.Has "apps/v1" }}

# Template metadata
{{ .Template.Name }}
{{ .Template.BasePath }}

# Files
{{ .Files.Get "config.ini" }}
{{ .Files.Glob "*.yaml" }}
```

### Template Functions

```yaml
# String functions
{{ .Values.name | upper }}
{{ .Values.name | lower }}
{{ .Values.name | title }}
{{ .Values.name | quote }}
{{ .Values.name | squote }}
{{ .Values.name | trim }}
{{ .Values.name | trimPrefix "prefix-" }}
{{ .Values.name | replace "old" "new" }}
{{ printf "%s-%s" .Release.Name .Chart.Name }}

# Default values
{{ .Values.name | default "default-name" }}
{{ .Values.name | required "name is required" }}

# Type conversion
{{ .Values.port | int }}
{{ .Values.enabled | toString }}
{{ .Values.list | toJson }}
{{ .Values.list | toYaml }}
{{ .Values.config | toPrettyJson }}

# Logic functions
{{ if .Values.enabled }}enabled{{ end }}
{{ if eq .Values.env "production" }}prod{{ else }}dev{{ end }}
{{ if and .Values.enabled .Values.feature }}both enabled{{ end }}
{{ if or .Values.enabled .Values.feature }}at least one enabled{{ end }}
{{ if not .Values.disabled }}enabled{{ end }}

# List functions
{{ .Values.list | first }}
{{ .Values.list | last }}
{{ .Values.list | rest }}
{{ .Values.list | initial }}
{{ .Values.list | append "item" }}
{{ .Values.list | prepend "item" }}
{{ .Values.list | has "item" }}
{{ .Values.list | without "item" }}
{{ list "a" "b" "c" | join "," }}

# Dict functions
{{ .Values.dict | keys }}
{{ .Values.dict | values }}
{{ pluck "key" .Values.dict }}
{{ merge .Values.dict1 .Values.dict2 }}
{{ dict "key1" "val1" "key2" "val2" }}

# Encoding
{{ .Values.password | b64enc }}
{{ .Values.encoded | b64dec }}
{{ .Values.data | sha256sum }}

# Date functions
{{ now | date "2006-01-02" }}
{{ now | unixEpoch }}

# Random
{{ randAlphaNum 10 }}
{{ randAlpha 8 }}
{{ randNumeric 6 }}
{{ uuidv4 }}

# Regex
{{ regexMatch "^[a-z]+$" .Values.name }}
{{ regexReplaceAll "pattern" .Values.text "replacement" }}
```

### Flow Control

```yaml
# If/Else
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "mychart.fullname" . }}
{{- end }}

# If/Else If/Else
{{- if eq .Values.service.type "NodePort" }}
  nodePort: {{ .Values.service.nodePort }}
{{- else if eq .Values.service.type "LoadBalancer" }}
  loadBalancerIP: {{ .Values.service.loadBalancerIP }}
{{- else }}
  # ClusterIP - no additional config
{{- end }}

# With (scope)
{{- with .Values.nodeSelector }}
nodeSelector:
  {{- toYaml . | nindent 2 }}
{{- end }}

# Range (loops)
env:
{{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value | quote }}
{{- end }}

# Range with index
{{- range $index, $value := .Values.items }}
- item{{ $index }}: {{ $value }}
{{- end }}

# Range with key-value
{{- range $key, $value := .Values.config }}
- name: {{ $key }}
  value: {{ $value | quote }}
{{- end }}

# Variables
{{- $relname := .Release.Name -}}
{{- $chartname := .Chart.Name -}}
name: {{ $relname }}-{{ $chartname }}
```

### Named Templates (_helpers.tpl)

```yaml
# templates/_helpers.tpl

{{/*
Expand the name of the chart.
*/}}
{{- define "mychart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "mychart.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "mychart.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "mychart.labels" -}}
helm.sh/chart: {{ include "mychart.chart" . }}
{{ include "mychart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "mychart.selectorLabels" -}}
app.kubernetes.io/name: {{ include "mychart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Create the name of the service account to use
*/}}
{{- define "mychart.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "mychart.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}

# Usage in templates:
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
```

### Whitespace Control

```yaml
# Remove whitespace before
{{- if .Values.enabled }}

# Remove whitespace after
{{ .Values.name -}}

# Remove both
{{- .Values.name -}}

# Example
env:
  {{- range .Values.env }}
  - name: {{ .name }}
    value: {{ .value | quote }}
  {{- end }}
```

---

## Values and Configuration

### Default Values Hierarchy

```
1. Chart's values.yaml (lowest priority)
2. Parent chart's values.yaml
3. User-supplied values file (-f flag)
4. Individual parameters (--set flag) (highest priority)
```

### Multiple Values Files

```bash
# values-base.yaml
replicaCount: 1
image:
  repository: nginx
  tag: "1.16"

# values-dev.yaml
ingress:
  enabled: false
resources:
  limits:
    memory: 256Mi

# values-prod.yaml
replicaCount: 3
ingress:
  enabled: true
  hosts:
    - host: app.example.com
resources:
  limits:
    memory: 512Mi
  requests:
    memory: 256Mi

# Install with merged values
helm install myapp ./mychart \
  -f values-base.yaml \
  -f values-prod.yaml
```

### Using --set

```bash
# Simple value
helm install myapp ./mychart --set replicaCount=3

# Nested value
helm install myapp ./mychart --set image.tag=1.17.0

# Multiple values
helm install myapp ./mychart \
  --set replicaCount=3 \
  --set image.tag=1.17.0 \
  --set service.type=LoadBalancer

# Array values
helm install myapp ./mychart \
  --set env[0].name=LOG_LEVEL \
  --set env[0].value=debug

# Complex nested values
helm install myapp ./mychart \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=app.example.com \
  --set ingress.hosts[0].paths[0].path=/

# Using --set-string (force string)
helm install myapp ./mychart --set-string nodeSelector."kubernetes\.io/role"=master

# Using --set-file
helm install myapp ./mychart --set-file config=./config.yaml
```

### Getting Values

```bash
# Get all values for a release
helm get values myrelease

# Get all values including defaults
helm get values myrelease --all

# Get values in JSON
helm get values myrelease -o json

# Get specific chart's default values
helm show values bitnami/nginx
```

### Environment-Specific Values

```yaml
# values.yaml (base)
environment: development
replicaCount: 1

# values-dev.yaml
environment: development
replicaCount: 1
ingress:
  enabled: false

# values-staging.yaml
environment: staging
replicaCount: 2
ingress:
  enabled: true
  hosts:
    - staging.example.com

# values-prod.yaml
environment: production
replicaCount: 3
ingress:
  enabled: true
  hosts:
    - app.example.com
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10

# Deploy to different environments
helm install myapp-dev ./mychart -f values-dev.yaml -n dev
helm install myapp-staging ./mychart -f values-staging.yaml -n staging
helm install myapp-prod ./mychart -f values-prod.yaml -n production
```

### Schema Validation (values.schema.json)

```json
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["replicaCount", "image"],
  "properties": {
    "replicaCount": {
      "type": "integer",
      "minimum": 1,
      "maximum": 10,
      "description": "Number of replicas"
    },
    "image": {
      "type": "object",
      "required": ["repository", "tag"],
      "properties": {
        "repository": {
          "type": "string",
          "pattern": "^[a-z0-9-./]+$"
        },
        "tag": {
          "type": "string"
        },
        "pullPolicy": {
          "type": "string",
          "enum": ["Always", "IfNotPresent", "Never"]
        }
      }
    },
    "service": {
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "enum": ["ClusterIP", "NodePort", "LoadBalancer"]
        },
        "port": {
          "type": "integer",
          "minimum": 1,
          "maximum": 65535
        }
      }
    }
  }
}
```

---

## Release Management

### Listing Releases

```bash
# List releases in current namespace
helm list
helm ls

# All namespaces
helm list -A
helm list --all-namespaces

# Specific namespace
helm list -n production

# Filter by status
helm list --deployed
helm list --failed
helm list --pending
helm list --superseded
helm list --uninstalled

# Show all releases
helm list -a

# Output formats
helm list -o json
helm list -o yaml
helm list -o table

# With specific columns
helm list --max 10
helm list --offset 5
```

### Release Information

```bash
# Get release status
helm status myrelease
helm status myrelease -n production

# Get release manifest
helm get manifest myrelease

# Get release values
helm get values myrelease
helm get values myrelease --all

# Get release hooks
helm get hooks myrelease

# Get release notes
helm get notes myrelease

# Get all release information
helm get all myrelease
```

### Release History

```bash
# View history
helm history myrelease

# Detailed history
helm history myrelease --max 100

# Output format
helm history myrelease -o json

# Example output:
# REVISION  UPDATED                   STATUS      CHART         APP VERSION  DESCRIPTION
# 1         Mon Oct 2 15:00:00 2023   superseded  mychart-0.1.0 1.16.0       Install complete
# 2         Mon Oct 2 16:00:00 2023   superseded  mychart-0.1.0 1.16.0       Upgrade complete
# 3         Mon Oct 2 17:00:00 2023   deployed    mychart-0.2.0 1.17.0       Upgrade complete
```

### Upgrading Releases

```bash
# Basic upgrade
helm upgrade myrelease ./mychart

# Upgrade with new version
helm upgrade myrelease bitnami/nginx --version 13.2.0

# Upgrade with values
helm upgrade myrelease ./mychart -f values-prod.yaml

# Upgrade or install
helm upgrade --install myrelease ./mychart

# Atomic upgrade (rollback on fail)
helm upgrade myrelease ./mychart --atomic

# Wait for resources to be ready
helm upgrade myrelease ./mychart --wait --timeout 10m

# Recreate pods
helm upgrade myrelease ./mychart --recreate-pods

# Force resource update
helm upgrade myrelease ./mychart --force

# Dry run
helm upgrade myrelease ./mychart --dry-run --debug

# Reuse previous values
helm upgrade myrelease ./mychart --reuse-values

# Reset all values to chart defaults
helm upgrade myrelease ./mychart --reset-values
```

### Rolling Back

```bash
# Rollback to previous revision
helm rollback myrelease

# Rollback to specific revision
helm rollback myrelease 2

# Rollback with wait
helm rollback myrelease --wait --timeout 5m

# Rollback and cleanup on fail
helm rollback myrelease --cleanup-on-fail

# Force recreate resources
helm rollback myrelease --force

# Dry run
helm rollback myrelease --dry-run
```

### Testing Releases

```bash
# Run tests
helm test myrelease

# Run tests with logs
helm test myrelease --logs

# Cleanup test pods after completion
helm test myrelease --cleanup

# Example test (templates/tests/test-connection.yaml):
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "mychart.fullname" . }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ include "mychart.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

---

## Helm Hooks

### Hook Types

```yaml
# Available hooks:
# pre-install    - Before resources are installed
# post-install   - After all resources are installed
# pre-delete     - Before any resources are deleted
# post-delete    - After all resources are deleted
# pre-upgrade    - Before resources are upgraded
# post-upgrade   - After all resources are upgraded
# pre-rollback   - Before resources are rolled back
# post-rollback  - After all resources are rolled back
# test           - When helm test is invoked
```

### Hook Example - Database Migration

```yaml
# templates/hooks/db-migration.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-db-migration"
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
  annotations:
    # Hook annotations
    "helm.sh/hook": pre-upgrade,pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    metadata:
      name: "{{ .Release.Name }}-db-migration"
    spec:
      restartPolicy: Never
      containers:
      - name: db-migration
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        command: ["/scripts/migrate.sh"]
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: {{ include "mychart.fullname" . }}
              key: database-url
```

### Hook Weight and Execution Order

```yaml
# Hook weight determines execution order (lower runs first)

# 1. Pre-upgrade hook - create backup (weight: -10)
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "-10"
    "helm.sh/hook-delete-policy": before-hook-creation

# 2. Pre-upgrade hook - run migration (weight: -5)
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation

# 3. Post-upgrade hook - verify (weight: 0)
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": post-upgrade
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": hook-succeeded

# 4. Post-upgrade hook - notify (weight: 5)
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": post-upgrade
    "helm.sh/hook-weight": "5"
    "helm.sh/hook-delete-policy": hook-succeeded
```

### Hook Delete Policies

```yaml
# Delete policies:
# before-hook-creation - Delete previous hook before new one (default)
# hook-succeeded       - Delete after successful execution
# hook-failed          - Delete if hook failed

# Keep hook on failure for debugging
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-delete-policy": hook-succeeded

# Always delete before creating new
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": post-upgrade
    "helm.sh/hook-delete-policy": before-hook-creation
```

### Common Hook Patterns

```yaml
# Pre-install: Initialize secrets
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-10"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: init-secrets
        image: alpine
        command: ["/bin/sh", "-c"]
        args:
          - |
            # Generate secrets and store in Kubernetes
            kubectl create secret generic app-secrets \
              --from-literal=api-key=$(openssl rand -hex 32)
      restartPolicy: Never

# Post-install: Run smoke tests
apiVersion: v1
kind: Pod
metadata:
  annotations:
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  containers:
  - name: smoke-test
    image: curlimages/curl
    command: ['sh', '-c']
    args:
      - |
        sleep 10
        curl -f http://{{ include "mychart.fullname" . }}:{{ .Values.service.port }}/health
  restartPolicy: Never

# Pre-delete: Backup data
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": pre-delete
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: backup
        image: backup-tool:latest
        command: ["/scripts/backup.sh"]
      restartPolicy: Never
```

---

## Dependencies

### Defining Dependencies

```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    version: "12.1.0"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
    tags:
      - database

  - name: redis
    version: "17.3.0"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
    tags:
      - cache

  - name: common
    version: "2.x.x"
    repository: https://charts.bitnami.com/bitnami

  - name: custom-lib
    version: "1.0.0"
    repository: "file://../custom-lib"
```

### Managing Dependencies

```bash
# Download dependencies
helm dependency update ./mychart
helm dependency build ./mychart

# List dependencies
helm dependency list ./mychart

# Output:
# NAME        VERSION  REPOSITORY                             STATUS
# postgresql  12.1.0   https://charts.bitnami.com/bitnami    ok
# redis       17.3.0   https://charts.bitnami.com/bitnami    ok

# View charts directory
ls mychart/charts/
# postgresql-12.1.0.tgz
# redis-17.3.0.tgz
```

### Conditional Dependencies

```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    condition: postgresql.enabled
  - name: mysql
    condition: mysql.enabled
  - name: redis
    tags:
      - cache
    condition: redis.enabled

# values.yaml
postgresql:
  enabled: true
  auth:
    password: changeme

mysql:
  enabled: false

redis:
  enabled: true

# Enable/disable via command line
helm install myapp ./mychart --set postgresql.enabled=false --set mysql.enabled=true
```

### Overriding Subchart Values

```yaml
# values.yaml - Override subchart values

# Method 1: Direct override
postgresql:
  auth:
    username: myapp
    password: secretpassword
    database: myappdb
  primary:
    persistence:
      size: 10Gi
  metrics:
    enabled: true

redis:
  auth:
    enabled: true
    password: redispass
  master:
    persistence:
      size: 5Gi
  replica:
    replicaCount: 2

# Method 2: Using global values
global:
  storageClass: "fast-ssd"
  postgresql:
    auth:
      postgresPassword: globalpass

# Install with overrides
helm install myapp ./mychart \
  --set postgresql.primary.persistence.size=20Gi \
  --set redis.replica.replicaCount=3
```

### Importing Values from Subcharts

```yaml
# Chart.yaml - Import specific values
dependencies:
  - name: postgresql
    version: "12.1.0"
    repository: https://charts.bitnami.com/bitnami
    import-values:
      - child: auth.password
        parent: dbPassword
      - child: auth.username
        parent: dbUsername

# Now you can use in templates:
env:
- name: DB_PASSWORD
  value: {{ .Values.dbPassword }}
- name: DB_USERNAME
  value: {{ .Values.dbUsername }}
```

### Using Library Charts

```yaml
# Library chart (type: library in Chart.yaml)
# common-lib/Chart.yaml
apiVersion: v2
name: common-lib
type: library
version: 1.0.0

# common-lib/templates/_helpers.tpl
{{- define "common-lib.labels" -}}
app.kubernetes.io/managed-by: {{ .Release.Service }}
app.kubernetes.io/instance: {{ .Release.Name }}
helm.sh/chart: {{ .Chart.Name }}-{{ .Chart.Version }}
{{- end }}

# Application chart depends on library
# mychart/Chart.yaml
dependencies:
  - name: common-lib
    version: "1.0.0"
    repository: "file://../common-lib"

# Use in templates
# mychart/templates/deployment.yaml
metadata:
  labels:
    {{- include "common-lib.labels" . | nindent 4 }}
```

---

## Chart Testing

### Unit Testing with helm lint

```bash
# Lint chart
helm lint ./mychart

# Lint with values
helm lint ./mychart -f values-prod.yaml

# Lint with strict mode
helm lint ./mychart --strict

# Lint with specific values
helm lint ./mychart --set replicaCount=3
```

### Template Testing

```bash
# Render templates locally
helm template myrelease ./mychart

# Render with values
helm template myrelease ./mychart -f values-prod.yaml

# Render specific templates
helm template myrelease ./mychart -s templates/deployment.yaml

# Show only specific resource
helm template myrelease ./mychart | kubectl apply --dry-run=client -f -

# Debug templates
helm template myrelease ./mychart --debug

# Validate generated manifests
helm template myrelease ./mychart | kubectl apply --dry-run=server -f -
```

### Integration Testing

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "mychart.fullname" . }}-test-connection"
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ include "mychart.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never

# templates/tests/test-api.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "mychart.fullname" . }}-test-api"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: test
    image: curlimages/curl:latest
    command: ['sh', '-c']
    args:
      - |
        curl -f http://{{ include "mychart.fullname" . }}:{{ .Values.service.port }}/api/health
        curl -f http://{{ include "mychart.fullname" . }}:{{ .Values.service.port }}/api/ready
  restartPolicy: Never

# Run tests
helm test myrelease
helm test myrelease --logs
```

### Chart Testing with ct (chart-testing)

```bash
# Install chart-testing
pip install yamllint yamale
curl -sSL https://raw.githubusercontent.com/helm/chart-testing/main/etc/install.sh | bash

# Lint charts
ct lint --config ct.yaml

# Install and test charts
ct install --config ct.yaml

# ct.yaml configuration
chart-repos:
  - bitnami=https://charts.bitnami.com/bitnami

helm-extra-args: --timeout 600s
check-version-increment: true
debug: true

# GitHub Actions workflow
name: Lint and Test Charts

on: pull_request

jobs:
  lint-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Set up Helm
        uses: azure/setup-helm@v3
        with:
          version: v3.13.0

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Set up chart-testing
        uses: helm/chart-testing-action@v2

      - name: Run chart-testing (lint)
        run: ct lint --config ct.yaml

      - name: Create kind cluster
        uses: helm/kind-action@v1

      - name: Run chart-testing (install)
        run: ct install --config ct.yaml
```

### Snapshot Testing

```bash
# Using helm unittest plugin
helm plugin install https://github.com/helm-unittest/helm-unittest

# Create test file
# tests/deployment_test.yaml
suite: test deployment
templates:
  - deployment.yaml
tests:
  - it: should create deployment
    asserts:
      - isKind:
          of: Deployment
      - equal:
          path: metadata.name
          value: RELEASE-NAME-mychart

  - it: should set replicas
    set:
      replicaCount: 3
    asserts:
      - equal:
          path: spec.replicas
          value: 3

  - it: should use correct image
    set:
      image.repository: nginx
      image.tag: "1.19"
    asserts:
      - equal:
          path: spec.template.spec.containers[0].image
          value: nginx:1.19

# Run tests
helm unittest ./mychart

# Run with specific test
helm unittest -f 'tests/deployment_test.yaml' ./mychart
```

---

## Security

### RBAC for Helm

```yaml
# ServiceAccount for Helm
apiVersion: v1
kind: ServiceAccount
metadata:
  name: helm-user
  namespace: default

---
# Role for Helm operations
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: helm-role
  namespace: default
rules:
- apiGroups: ["", "apps", "batch", "extensions"]
  resources: ["*"]
  verbs: ["*"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: helm-rolebinding
  namespace: default
subjects:
- kind: ServiceAccount
  name: helm-user
  namespace: default
roleRef:
  kind: Role
  name: helm-role
  apiGroup: rbac.authorization.k8s.io

# Use ServiceAccount
kubectl config set-credentials helm-user --token=$(kubectl get secret $(kubectl get sa helm-user -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 -d)
```

### Signing Charts

```bash
# Generate GPG key
gpg --gen-key

# Package and sign chart
helm package --sign --key 'keyname' --keyring ~/.gnupg/secring.gpg mychart

# Verify signed chart
helm verify mychart-0.1.0.tgz

# Install with verification
helm install myrelease mychart-0.1.0.tgz --verify
```

### Provenance Files

```bash
# Create provenance file
helm package mychart --sign --key 'John Doe' --keyring ~/.gnupg/secring.gpg

# Generates:
# - mychart-0.1.0.tgz
# - mychart-0.1.0.tgz.prov

# Verify provenance
helm verify mychart-0.1.0.tgz --keyring ~/.gnupg/pubring.gpg
```

### Secrets Management

```yaml
# Method 1: Kubernetes Secrets
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "mychart.fullname" . }}
type: Opaque
data:
  database-password: {{ .Values.database.password | b64enc }}
  api-key: {{ .Values.apiKey | b64enc }}

# Method 2: External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: {{ include "mychart.fullname" . }}
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: {{ include "mychart.fullname" . }}-secrets
  data:
  - secretKey: database-password
    remoteRef:
      key: secret/data/myapp
      property: db-password

# Method 3: Sealed Secrets
# Install sealed-secrets controller first
# kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# Create sealed secret
kubectl create secret generic mysecret \
  --from-literal=password=mypassword \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > templates/sealedsecret.yaml

# Method 4: Helm Secrets Plugin
helm plugin install https://github.com/jkroepke/helm-secrets

# secrets.yaml (encrypted with SOPS)
database:
  password: ENC[AES256_GCM,data:xxxxx,type:str]

# Install with secrets
helm secrets install myrelease ./mychart -f secrets.yaml
```

### Security Best Practices

```yaml
# 1. Use specific image tags (not latest)
image:
  repository: nginx
  tag: "1.21.6"  # NOT "latest"

# 2. Define security contexts
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
  seccompProfile:
    type: RuntimeDefault

securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true

# 3. Set resource limits
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

# 4. Use NetworkPolicies
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: {{ include "mychart.fullname" . }}
spec:
  podSelector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432

# 5. Use PodDisruptionBudget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: {{ include "mychart.fullname" . }}
spec:
  minAvailable: 1
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
```

---

## Advanced Topics

### Helm Plugins

```bash
# List plugins
helm plugin list

# Install plugin
helm plugin install https://github.com/databus23/helm-diff

# Popular plugins:

# 1. helm-diff - Preview changes before upgrade
helm plugin install https://github.com/databus23/helm-diff
helm diff upgrade myrelease ./mychart -f values.yaml

# 2. helm-secrets - Manage secrets
helm plugin install https://github.com/jkroepke/helm-secrets
helm secrets install myrelease ./mychart -f secrets.yaml

# 3. helm-git - Install charts from git
helm plugin install https://github.com/aslafy-z/helm-git
helm install myrelease git+https://github.com/user/charts@mychart

# 4. helm-unittest - Unit test charts
helm plugin install https://github.com/helm-unittest/helm-unittest
helm unittest ./mychart

# 5. helm-docs - Generate documentation
helm plugin install https://github.com/norwoodj/helm-docs
helm-docs

# Update plugin
helm plugin update diff

# Uninstall plugin
helm plugin uninstall diff
```

### Helm with CI/CD

```yaml
# GitLab CI example
.helm_template: &helm_template
  image: alpine/helm:3.13.0
  before_script:
    - helm repo add bitnami https://charts.bitnami.com/bitnami
    - helm dependency update

lint:
  <<: *helm_template
  stage: test
  script:
    - helm lint ./mychart
    - helm template test ./mychart --debug

package:
  <<: *helm_template
  stage: build
  script:
    - helm package ./mychart
  artifacts:
    paths:
      - "*.tgz"

deploy_dev:
  <<: *helm_template
  stage: deploy
  environment:
    name: development
  script:
    - helm upgrade --install myapp ./mychart
        -f values-dev.yaml
        --namespace dev
        --create-namespace
        --wait
  only:
    - develop

deploy_prod:
  <<: *helm_template
  stage: deploy
  environment:
    name: production
  script:
    - helm upgrade --install myapp ./mychart
        -f values-prod.yaml
        --namespace production
        --atomic
        --wait
        --timeout 10m
  only:
    - main
  when: manual

# GitHub Actions example
name: Helm Chart CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lint-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Helm
        uses: azure/setup-helm@v3
        with:
          version: '3.13.0'

      - name: Lint
        run: helm lint ./mychart

      - name: Template
        run: helm template test ./mychart --debug

  deploy:
    needs: lint-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3

      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG }}

      - name: Deploy
        run: |
          helm upgrade --install myapp ./mychart \
            -f values-prod.yaml \
            --namespace production \
            --create-namespace \
            --atomic \
            --wait
```

### Helm Chart Repositories

```bash
# ChartMuseum (Chart repository server)
# Install
docker run -d \
  -p 8080:8080 \
  -e DEBUG=1 \
  -e STORAGE=local \
  -e STORAGE_LOCAL_ROOTDIR=/charts \
  -v $(pwd)/charts:/charts \
  ghcr.io/helm/chartmuseum:v0.16.0

# Add repo
helm repo add myrepo http://localhost:8080

# Push chart (with push plugin)
helm plugin install https://github.com/chartmuseum/helm-push
helm cm-push mychart-0.1.0.tgz myrepo

# Harbor (Enterprise registry)
# Add Harbor repo
helm repo add harbor https://harbor.example.com/chartrepo/myproject \
  --username admin \
  --password password

# Push to Harbor
helm push mychart-0.1.0.tgz oci://harbor.example.com/myproject

# Artifactory
helm repo add artifactory https://artifactory.example.com/artifactory/helm-local \
  --username user \
  --password password

# GitHub Pages
# 1. Create gh-pages branch
git checkout --orphan gh-pages
git rm -rf .

# 2. Package chart
helm package mychart

# 3. Create index
helm repo index . --url https://username.github.io/helm-charts/

# 4. Push
git add .
git commit -m "Initial chart repo"
git push origin gh-pages

# 5. Use repo
helm repo add myrepo https://username.github.io/helm-charts/
```

### Helm Operators

```yaml
# Using Helm with Operators
# Example: flux-helm-controller

apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: bitnami
  namespace: flux-system
spec:
  interval: 1h
  url: https://charts.bitnami.com/bitnami

---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: nginx
  namespace: default
spec:
  interval: 10m
  chart:
    spec:
      chart: nginx
      version: '13.x'
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
  values:
    replicaCount: 3
    service:
      type: LoadBalancer
  install:
    remediation:
      retries: 3
  upgrade:
    remediation:
      retries: 3
```

### Multi-Environment Management

```bash
# helmfile.yaml - Manage multiple releases
repositories:
  - name: bitnami
    url: https://charts.bitnami.com/bitnami
  - name: prometheus-community
    url: https://prometheus-community.github.io/helm-charts

releases:
  # Development
  - name: myapp-dev
    namespace: dev
    chart: ./mychart
    values:
      - values-base.yaml
      - values-dev.yaml
    set:
      - name: environment
        value: development

  # Staging
  - name: myapp-staging
    namespace: staging
    chart: ./mychart
    values:
      - values-base.yaml
      - values-staging.yaml
    set:
      - name: environment
        value: staging

  # Production
  - name: myapp-prod
    namespace: production
    chart: ./mychart
    values:
      - values-base.yaml
      - values-prod.yaml
    set:
      - name: environment
        value: production

# Install helmfile
brew install helmfile

# Sync all releases
helmfile sync

# Sync specific environment
helmfile -e production sync

# Diff changes
helmfile diff

# Apply changes
helmfile apply
```

---

## Troubleshooting

### Common Issues

```bash
# 1. "Release already exists" error
# Solution: Uninstall or use --replace flag
helm uninstall myrelease
# or
helm install myrelease ./mychart --replace

# 2. "Timed out waiting for condition"
# Solution: Increase timeout
helm install myrelease ./mychart --timeout 10m --wait

# 3. Template rendering errors
# Solution: Debug template
helm template myrelease ./mychart --debug
helm install myrelease ./mychart --dry-run --debug

# 4. Values not being applied
# Solution: Check values precedence
helm get values myrelease --all
helm upgrade myrelease ./mychart --reuse-values -f new-values.yaml

# 5. Chart dependencies not found
# Solution: Update dependencies
helm dependency update ./mychart
helm dependency build ./mychart

# 6. Permission denied errors
# Solution: Check RBAC
kubectl auth can-i create deployments --namespace default

# 7. Release stuck in pending state
# Solution: Force delete and reinstall
helm uninstall myrelease --no-hooks
kubectl delete secret -l owner=helm,name=myrelease

# 8. Hooks failing
# Solution: Check hook logs
kubectl logs -l 'helm.sh/hook'
helm get hooks myrelease
```

### Debugging Commands

```bash
# Show debug output
helm install myrelease ./mychart --debug --dry-run

# Render templates
helm template myrelease ./mychart --debug

# Validate chart
helm lint ./mychart --strict

# Get release manifest
helm get manifest myrelease

# Check release history
helm history myrelease

# Get all release info
helm get all myrelease

# Show computed values
helm get values myrelease --all

# Check hooks
helm get hooks myrelease

# View chart metadata
helm show chart ./mychart
helm show values ./mychart
```

### Release Recovery

```bash
# List all releases including failed
helm list -a

# Check release status
helm status myrelease

# View release history
helm history myrelease

# Rollback to working version
helm rollback myrelease 2

# Force delete stuck release
kubectl delete secret -n <namespace> -l name=<release-name>,owner=helm

# Manual cleanup
kubectl get secret -n <namespace> | grep <release-name>
kubectl delete secret sh.helm.release.v1.<release-name>.v1 -n <namespace>

# Reinstall after cleanup
helm install myrelease ./mychart
```

### Performance Issues

```bash
# 1. Large chart size
# Solution: Use .helmignore
echo "tests/" >> .helmignore
echo "docs/" >> .helmignore

# 2. Slow template rendering
# Solution: Optimize templates (avoid expensive functions in loops)
# Bad:
{{- range .Values.items }}
  {{- include "mychart.labels" . }}
{{- end }}

# Good:
{{- $labels := include "mychart.labels" . }}
{{- range .Values.items }}
  {{ $labels }}
{{- end }}

# 3. Too many resources
# Solution: Split into multiple charts or use subcharts

# 4. Large values files
# Solution: Use multiple smaller values files
helm install myrelease ./mychart \
  -f values-base.yaml \
  -f values-env.yaml \
  -f values-overrides.yaml
```

---

## Best Practices

### Chart Development

```yaml
# 1. Use semantic versioning
# Chart.yaml
version: 1.2.3  # MAJOR.MINOR.PATCH

# 2. Pin dependency versions
dependencies:
  - name: postgresql
    version: "12.1.0"  # NOT "~12.1" or "^12"
    repository: https://charts.bitnami.com/bitnami

# 3. Provide sensible defaults
# values.yaml
replicaCount: 1
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

# 4. Use labels consistently
metadata:
  labels:
    app.kubernetes.io/name: {{ include "mychart.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
    app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
    helm.sh/chart: {{ include "mychart.chart" . }}

# 5. Document values
# values.yaml
# -- Number of replicas
replicaCount: 1

# -- Image configuration
image:
  # -- Image repository
  repository: nginx
  # -- Image pull policy
  pullPolicy: IfNotPresent
  # -- Image tag (defaults to chart appVersion)
  tag: ""
```

### Template Best Practices

```yaml
# 1. Use named templates for common patterns
{{- define "mychart.labels" -}}
app.kubernetes.io/name: {{ include "mychart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

# 2. Validate required values
{{- if not .Values.image.repository }}
  {{- fail "image.repository is required" }}
{{- end }}

# 3. Use default function
image: {{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}

# 4. Handle optional sections properly
{{- with .Values.nodeSelector }}
nodeSelector:
  {{- toYaml . | nindent 2 }}
{{- end }}

# 5. Use proper indentation
{{- include "mychart.labels" . | nindent 4 }}

# 6. Add checksums for config changes
annotations:
  checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}

# 7. Quote string values
env:
- name: LOG_LEVEL
  value: {{ .Values.logLevel | quote }}
```

### Values Organization

```yaml
# Group related values
database:
  host: postgresql
  port: 5432
  name: myapp
  auth:
    username: user
    password: pass

# Use feature flags
features:
  metrics: true
  tracing: false
  profiling: false

# Environment-specific values
environments:
  dev:
    replicas: 1
    resources:
      limits:
        memory: 256Mi
  prod:
    replicas: 3
    resources:
      limits:
        memory: 1Gi
```

### Security Best Practices

```yaml
# 1. Don't hardcode secrets
# Bad:
env:
- name: DB_PASSWORD
  value: "mysecretpassword"

# Good:
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: {{ include "mychart.fullname" . }}
      key: db-password

# 2. Use security contexts
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000

securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true

# 3. Define resource limits
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

# 4. Use network policies
# Enable via values
networkPolicy:
  enabled: true
  policyTypes:
    - Ingress
    - Egress

# 5. Scan images
# Use specific tags
image:
  repository: nginx
  tag: "1.21.6"  # Not "latest"

# 6. Sign charts
helm package --sign --key 'keyname' mychart
```

### Production Checklist

```yaml
# Production deployment checklist:

# ✓ Resource limits defined
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

# ✓ Multiple replicas
replicaCount: 3

# ✓ Pod disruption budget
podDisruptionBudget:
  enabled: true
  minAvailable: 1

# ✓ Liveness/Readiness probes
livenessProbe:
  httpGet:
    path: /healthz
    port: http
  initialDelaySeconds: 30

readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 5

# ✓ Affinity rules
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchLabels:
            app.kubernetes.io/name: {{ include "mychart.name" . }}
        topologyKey: kubernetes.io/hostname

# ✓ HPA enabled
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

# ✓ Security context
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000

# ✓ Network policies
networkPolicy:
  enabled: true

# ✓ Monitoring enabled
metrics:
  enabled: true
  serviceMonitor:
    enabled: true

# ✓ Backup strategy
backup:
  enabled: true
  schedule: "0 2 * * *"
```

---

## Practical Examples

### Example 1: Simple Web Application

```yaml
# Chart.yaml
apiVersion: v2
name: webapp
description: Simple web application
type: application
version: 1.0.0
appVersion: "1.0"

# values.yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.21"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: webapp.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: webapp-tls
      hosts:
        - webapp.example.com

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 50m
    memory: 64Mi

# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "webapp.fullname" . }}
  labels:
    {{- include "webapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "webapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "webapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
        livenessProbe:
          httpGet:
            path: /
            port: http
        readinessProbe:
          httpGet:
            path: /
            port: http
        resources:
          {{- toYaml .Values.resources | nindent 12 }}

# Deploy
helm install webapp ./webapp -f values-prod.yaml
```

### Example 2: Microservices Application

```yaml
# Chart.yaml
apiVersion: v2
name: microservices-app
version: 1.0.0
dependencies:
  - name: frontend
    version: "1.0.0"
    repository: "file://./charts/frontend"
  - name: backend
    version: "1.0.0"
    repository: "file://./charts/backend"
  - name: postgresql
    version: "12.1.0"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
  - name: redis
    version: "17.3.0"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled

# values.yaml
frontend:
  replicaCount: 2
  image:
    repository: myregistry/frontend
    tag: "1.0.0"
  service:
    type: ClusterIP
    port: 3000
  ingress:
    enabled: true
    hosts:
      - host: app.example.com

backend:
  replicaCount: 3
  image:
    repository: myregistry/backend
    tag: "1.0.0"
  service:
    type: ClusterIP
    port: 8080
  env:
    LOG_LEVEL: info
    DATABASE_HOST: "{{ .Release.Name }}-postgresql"
    REDIS_HOST: "{{ .Release.Name }}-redis-master"

postgresql:
  enabled: true
  auth:
    username: appuser
    password: changeme
    database: appdb
  primary:
    persistence:
      size: 10Gi

redis:
  enabled: true
  architecture: replication
  auth:
    enabled: false
  master:
    persistence:
      size: 5Gi

# Deploy
helm dependency update ./microservices-app
helm install myapp ./microservices-app -f values-prod.yaml
```

### Example 3: Database Backup Job

```yaml
# templates/cronjob-backup.yaml
{{- if .Values.backup.enabled }}
apiVersion: batch/v1
kind: CronJob
metadata:
  name: {{ include "mychart.fullname" . }}-backup
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  schedule: {{ .Values.backup.schedule | quote }}
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            {{- include "mychart.selectorLabels" . | nindent 12 }}
            job: backup
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: postgres:14-alpine
            command:
            - /bin/sh
            - -c
            - |
              BACKUP_FILE="/backup/backup-$(date +%Y%m%d-%H%M%S).sql.gz"
              pg_dump -h {{ .Values.database.host }} \
                      -U {{ .Values.database.username }} \
                      -d {{ .Values.database.name }} | gzip > $BACKUP_FILE

              # Upload to S3
              aws s3 cp $BACKUP_FILE s3://{{ .Values.backup.s3Bucket }}/

              # Cleanup old backups
              find /backup -mtime +{{ .Values.backup.retentionDays }} -delete
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "mychart.fullname" . }}
                  key: database-password
            - name: AWS_ACCESS_KEY_ID
              valueFrom:
                secretKeyRef:
                  name: {{ include "mychart.fullname" . }}-backup
                  key: aws-access-key-id
            - name: AWS_SECRET_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: {{ include "mychart.fullname" . }}-backup
                  key: aws-secret-access-key
            volumeMounts:
            - name: backup
              mountPath: /backup
          volumes:
          - name: backup
            persistentVolumeClaim:
              claimName: {{ include "mychart.fullname" . }}-backup
{{- end }}

# values.yaml
backup:
  enabled: true
  schedule: "0 2 * * *"  # 2 AM daily
  retentionDays: 30
  s3Bucket: my-backups
```

### Example 4: Blue-Green Deployment

```yaml
# values-blue.yaml
version: blue
replicaCount: 3
image:
  tag: "1.0.0"
service:
  selector:
    version: blue

# values-green.yaml
version: green
replicaCount: 3
image:
  tag: "2.0.0"
service:
  selector:
    version: green

# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}-{{ .Values.version }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
    version: {{ .Values.version }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
      version: {{ .Values.version }}
  template:
    metadata:
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
        version: {{ .Values.version }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"

# templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "mychart.fullname" . }}
spec:
  selector:
    {{- include "mychart.selectorLabels" . | nindent 4 }}
    version: {{ .Values.service.selector.version }}
  ports:
  - port: 80
    targetPort: http

# Deploy both versions
helm install myapp-blue ./mychart -f values-blue.yaml
helm install myapp-green ./mychart -f values-green.yaml

# Switch traffic
kubectl patch svc myapp -p '{"spec":{"selector":{"version":"green"}}}'

# Cleanup old version
helm uninstall myapp-blue
```

### Example 5: Complete Production Chart

```bash
# Create comprehensive chart
helm create production-app

# File structure:
production-app/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
├── values-prod.yaml
├── charts/
├── templates/
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── serviceaccount.yaml
│   ├── hpa.yaml
│   ├── pdb.yaml
│   ├── networkpolicy.yaml
│   ├── servicemonitor.yaml
│   └── tests/
│       └── test-connection.yaml
└── README.md

# Deploy to environments
helm install myapp ./production-app -f values-dev.yaml -n dev
helm install myapp ./production-app -f values-staging.yaml -n staging
helm install myapp ./production-app -f values-prod.yaml -n production

# Upgrade production with canary
helm upgrade myapp ./production-app \
  -f values-prod.yaml \
  --set canary.enabled=true \
  --set canary.weight=10 \
  -n production

# Full production rollout
helm upgrade myapp ./production-app \
  -f values-prod.yaml \
  --set canary.enabled=false \
  --atomic \
  --wait \
  --timeout 10m \
  -n production
```

---

## Quick Reference

### Essential Commands Cheat Sheet

```bash
# Repository management
helm repo add <name> <url>
helm repo update
helm repo list

# Chart management
helm create <chart>
helm lint <chart>
helm package <chart>
helm dependency update <chart>

# Installation
helm install <release> <chart>
helm install <release> <chart> -f values.yaml
helm install <release> <chart> --set key=value

# Upgrades and rollbacks
helm upgrade <release> <chart>
helm upgrade --install <release> <chart>
helm rollback <release> <revision>

# Information
helm list
helm status <release>
helm get values <release>
helm get manifest <release>
helm history <release>

# Debugging
helm template <release> <chart>
helm install <release> <chart> --dry-run --debug
helm get all <release>

# Cleanup
helm uninstall <release>
helm uninstall <release> --keep-history
```

---

**End of Helm Complete Guide**

For more information, visit:
- Official Documentation: https://helm.sh/docs/
- Chart Hub: https://artifacthub.io/
- GitHub: https://github.com/helm/helm
