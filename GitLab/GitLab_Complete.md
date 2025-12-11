# GitLab Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Repository Management](#repository-management)
4. [GitLab CI/CD](#gitlab-cicd)
5. [GitLab Runner](#gitlab-runner)
6. [Pipeline Configuration](#pipeline-configuration)
7. [Advanced CI/CD Features](#advanced-cicd-features)
8. [Merge Requests](#merge-requests)
9. [GitLab Container Registry](#gitlab-container-registry)
10. [Security & Compliance](#security--compliance)
11. [GitLab Pages](#gitlab-pages)
12. [Package Registry](#package-registry)
13. [Auto DevOps](#auto-devops)
14. [GitLab API](#gitlab-api)
15. [GitLab Administration](#gitlab-administration)
16. [Troubleshooting](#troubleshooting)
17. [Best Practices](#best-practices)
18. [Practical Examples](#practical-examples)

---

## Introduction

### What is GitLab?

GitLab is a complete DevOps platform delivered as a single application. It provides:
- Git repository management
- Integrated CI/CD pipelines
- Container registry
- Security scanning
- Issue tracking
- Wiki and documentation
- Package management
- Kubernetes integration

### GitLab Versions

**GitLab.com (SaaS)**
- Free tier: 400 CI/CD minutes/month
- Premium: $29/user/month
- Ultimate: $99/user/month

**GitLab Self-Managed**
- Community Edition (CE): Free, open-source
- Enterprise Edition (EE): Premium features

### GitLab Architecture

```
GitLab Platform
├── Source Code Management (SCM)
│   ├── Git repositories
│   ├── Code review (Merge Requests)
│   └── Branch protection
├── CI/CD
│   ├── Pipeline automation
│   ├── GitLab Runners
│   └── Deployment environments
├── Security
│   ├── SAST, DAST
│   ├── Dependency scanning
│   └── Container scanning
├── Operations
│   ├── Monitoring
│   ├── Incident management
│   └── Feature flags
└── Package Management
    ├── Container Registry
    ├── NPM, Maven, PyPI
    └── Helm charts
```

---

## Getting Started

### Account Setup

```bash
# Sign up at https://gitlab.com

# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Copy public key
cat ~/.ssh/id_ed25519.pub

# Add to GitLab:
# Profile → Settings → SSH Keys → Add new key

# Test SSH connection
ssh -T git@gitlab.com
```

### Creating Your First Project

```bash
# Via GitLab UI:
# Projects → New project → Create blank project
# - Project name: my-project
# - Visibility: Private/Internal/Public
# - Initialize with README

# Clone repository
git clone git@gitlab.com:username/my-project.git
cd my-project

# Create files
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"
git push -u origin main

# Or push existing repository
git remote add origin git@gitlab.com:username/my-project.git
git push -u origin main
```

### GitLab CLI (glab)

```bash
# Install glab
# macOS
brew install glab

# Linux
sudo snap install glab

# Windows
scoop install glab

# Authenticate
glab auth login

# Common commands
glab repo list
glab repo create myproject
glab mr list
glab mr create
glab ci view
glab ci trace
```

### Project Structure

```
GitLab Project
├── Repository
│   ├── Branches
│   ├── Tags
│   ├── Commits
│   └── Files
├── Issues
│   ├── Issue boards
│   └── Labels
├── Merge Requests
│   ├── Approvals
│   └── Reviews
├── CI/CD
│   ├── Pipelines
│   ├── Jobs
│   └── Schedules
├── Deployments
│   ├── Environments
│   └── Feature flags
├── Packages & Registries
│   ├── Container Registry
│   └── Package Registry
└── Settings
    ├── General
    ├── CI/CD
    └── Repository
```

---

## Repository Management

### Repository Settings

```bash
# Settings → General

# Project information
Name: my-project
Description: Project description
Project avatar: Upload image
Topics: devops, kubernetes, docker

# Visibility, project features, permissions
Visibility level: Private/Internal/Public
Issues: Enabled
Repository: Enabled
Merge Requests: Enabled
CI/CD: Enabled
Container Registry: Enabled
Analytics: Enabled
Wiki: Enabled
Snippets: Enabled

# Merge request settings
Merge method: Merge commit / Fast-forward merge / Squash commit
Merge commit message template
Squash commit message template
Merge checks: Pipeline must succeed
```

### Branch Protection

```bash
# Settings → Repository → Protected branches

# Protect main branch
Branch: main
Allowed to merge: Maintainers
Allowed to push: No one
Allowed to force push: No
Code owner approval: Required

# Protect develop branch
Branch: develop
Allowed to merge: Developers + Maintainers
Allowed to push: Developers + Maintainers
Allowed to force push: No

# Pattern protection
Branch: release/*
Allowed to merge: Maintainers
Allowed to push: Maintainers
```

### Protected Tags

```bash
# Settings → Repository → Protected tags

# Protect release tags
Tag: v*
Allowed to create: Maintainers
Allowed to update: No one

# Protect stable tags
Tag: stable-*
Allowed to create: Maintainers
Allowed to update: No one
```

### Repository Mirroring

```bash
# Settings → Repository → Mirroring repositories

# Pull from external repository
Git repository URL: https://github.com/user/repo.git
Mirror direction: Pull
Authentication: Username/Password or SSH key
Mirror only protected branches: No
Update schedule: Every hour

# Push to external repository
Git repository URL: https://github.com/user/repo.git
Mirror direction: Push
Authentication: Password or SSH public key
Keep divergent refs: Yes
Mirror specific branches: main, develop
```

### Repository Templates

```bash
# Create template project
# Settings → General → Badges
# Add badges for build status, coverage, etc.

# .gitignore templates
.gitignore (select from templates)
- Node
- Python
- Java
- Go
- Ruby

# License templates
LICENSE (select from templates)
- MIT
- Apache 2.0
- GPL
- BSD
```

### Webhooks

```bash
# Settings → Webhooks

# Add webhook
URL: https://your-server.com/webhook
Secret token: your-secret-token
Trigger events:
- Push events
- Tag push events
- Comments
- Confidential comments
- Issues events
- Merge request events
- Job events
- Pipeline events
- Wiki page events
- Deployment events
- Releases events

Enable SSL verification: Yes

# Test webhook
Test → Push events

# Webhook payload example (push):
{
  "object_kind": "push",
  "before": "abc123",
  "after": "def456",
  "ref": "refs/heads/main",
  "user_name": "John Doe",
  "user_email": "john@example.com",
  "project": {
    "name": "my-project",
    "path_with_namespace": "username/my-project"
  },
  "commits": [...]
}
```

---

## GitLab CI/CD

### Introduction to GitLab CI/CD

GitLab CI/CD uses a `.gitlab-ci.yml` file in the repository root to define pipeline configuration.

```yaml
# Basic .gitlab-ci.yml structure
stages:
  - build
  - test
  - deploy

build_job:
  stage: build
  script:
    - echo "Building..."
    - npm install
    - npm run build

test_job:
  stage: test
  script:
    - echo "Testing..."
    - npm test

deploy_job:
  stage: deploy
  script:
    - echo "Deploying..."
    - ./deploy.sh
```

### Pipeline Triggers

```bash
# Automatic triggers:
# - Push to branch
# - Merge request
# - Tag creation
# - Schedule

# Manual trigger:
# CI/CD → Pipelines → Run pipeline
# Select branch/tag

# Trigger via API:
curl -X POST \
  --form token=<token> \
  --form ref=main \
  https://gitlab.com/api/v4/projects/<project_id>/trigger/pipeline

# Trigger on external events (webhook)
# Trigger on merge request
# Trigger on schedule
```

### Basic Pipeline Configuration

```yaml
# .gitlab-ci.yml

# Default image for all jobs
default:
  image: node:16
  before_script:
    - npm ci

# Pipeline stages
stages:
  - build
  - test
  - deploy

# Variables
variables:
  NODE_ENV: production
  API_URL: https://api.example.com

# Build job
build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

# Test job
test:unit:
  stage: test
  script:
    - npm run test:unit
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'

test:integration:
  stage: test
  script:
    - npm run test:integration
  services:
    - postgres:14

# Deploy job
deploy:staging:
  stage: deploy
  script:
    - echo "Deploying to staging..."
    - ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy:production:
  stage: deploy
  script:
    - echo "Deploying to production..."
    - ./deploy.sh production
  environment:
    name: production
    url: https://example.com
  only:
    - main
  when: manual
```

### Job Keywords

```yaml
# Complete job example with all keywords
job_name:
  stage: test
  image: python:3.11
  services:
    - postgres:14
  before_script:
    - pip install -r requirements.txt
  script:
    - python manage.py test
  after_script:
    - echo "Cleanup"
  variables:
    DATABASE_URL: postgresql://user:pass@postgres/db
  artifacts:
    paths:
      - reports/
    expire_in: 30 days
    when: always
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - .cache/
  dependencies:
    - build_job
  needs:
    - build_job
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  allow_failure: false
  when: on_success
  timeout: 1h
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
  tags:
    - docker
    - linux
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop_staging
  coverage: '/Coverage: \d+\.\d+%/'
  parallel: 3
```

---

## GitLab Runner

### Installing GitLab Runner

```bash
# Linux (Debian/Ubuntu)
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
sudo apt-get install gitlab-runner

# Linux (RHEL/CentOS)
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash
sudo yum install gitlab-runner

# macOS
brew install gitlab-runner

# Windows
# Download from https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-windows-amd64.exe

# Docker
docker run -d --name gitlab-runner --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v gitlab-runner-config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest
```

### Registering a Runner

```bash
# Interactive registration
sudo gitlab-runner register

# Prompts:
# GitLab instance URL: https://gitlab.com
# Registration token: (from Settings → CI/CD → Runners)
# Description: my-runner
# Tags: docker, linux
# Executor: docker
# Default Docker image: alpine:latest

# Non-interactive registration
sudo gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com" \
  --registration-token "PROJECT_REGISTRATION_TOKEN" \
  --executor "docker" \
  --docker-image alpine:latest \
  --description "docker-runner" \
  --tag-list "docker,aws" \
  --run-untagged="true" \
  --locked="false"
```

### Runner Executors

```bash
# 1. Shell Executor
# Runs on local machine
sudo gitlab-runner register \
  --executor shell

# 2. Docker Executor (most common)
sudo gitlab-runner register \
  --executor docker \
  --docker-image alpine:latest \
  --docker-volumes /var/run/docker.sock:/var/run/docker.sock

# 3. Docker Machine Executor (auto-scaling)
sudo gitlab-runner register \
  --executor docker+machine

# 4. Kubernetes Executor
sudo gitlab-runner register \
  --executor kubernetes \
  --kubernetes-namespace gitlab-runner

# 5. VirtualBox Executor
sudo gitlab-runner register \
  --executor virtualbox

# 6. SSH Executor
sudo gitlab-runner register \
  --executor ssh \
  --ssh-host example.com \
  --ssh-user gitlab-runner
```

### Runner Configuration

```toml
# /etc/gitlab-runner/config.toml

concurrent = 4
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "docker-runner"
  url = "https://gitlab.com"
  token = "TOKEN"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
    shm_size = 0
```

### Runner Management

```bash
# List runners
sudo gitlab-runner list

# Verify runner
sudo gitlab-runner verify

# Start runner
sudo gitlab-runner start

# Stop runner
sudo gitlab-runner stop

# Restart runner
sudo gitlab-runner restart

# Unregister runner
sudo gitlab-runner unregister --name my-runner

# Unregister all runners
sudo gitlab-runner unregister --all-runners

# Check runner status
sudo gitlab-runner status

# Run single job
sudo gitlab-runner exec docker build_job
```

### Shared vs Specific Runners

```bash
# Shared Runners (GitLab.com)
# - Available to all projects
# - Limited minutes on free tier
# - Managed by GitLab

# Specific Runners (Project/Group)
# - Registered to specific project or group
# - Full control
# - No minute limitations

# Enable shared runners:
# Settings → CI/CD → Runners → Enable shared runners

# Disable shared runners:
# Settings → CI/CD → Runners → Disable shared runners

# Use specific runner in job:
job:
  tags:
    - my-specific-runner
```

### Runner Autoscaling

```toml
# config.toml for Docker Machine autoscaling

[[runners]]
  name = "autoscale-runner"
  url = "https://gitlab.com"
  token = "TOKEN"
  executor = "docker+machine"
  limit = 10  # Maximum parallel jobs

  [runners.docker]
    image = "alpine:latest"

  [runners.machine]
    IdleCount = 2  # Keep 2 idle machines
    IdleTime = 600  # 10 minutes
    MaxBuilds = 20  # Max builds per machine
    MachineDriver = "amazonec2"
    MachineName = "gitlab-runner-%s"

    [runners.machine.amazonec2]
      access-key = "AWS_ACCESS_KEY"
      secret-key = "AWS_SECRET_KEY"
      region = "us-east-1"
      instance-type = "t3.medium"
      ami = "ami-12345678"
      vpc-id = "vpc-12345"
      subnet-id = "subnet-12345"
      security-group = "sg-12345"
```

---

## Pipeline Configuration

### Stages

```yaml
# Define pipeline stages
stages:
  - .pre       # Special stage (always first)
  - build
  - test
  - deploy
  - .post      # Special stage (always last)

# Jobs without stage go to 'test' stage by default
job_without_stage:
  script:
    - echo "Runs in test stage"
```

### Variables

```yaml
# Global variables
variables:
  GLOBAL_VAR: "global value"
  DATABASE_URL: "postgres://localhost/db"

# Job-specific variables
build:
  variables:
    BUILD_ENV: "production"
  script:
    - echo $BUILD_ENV
    - echo $GLOBAL_VAR

# Predefined CI/CD variables:
# CI_COMMIT_SHA - Commit SHA
# CI_COMMIT_REF_NAME - Branch or tag name
# CI_PIPELINE_ID - Pipeline ID
# CI_JOB_ID - Job ID
# CI_PROJECT_NAME - Project name
# CI_PROJECT_PATH - Project path
# CI_REGISTRY - Container registry URL
# CI_REGISTRY_USER - Registry username
# CI_REGISTRY_PASSWORD - Registry password

# Protected variables (Settings → CI/CD → Variables)
# - Masked (hidden in job logs)
# - Protected (only available in protected branches/tags)
```

### Rules

```yaml
# Rules-based job execution (replaces only/except)

# Run on main branch
deploy:
  script: ./deploy.sh
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# Run on merge requests
test:
  script: npm test
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# Run on tags
release:
  script: ./release.sh
  rules:
    - if: $CI_COMMIT_TAG

# Multiple conditions
job:
  script: echo "hello"
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: always
    - if: $CI_COMMIT_BRANCH == "develop"
      when: manual
    - when: never

# File changes
build:
  script: npm run build
  rules:
    - changes:
        - src/**/*
        - package.json
      when: always

# Exists
job:
  script: echo "has Dockerfile"
  rules:
    - exists:
        - Dockerfile
```

### Artifacts

```yaml
# Artifacts are files generated by jobs
# Passed to subsequent jobs
# Available for download

build:
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
      - reports/
    exclude:
      - dist/tmp/
    expire_in: 1 week  # 30 mins, 1 hour, 1 day, 1 week, never
    when: always       # on_success (default), on_failure, always
    name: "$CI_JOB_NAME-$CI_COMMIT_REF_NAME"
    expose_as: "Build artifacts"

# Reports (special artifacts)
test:
  script:
    - npm test
  artifacts:
    reports:
      junit: reports/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
      dotenv: build.env

# Dependencies
deploy:
  script:
    - ls dist/  # Access build artifacts
  dependencies:
    - build

# Needs (with artifacts)
deploy:
  needs:
    - job: build
      artifacts: true
```

### Cache

```yaml
# Cache directories between pipeline runs
# Faster builds

# Global cache
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .npm/

# Job cache
build:
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  script:
    - npm ci
    - npm run build

# Multiple caches
test:
  cache:
    - key: npm-cache
      paths:
        - node_modules/
    - key: build-cache
      paths:
        - .cache/

# Cache policy
build:
  cache:
    key: build-cache
    paths:
      - node_modules/
    policy: pull-push  # pull, push, pull-push (default)

test:
  cache:
    key: build-cache
    paths:
      - node_modules/
    policy: pull  # Only download, don't upload
```

### Services

```yaml
# Docker services (databases, etc.)

# Single service
test:
  image: python:3.11
  services:
    - postgres:14
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: testuser
    POSTGRES_PASSWORD: testpass
  script:
    - python manage.py test

# Multiple services
integration_test:
  image: node:16
  services:
    - postgres:14
    - redis:7
    - mongo:6
  variables:
    POSTGRES_DB: testdb
    POSTGRES_PASSWORD: testpass
  script:
    - npm run test:integration

# Custom service configuration
test:
  services:
    - name: postgres:14
      alias: db
      command: ["postgres", "-c", "log_statement=all"]
  variables:
    POSTGRES_DB: testdb
  script:
    - psql -h db -U postgres -c "SELECT 1"
```

### Parallel Jobs

```yaml
# Run job multiple times in parallel
test:
  script:
    - npm test
  parallel: 5  # Creates 5 parallel jobs

# With matrix
test:
  script:
    - npm test
  parallel:
    matrix:
      - NODE_VERSION: ["14", "16", "18"]
        OS: ["ubuntu", "alpine"]
  # Creates 6 jobs (3 versions × 2 OS)

# Access parallel variables
test:
  script:
    - echo "Testing on Node $NODE_VERSION with $OS"
  parallel:
    matrix:
      - NODE_VERSION: ["14", "16", "18"]
        OS: ["ubuntu", "alpine"]
```

### Includes

```yaml
# Include external YAML files

# Local file
include:
  - local: '/templates/.gitlab-ci-template.yml'

# Remote file
include:
  - remote: 'https://example.com/ci-template.yml'

# Template from GitLab
include:
  - template: Auto-DevOps.gitlab-ci.yml

# Project file
include:
  - project: 'my-group/my-project'
    ref: main
    file: '/templates/.gitlab-ci-template.yml'

# Multiple includes
include:
  - local: '/templates/build.yml'
  - local: '/templates/test.yml'
  - local: '/templates/deploy.yml'
```

### Extends

```yaml
# Reuse configuration with extends

.test_template:
  stage: test
  image: node:16
  before_script:
    - npm ci
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

test:unit:
  extends: .test_template
  script:
    - npm run test:unit

test:integration:
  extends: .test_template
  script:
    - npm run test:integration

# Multiple inheritance
.database_template:
  services:
    - postgres:14
  variables:
    POSTGRES_DB: testdb

test:e2e:
  extends:
    - .test_template
    - .database_template
  script:
    - npm run test:e2e
```

---

## Advanced CI/CD Features

### Environments

```yaml
# Define deployment environments

deploy:staging:
  stage: deploy
  script:
    - echo "Deploying to staging"
    - ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop_staging
    auto_stop_in: 1 day
  only:
    - develop

deploy:production:
  stage: deploy
  script:
    - echo "Deploying to production"
    - ./deploy.sh production
  environment:
    name: production
    url: https://example.com
    action: start
  only:
    - main
  when: manual

# Stop environment
stop_staging:
  stage: deploy
  script:
    - echo "Stopping staging"
    - ./stop.sh staging
  environment:
    name: staging
    action: stop
  when: manual

# Dynamic environments
deploy:review:
  stage: deploy
  script:
    - ./deploy-review.sh
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: https://$CI_COMMIT_REF_SLUG.example.com
    on_stop: stop_review
    auto_stop_in: 1 week
  only:
    - merge_requests

stop_review:
  stage: deploy
  script:
    - ./stop-review.sh
  environment:
    name: review/$CI_COMMIT_REF_NAME
    action: stop
  when: manual
  only:
    - merge_requests
```

### Pipeline Schedules

```bash
# Create pipeline schedule
# CI/CD → Schedules → New schedule

# Settings:
Description: Nightly build
Interval pattern: 0 2 * * *  # Cron syntax (2 AM daily)
Cron timezone: UTC
Target branch: main
Variables:
  SCHEDULE_TYPE: nightly

# Use in pipeline:
nightly_build:
  script:
    - npm run build
    - npm run test:full
  rules:
    - if: $SCHEDULE_TYPE == "nightly"
```

### Trigger Pipelines

```yaml
# Trigger downstream pipeline
trigger_downstream:
  stage: deploy
  trigger:
    project: group/downstream-project
    branch: main
    strategy: depend  # Wait for downstream pipeline

# Multi-project pipeline
trigger_qa:
  stage: test
  trigger:
    project: qa-team/qa-tests
    strategy: depend

# Parent-child pipeline
trigger_child:
  stage: build
  trigger:
    include: .child-pipeline.yml
    strategy: depend

# .child-pipeline.yml
build:
  script:
    - echo "Child pipeline"
```

### Needs (DAG)

```yaml
# Directed Acyclic Graph (DAG)
# Jobs don't have to wait for entire stage

stages:
  - build
  - test
  - deploy

build:linux:
  stage: build
  script: build-linux.sh
  artifacts:
    paths:
      - build-linux/

build:windows:
  stage: build
  script: build-windows.sh
  artifacts:
    paths:
      - build-windows/

test:linux:
  stage: test
  needs: [build:linux]  # Only needs build:linux, not build:windows
  script: test-linux.sh

test:windows:
  stage: test
  needs: [build:windows]
  script: test-windows.sh

deploy:
  stage: deploy
  needs: [test:linux, test:windows]
  script: deploy.sh
```

### Docker in Docker (DinD)

```yaml
# Build Docker images in CI

build:docker:
  image: docker:24
  services:
    - docker:24-dind
  variables:
    DOCKER_HOST: tcp://docker:2376
    DOCKER_TLS_CERTDIR: "/certs"
    DOCKER_TLS_VERIFY: 1
    DOCKER_CERT_PATH: "$DOCKER_TLS_CERTDIR/client"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
```

### Kaniko (Rootless Docker Builds)

```yaml
# Build Docker images without Docker daemon

build:kaniko:
  stage: build
  image:
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script:
    - echo "{\"auths\":{\"$CI_REGISTRY\":{\"username\":\"$CI_REGISTRY_USER\",\"password\":\"$CI_REGISTRY_PASSWORD\"}}}" > /kaniko/.docker/config.json
    - /kaniko/executor
      --context $CI_PROJECT_DIR
      --dockerfile $CI_PROJECT_DIR/Dockerfile
      --destination $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
      --destination $CI_REGISTRY_IMAGE:latest
```

### Code Quality

```yaml
# Code quality scanning
code_quality:
  stage: test
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker run
        --env SOURCE_CODE="$PWD"
        --volume "$PWD":/code
        --volume /var/run/docker.sock:/var/run/docker.sock
        registry.gitlab.com/gitlab-org/ci-cd/codequality:latest /code
  artifacts:
    reports:
      codequality: gl-code-quality-report.json
    expire_in: 1 week
```

### Test Coverage Visualization

```yaml
test:coverage:
  stage: test
  script:
    - npm test -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# View coverage in merge request diff
# Settings → CI/CD → General pipelines → Test coverage parsing
# Regular expression: Lines\s*:\s*(\d+\.\d+)%
```

---

## Merge Requests

### Creating Merge Requests

```bash
# Via GitLab UI
# Repository → Merge Requests → New merge request
# Source branch: feature/new-feature
# Target branch: main
# Title: Add new feature
# Description: Detailed description
# Assignee: user
# Reviewer: user
# Labels: enhancement, frontend
# Milestone: v1.0

# Via Git push options
git push -o merge_request.create \
  -o merge_request.target=main \
  -o merge_request.title="Add new feature" \
  -o merge_request.description="Detailed description" \
  origin feature/new-feature

# Via glab CLI
glab mr create \
  --title "Add new feature" \
  --description "Detailed description" \
  --source feature/new-feature \
  --target main \
  --assignee @me \
  --label enhancement
```

### Merge Request Templates

```markdown
# Create .gitlab/merge_request_templates/default.md

## Description
Brief description of changes

## Related Issues
Closes #123

## Type of Change
- [ ] Bug fix (non-breaking change)
- [ ] New feature (non-breaking change)
- [ ] Breaking change (fix or feature that breaks existing functionality)
- [ ] Documentation update

## How Has This Been Tested?
- [ ] Unit tests
- [ ] Integration tests
- [ ] Manual testing

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex code
- [ ] Documentation updated
- [ ] No new warnings
- [ ] Tests added/updated
- [ ] All tests passing
- [ ] No merge conflicts

## Screenshots (if applicable)
Add screenshots here

## Additional Notes
Any additional information
```

### Merge Request Approvals

```bash
# Settings → Merge requests → Approval rules

# Create approval rule
Rule name: Backend changes
Approvals required: 2
Eligible approvers: @backend-team
Target branch: main
File patterns: api/**/*.py

# Protected branches approval
# Settings → Repository → Protected branches
# main → Allowed to merge → Developers + Maintainers
# Code owner approval required: Yes

# CODEOWNERS file
# .gitlab/CODEOWNERS
*.js @frontend-team
*.py @backend-team
/docs/ @docs-team
Dockerfile @devops-team
* @tech-lead
```

### Merge Request Pipelines

```yaml
# Run pipeline for merge requests

test:
  script:
    - npm test
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# Merge request widgets
# - Pipeline status
# - Code coverage
# - Security scanning
# - License compliance
# - Accessibility testing
```

### Merge Strategies

```bash
# Settings → Merge requests → Merge method

# 1. Merge commit (default)
# Creates merge commit preserving all commits
git merge --no-ff feature-branch

# 2. Merge commit with semi-linear history
# Requires fast-forward merge or rebase before merge
git merge --ff-only feature-branch

# 3. Fast-forward merge
# No merge commit, linear history
git merge --ff feature-branch

# 4. Squash commits
# Combines all commits into single commit
git merge --squash feature-branch
git commit -m "Feature: Add user authentication"

# Settings:
# - Enable "Delete source branch" option
# - Enable "Squash commits when merging"
# - Enable "Require merge request approval"
```

### Draft Merge Requests

```bash
# Create draft (WIP) merge request
# Title: Draft: Add new feature
# or
# Title: WIP: Add new feature

# Mark as ready
# Remove "Draft:" or "WIP:" prefix

# Draft merge requests:
# - Can't be merged
# - Useful for early feedback
# - Run CI/CD pipelines
```

---

## GitLab Container Registry

### Enabling Container Registry

```bash
# Container Registry is enabled by default for projects

# Access registry
# Packages & Registries → Container Registry

# Registry URL
registry.gitlab.com/username/project

# Authenticate
docker login registry.gitlab.com
# Username: GitLab username
# Password: Personal access token or deploy token
```

### Building and Pushing Images

```yaml
# .gitlab-ci.yml

variables:
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    # Build image
    - docker build -t $IMAGE_TAG .
    - docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest

    # Push to registry
    - docker push $IMAGE_TAG
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
    - develop

# Pull image in subsequent jobs
deploy:
  stage: deploy
  image: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  script:
    - ./deploy.sh
```

### Multi-Platform Builds

```yaml
build:multiplatform:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker buildx create --use
  script:
    - docker buildx build \
        --platform linux/amd64,linux/arm64 \
        --tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA \
        --tag $CI_REGISTRY_IMAGE:latest \
        --push \
        .
```

### Cleanup Policies

```bash
# Settings → Packages & Registries → Container Registry

# Cleanup policy:
# - Run cleanup: Every day/week/month
# - Keep N tags
# - Keep tags matching: ^v\d+\.\d+\.\d+$ (semver tags)
# - Remove tags older than: 90 days
# - Remove tags matching: .*-dev
# - Exclude tags: latest, stable

# Manual cleanup
# Delete specific tag via UI or API

# API cleanup
curl --request DELETE \
  --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/<project_id>/registry/repositories/<repo_id>/tags/<tag>"
```

---

## Security & Compliance

### SAST (Static Application Security Testing)

```yaml
# Include SAST template
include:
  - template: Security/SAST.gitlab-ci.yml

# Customize SAST
sast:
  stage: test
  variables:
    SAST_EXCLUDED_PATHS: spec,test,tests,tmp
    SAST_EXCLUDED_ANALYZERS: bandit,brakeman

# Results appear in:
# - Merge request → Security tab
# - Security → Vulnerability Report
```

### DAST (Dynamic Application Security Testing)

```yaml
# Include DAST template
include:
  - template: Security/DAST.gitlab-ci.yml

# Configure DAST
dast:
  stage: test
  variables:
    DAST_WEBSITE: https://example.com
    DAST_FULL_SCAN_ENABLED: "true"
    DAST_AUTH_URL: https://example.com/login
    DAST_USERNAME: testuser
    DAST_PASSWORD: $DAST_PASSWORD  # CI/CD variable
```

### Dependency Scanning

```yaml
# Include dependency scanning template
include:
  - template: Security/Dependency-Scanning.gitlab-ci.yml

# Scans:
# - package.json (npm)
# - requirements.txt (pip)
# - Gemfile.lock (bundler)
# - pom.xml (maven)
# - go.mod (go)
# - composer.lock (composer)
```

### Container Scanning

```yaml
# Include container scanning template
include:
  - template: Security/Container-Scanning.gitlab-ci.yml

# Scan Docker image
container_scanning:
  stage: test
  variables:
    CS_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  dependencies:
    - build
```

### Secret Detection

```yaml
# Include secret detection template
include:
  - template: Security/Secret-Detection.gitlab-ci.yml

# Scans for:
# - API keys
# - Passwords
# - Tokens
# - Private keys
# - Database credentials
```

### License Compliance

```yaml
# Include license compliance template
include:
  - template: Security/License-Scanning.gitlab-ci.yml

# Configure allowed/denied licenses
# Settings → CI/CD → License Compliance
# Allowed: MIT, Apache-2.0, BSD
# Denied: GPL, AGPL
```

### Security Dashboard

```bash
# View security findings
# Security & Compliance → Vulnerability Report

# Filter by:
# - Severity (Critical, High, Medium, Low, Info)
# - Scanner (SAST, DAST, Dependency, Container)
# - Status (Detected, Confirmed, Dismissed, Resolved)

# Create issue from vulnerability
# Vulnerability → Create issue

# Dismiss vulnerability
# Vulnerability → Dismiss → Add reason
```

---

## GitLab Pages

### Deploying Static Sites

```yaml
# .gitlab-ci.yml for static site

image: node:16

pages:
  stage: deploy
  script:
    - npm ci
    - npm run build
    - mv dist public  # Must output to 'public' directory
  artifacts:
    paths:
      - public
  only:
    - main

# Site will be available at:
# https://username.gitlab.io/project-name
```

### Custom Domain

```bash
# Settings → Pages → New Domain

# Add domain: example.com
# DNS records:
# A record: 35.185.44.232
# TXT record: gitlab-pages-verification-code=xxx

# Enable HTTPS:
# - Let's Encrypt (automatic)
# - Custom certificate

# Update DNS:
# CNAME: username.gitlab.io
```

### Examples

```yaml
# Jekyll site
pages:
  image: ruby:2.7
  script:
    - bundle install
    - bundle exec jekyll build -d public
  artifacts:
    paths:
      - public
  only:
    - main

# Hugo site
pages:
  image: registry.gitlab.com/pages/hugo:latest
  script:
    - hugo
  artifacts:
    paths:
      - public
  only:
    - main

# React app
pages:
  image: node:16
  script:
    - npm ci
    - npm run build
    - mv build public
  artifacts:
    paths:
      - public
  only:
    - main

# Vue.js app
pages:
  image: node:16
  script:
    - npm ci
    - npm run build
    - mv dist public
  artifacts:
    paths:
      - public
  only:
    - main
```

---

## Package Registry

### NPM Registry

```yaml
# Publish to GitLab NPM registry

publish:npm:
  stage: deploy
  image: node:16
  script:
    # Set registry URL
    - echo "@${CI_PROJECT_ROOT_NAMESPACE}:registry=${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/npm/" > .npmrc
    - echo "${CI_API_V4_URL#https?}/projects/${CI_PROJECT_ID}/packages/npm/:_authToken=\${CI_JOB_TOKEN}" >> .npmrc

    # Publish
    - npm publish
  only:
    - tags

# Install from GitLab registry
# .npmrc
@mygroup:registry=https://gitlab.com/api/v4/projects/<project_id>/packages/npm/

# Install package
npm install @mygroup/my-package
```

### Maven Registry

```yaml
# Publish to GitLab Maven registry

publish:maven:
  stage: deploy
  image: maven:3.8-openjdk-11
  script:
    - mvn deploy -s ci_settings.xml
  only:
    - tags

# ci_settings.xml
<settings>
  <servers>
    <server>
      <id>gitlab-maven</id>
      <configuration>
        <httpHeaders>
          <property>
            <name>Job-Token</name>
            <value>${CI_JOB_TOKEN}</value>
          </property>
        </httpHeaders>
      </configuration>
    </server>
  </servers>
</settings>

# pom.xml
<distributionManagement>
  <repository>
    <id>gitlab-maven</id>
    <url>https://gitlab.com/api/v4/projects/<project_id>/packages/maven</url>
  </repository>
</distributionManagement>
```

### PyPI Registry

```yaml
# Publish to GitLab PyPI registry

publish:pypi:
  stage: deploy
  image: python:3.11
  script:
    - pip install twine
    - python setup.py sdist bdist_wheel
    - TWINE_PASSWORD=${CI_JOB_TOKEN} TWINE_USERNAME=gitlab-ci-token python -m twine upload --repository-url ${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/pypi dist/*
  only:
    - tags

# Install from GitLab registry
pip install --index-url https://__token__:<personal_token>@gitlab.com/api/v4/projects/<project_id>/packages/pypi/simple my-package
```

### Helm Charts

```yaml
# Publish Helm charts

publish:helm:
  stage: deploy
  image: alpine/helm:3.13.0
  script:
    - helm package mychart/
    - 'curl --request POST --user gitlab-ci-token:$CI_JOB_TOKEN --form "chart=@mychart-0.1.0.tgz" "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/helm/api/stable/charts"'
  only:
    - tags

# Add Helm repository
helm repo add --username <username> --password <access_token> myrepo https://gitlab.com/api/v4/projects/<project_id>/packages/helm/stable
```

---

## Auto DevOps

### Enabling Auto DevOps

```bash
# Settings → CI/CD → Auto DevOps
# Enable Auto DevOps ✓

# Auto DevOps includes:
# - Auto Build (Docker image)
# - Auto Test
# - Auto Code Quality
# - Auto SAST
# - Auto Dependency Scanning
# - Auto Container Scanning
# - Auto Review Apps
# - Auto Deploy to Kubernetes
# - Auto Monitoring

# Requirements:
# - Dockerfile (or buildpack detection)
# - Kubernetes cluster (for deployment)
```

### Customizing Auto DevOps

```yaml
# .gitlab-ci.yml
# Customize Auto DevOps stages

include:
  - template: Auto-DevOps.gitlab-ci.yml

# Override variables
variables:
  POSTGRES_ENABLED: "true"
  POSTGRES_VERSION: "14"
  POSTGRES_DB: "myapp"

# Override jobs
test:
  before_script:
    - npm ci
  script:
    - npm test
    - npm run lint

# Disable specific jobs
code_quality:
  rules:
    - when: never
```

### Auto Deploy

```yaml
# Auto Deploy to Kubernetes

# Requirements:
# 1. Add Kubernetes cluster
#    Infrastructure → Kubernetes clusters → Add cluster

# 2. Install GitLab Runner in cluster
# 3. Install Ingress controller
# 4. Configure domain

# Settings → CI/CD → Variables
AUTO_DEVOPS_DOMAIN: example.com
KUBE_NAMESPACE: production

# Deployments:
# - Review apps: Temporary apps for merge requests
# - Staging: deploy to staging environment
# - Production: deploy to production (manual)
```

---

## GitLab API

### Authentication

```bash
# Personal Access Token
# Profile → Access Tokens → Add new token
# Scopes: api, read_api, read_repository, write_repository

# Use token
curl --header "PRIVATE-TOKEN: <your_access_token>" \
  "https://gitlab.com/api/v4/projects"

# OAuth2
# Project → Settings → Applications → Add new application

# Job Token (in CI/CD)
curl --header "JOB-TOKEN: $CI_JOB_TOKEN" \
  "https://gitlab.com/api/v4/projects/$CI_PROJECT_ID"
```

### Common API Operations

```bash
# List projects
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects"

# Get project
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/<project_id>"

# List branches
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/<project_id>/repository/branches"

# Create branch
curl --request POST \
  --header "PRIVATE-TOKEN: <token>" \
  --data "branch=new-branch&ref=main" \
  "https://gitlab.com/api/v4/projects/<project_id>/repository/branches"

# List merge requests
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/<project_id>/merge_requests"

# Create merge request
curl --request POST \
  --header "PRIVATE-TOKEN: <token>" \
  --data "source_branch=feature&target_branch=main&title=New Feature" \
  "https://gitlab.com/api/v4/projects/<project_id>/merge_requests"

# List pipelines
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/<project_id>/pipelines"

# Trigger pipeline
curl --request POST \
  --header "PRIVATE-TOKEN: <token>" \
  --data "ref=main" \
  "https://gitlab.com/api/v4/projects/<project_id>/pipeline"

# Get job log
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.com/api/v4/projects/<project_id>/jobs/<job_id>/trace"
```

### Python API Client

```python
# Install python-gitlab
# pip install python-gitlab

import gitlab

# Connect to GitLab
gl = gitlab.Gitlab('https://gitlab.com', private_token='<token>')

# Get project
project = gl.projects.get('<project_id>')

# List branches
branches = project.branches.list()
for branch in branches:
    print(branch.name)

# Create branch
branch = project.branches.create({'branch': 'new-feature', 'ref': 'main'})

# List merge requests
mrs = project.mergerequests.list(state='opened')
for mr in mrs:
    print(f"MR !{mr.iid}: {mr.title}")

# Create merge request
mr = project.mergerequests.create({
    'source_branch': 'feature',
    'target_branch': 'main',
    'title': 'New Feature',
    'description': 'Description here'
})

# Approve merge request
mr.approve()

# Merge merge request
mr.merge()

# List pipelines
pipelines = project.pipelines.list()
for pipeline in pipelines:
    print(f"Pipeline #{pipeline.id}: {pipeline.status}")

# Create pipeline
pipeline = project.pipelines.create({'ref': 'main'})

# List jobs
jobs = pipeline.jobs.list()
for job in jobs:
    print(f"Job {job.name}: {job.status}")

# Get job trace
job = project.jobs.get(job_id)
trace = job.trace()
print(trace.decode())
```

---

## GitLab Administration

### Instance Administration (Self-Hosted)

```bash
# Access admin area
# Admin Area (wrench icon)

# Overview
# - Projects
# - Users
# - Groups
# - Statistics
# - Jobs

# Settings
# - General (visibility, account limits)
# - CI/CD (runners, variables)
# - Repository (mirroring, storage)
# - Security (password requirements, 2FA)
```

### User Management

```bash
# Admin Area → Users

# Create user
New user →
- Name
- Username
- Email
- Access level (Regular, Admin)
- Projects limit
- Can create group

# Impersonate user
User → Impersonate

# Block/Unblock user
User → Block / Unblock

# Delete user
User → Delete user → Delete user and contributions
```

### Runner Management

```bash
# Admin Area → CI/CD → Runners

# Instance runners (available to all projects)
# Group runners (available to group projects)
# Project runners (specific to project)

# Runner configuration
Maximum job timeout: 3600 seconds
Tags: docker, linux, shell
Run untagged jobs: No
Lock to current projects: Yes
```

### Backup and Restore

```bash
# Backup GitLab (omnibus)
sudo gitlab-backup create

# Backup location
/var/opt/gitlab/backups/

# Backup with specific name
sudo gitlab-backup create BACKUP=mybackup

# Restore backup
sudo gitlab-backup restore BACKUP=<timestamp>

# Restore specific components
sudo gitlab-backup restore BACKUP=<timestamp> SKIP=db,uploads

# Backup configuration
/etc/gitlab/gitlab-secrets.json
/etc/gitlab/gitlab.rb
```

### Monitoring

```bash
# Built-in monitoring
# Admin Area → Monitoring

# Metrics:
# - System info
# - Background jobs
# - Requests per second
# - Database performance

# Enable Prometheus
# /etc/gitlab/gitlab.rb
prometheus_monitoring['enable'] = true
gitlab_rails['monitoring_whitelist'] = ['127.0.0.1/32']

sudo gitlab-ctl reconfigure

# Grafana dashboards
# Admin Area → Monitoring → Grafana
```

---

## Troubleshooting

### Pipeline Issues

```bash
# 1. Pipeline not running
# Check:
# - .gitlab-ci.yml syntax (CI/CD → Editor → Validate)
# - Runner availability (Settings → CI/CD → Runners)
# - Pipeline triggers (rules, only, except)

# 2. Job stuck
# Check:
# - Runner logs: sudo gitlab-runner verify
# - Available runners with matching tags
# - Concurrent job limit

# 3. Job failing
# Check:
# - Job logs
# - Environment variables
# - Services status
# - Artifact dependencies

# 4. Cache not working
# Check:
# - Cache key
# - Cache paths
# - Runner cache configuration

# 5. Artifacts not available
# Check:
# - Artifact expiration
# - Artifact paths match build output
# - Job dependencies
```

### Runner Issues

```bash
# Check runner status
sudo gitlab-runner status

# Verify runner registration
sudo gitlab-runner verify

# Check runner logs
sudo gitlab-runner --debug run

# Restart runner
sudo gitlab-runner restart

# Clear runner cache
sudo gitlab-runner cache-clear

# Re-register runner
sudo gitlab-runner unregister --all-runners
sudo gitlab-runner register
```

### Performance Issues

```bash
# 1. Slow clone
# Use shallow clone
variables:
  GIT_DEPTH: 1

# 2. Slow builds
# Use caching effectively
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .cache/

# 3. Docker layer caching
# Use Kaniko or buildkit

# 4. Large artifacts
# Reduce artifact size
# Decrease artifact expiration
artifacts:
  expire_in: 1 week
```

### Authentication Issues

```bash
# 1. SSH key not working
# Check:
ssh -T git@gitlab.com
# Verify key added to GitLab
# Check SSH agent: ssh-add -l

# 2. HTTP 401 Unauthorized
# Check:
# - Personal access token validity
# - Token scopes
# - Repository permissions

# 3. Runner authentication failed
# Check:
# - Registration token (Settings → CI/CD → Runners)
# - Re-register runner
```

---

## Best Practices

### Repository Organization

```bash
# Group structure
company/
├── frontend/
│   ├── web-app
│   ├── mobile-app
│   └── design-system
├── backend/
│   ├── api-gateway
│   ├── user-service
│   └── payment-service
└── infrastructure/
    ├── terraform
    ├── kubernetes
    └── ansible

# Naming conventions
# - Lowercase with hyphens: user-service
# - Descriptive: frontend-web-app
# - Avoid: repo1, test, new

# README structure
# - Project description
# - Prerequisites
# - Installation
# - Usage
# - Configuration
# - Testing
# - Deployment
# - Contributing
# - License
```

### CI/CD Best Practices

```yaml
# 1. Use templates and extends
.test_template:
  stage: test
  image: node:16
  before_script:
    - npm ci

test:unit:
  extends: .test_template
  script:
    - npm run test:unit

# 2. Fail fast
stages:
  - validate
  - build
  - test
  - deploy

validate:
  stage: validate
  script:
    - npm run lint
    - npm run type-check

# 3. Use needs for DAG
build:
  stage: build
  script: build.sh

test:unit:
  needs: [build]
  script: test-unit.sh

test:integration:
  needs: [build]
  script: test-integration.sh

# 4. Cache dependencies
cache:
  key:
    files:
      - package-lock.json
  paths:
    - node_modules/

# 5. Use specific Docker images
image: node:16.20-alpine  # Not node:latest

# 6. Secure secrets
# Use CI/CD variables (masked & protected)
# Never commit secrets

# 7. Optimize artifacts
artifacts:
  paths:
    - dist/
  expire_in: 1 week
  when: on_success
```

### Security Best Practices

```yaml
# 1. Enable security scanning
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml

# 2. Protect branches
# main: Maintainers only
# develop: Developers + Maintainers
# feature/*: All developers

# 3. Require approvals
# Settings → Merge requests → Approval rules
# Minimum approvals: 2
# Code owner approval: Required

# 4. Use protected variables
# Settings → CI/CD → Variables
# Protected: Yes (only available in protected branches)
# Masked: Yes (hidden in logs)

# 5. Regular dependency updates
# Dependabot or Renovate bot

# 6. Audit logs
# Admin Area → Monitoring → Audit Events
```

### Merge Request Best Practices

```bash
# 1. Small, focused MRs
# - Single purpose
# - < 400 lines of code
# - Easy to review

# 2. Clear descriptions
# - What changed
# - Why it changed
# - How to test
# - Related issues

# 3. Use templates
# .gitlab/merge_request_templates/

# 4. Self-review
# - Review your own diff
# - Add comments
# - Test locally

# 5. Respond to feedback
# - Address comments promptly
# - Mark resolved when fixed
# - Ask questions if unclear

# 6. Keep MR updated
# - Rebase on target branch
# - Resolve conflicts
# - Update after feedback
```

---

## Practical Examples

### Example 1: Node.js Full Stack App

```yaml
# .gitlab-ci.yml

image: node:16

stages:
  - install
  - validate
  - test
  - build
  - deploy

variables:
  NODE_ENV: production
  npm_config_cache: "$CI_PROJECT_DIR/.npm"

cache:
  key:
    files:
      - package-lock.json
  paths:
    - .npm/
    - node_modules/

install:
  stage: install
  script:
    - npm ci --cache .npm --prefer-offline
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 day

lint:
  stage: validate
  needs: [install]
  script:
    - npm run lint
    - npm run format:check

type-check:
  stage: validate
  needs: [install]
  script:
    - npm run type-check

test:unit:
  stage: test
  needs: [install]
  script:
    - npm run test:unit -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
      junit: reports/junit.xml

test:integration:
  stage: test
  needs: [install]
  services:
    - postgres:14
    - redis:7
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: testuser
    POSTGRES_PASSWORD: testpass
    REDIS_URL: redis://redis:6379
  script:
    - npm run test:integration

build:
  stage: build
  needs: [install]
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

deploy:staging:
  stage: deploy
  needs: [build, test:unit, test:integration]
  image: registry.gitlab.com/gitlab-org/cloud-deploy/aws-base:latest
  script:
    - aws s3 sync dist/ s3://$S3_BUCKET_STAGING/ --delete
    - aws cloudfront create-invalidation --distribution-id $CF_DIST_STAGING --paths "/*"
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy:production:
  stage: deploy
  needs: [build, test:unit, test:integration]
  image: registry.gitlab.com/gitlab-org/cloud-deploy/aws-base:latest
  script:
    - aws s3 sync dist/ s3://$S3_BUCKET_PROD/ --delete
    - aws cloudfront create-invalidation --distribution-id $CF_DIST_PROD --paths "/*"
  environment:
    name: production
    url: https://example.com
  only:
    - main
  when: manual
```

### Example 2: Python Django with Docker

```yaml
# .gitlab-ci.yml

stages:
  - test
  - build
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

.python_template:
  image: python:3.11
  before_script:
    - pip install -r requirements.txt -r requirements-dev.txt
  cache:
    key: ${CI_COMMIT_REF_SLUG}-pip
    paths:
      - .cache/pip

test:lint:
  extends: .python_template
  stage: test
  script:
    - flake8 .
    - black --check .
    - isort --check-only .
    - mypy .

test:unit:
  extends: .python_template
  stage: test
  services:
    - postgres:14
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: testuser
    POSTGRES_PASSWORD: testpass
    DATABASE_URL: postgresql://testuser:testpass@postgres/testdb
  script:
    - python manage.py test
    - coverage run --source='.' manage.py test
    - coverage report
    - coverage xml
  coverage: '/TOTAL.*\s+(\d+%)$/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml

build:docker:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $IMAGE_TAG .
    - docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest
    - docker push $IMAGE_TAG
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
    - develop

deploy:k8s:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context $KUBE_CONTEXT
    - kubectl set image deployment/django-app django-app=$IMAGE_TAG -n production
    - kubectl rollout status deployment/django-app -n production
  environment:
    name: production
    url: https://api.example.com
  only:
    - main
  when: manual
```

### Example 3: Microservices Monorepo

```yaml
# .gitlab-ci.yml

stages:
  - test
  - build
  - deploy

workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'
    - if: '$CI_COMMIT_BRANCH == "develop"'

.service_template:
  rules:
    - changes:
        - $SERVICE_PATH/**/*
        - .gitlab-ci.yml

# User Service
test:user-service:
  extends: .service_template
  stage: test
  image: node:16
  variables:
    SERVICE_PATH: services/user
  before_script:
    - cd $SERVICE_PATH
    - npm ci
  script:
    - npm test

build:user-service:
  extends: .service_template
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  variables:
    SERVICE_PATH: services/user
    SERVICE_NAME: user-service
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - cd $SERVICE_PATH
    - docker build -t $CI_REGISTRY_IMAGE/$SERVICE_NAME:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE/$SERVICE_NAME:$CI_COMMIT_SHA

# Order Service
test:order-service:
  extends: .service_template
  stage: test
  image: node:16
  variables:
    SERVICE_PATH: services/order
  before_script:
    - cd $SERVICE_PATH
    - npm ci
  script:
    - npm test

build:order-service:
  extends: .service_template
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  variables:
    SERVICE_PATH: services/order
    SERVICE_NAME: order-service
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - cd $SERVICE_PATH
    - docker build -t $CI_REGISTRY_IMAGE/$SERVICE_NAME:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE/$SERVICE_NAME:$CI_COMMIT_SHA

# Deploy all services
deploy:all:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/user-service user-service=$CI_REGISTRY_IMAGE/user-service:$CI_COMMIT_SHA
    - kubectl set image deployment/order-service order-service=$CI_REGISTRY_IMAGE/order-service:$CI_COMMIT_SHA
    - kubectl rollout status deployment/user-service
    - kubectl rollout status deployment/order-service
  environment:
    name: production
  only:
    - main
  when: manual
```

### Example 4: Infrastructure as Code (Terraform)

```yaml
# .gitlab-ci.yml

image:
  name: hashicorp/terraform:1.5
  entrypoint: [""]

stages:
  - validate
  - plan
  - apply

variables:
  TF_ROOT: ${CI_PROJECT_DIR}/terraform
  TF_STATE_NAME: default

cache:
  key: terraform
  paths:
    - ${TF_ROOT}/.terraform

before_script:
  - cd ${TF_ROOT}
  - terraform init

validate:
  stage: validate
  script:
    - terraform validate
    - terraform fmt -check

plan:staging:
  stage: plan
  script:
    - terraform workspace select staging || terraform workspace new staging
    - terraform plan -var-file=staging.tfvars -out=tfplan
  artifacts:
    paths:
      - ${TF_ROOT}/tfplan
    expire_in: 1 week
  only:
    - develop

plan:production:
  stage: plan
  script:
    - terraform workspace select production || terraform workspace new production
    - terraform plan -var-file=production.tfvars -out=tfplan
  artifacts:
    paths:
      - ${TF_ROOT}/tfplan
    expire_in: 1 week
  only:
    - main

apply:staging:
  stage: apply
  script:
    - terraform workspace select staging
    - terraform apply tfplan
  environment:
    name: staging
  dependencies:
    - plan:staging
  only:
    - develop
  when: manual

apply:production:
  stage: apply
  script:
    - terraform workspace select production
    - terraform apply tfplan
  environment:
    name: production
  dependencies:
    - plan:production
  only:
    - main
  when: manual
```

---

## Quick Reference

### Essential Git Commands

```bash
# Clone
git clone git@gitlab.com:username/project.git

# Branches
git checkout -b feature/new-feature
git push -u origin feature/new-feature

# Commits
git add .
git commit -m "feat: add new feature"
git push

# Merge Request
git push -o merge_request.create \
  -o merge_request.target=main \
  -o merge_request.title="Add new feature"
```

### GitLab CLI (glab)

```bash
# Authentication
glab auth login

# Projects
glab repo create myproject
glab repo list

# Merge Requests
glab mr create
glab mr list
glab mr view 123
glab mr merge 123

# Pipelines
glab ci view
glab ci trace
glab ci lint
```

### Pipeline Syntax Quick Reference

```yaml
stages:
  - build
  - test
  - deploy

job:
  stage: test
  image: node:16
  services:
    - postgres:14
  variables:
    VAR: value
  before_script:
    - npm ci
  script:
    - npm test
  after_script:
    - cleanup
  artifacts:
    paths:
      - dist/
  cache:
    paths:
      - node_modules/
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  environment:
    name: production
```

---

**End of GitLab Complete Guide**

For more information, visit:
- Official Documentation: https://docs.gitlab.com/
- GitLab CI/CD: https://docs.gitlab.com/ee/ci/
- GitLab API: https://docs.gitlab.com/ee/api/
- Community Forum: https://forum.gitlab.com/
