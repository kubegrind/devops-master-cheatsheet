# Bitbucket Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Repository Management](#repository-management)
4. [Branching Strategies](#branching-strategies)
5. [Pull Requests](#pull-requests)
6. [Bitbucket Pipelines](#bitbucket-pipelines)
7. [Pipeline Configuration](#pipeline-configuration)
8. [Advanced Pipeline Features](#advanced-pipeline-features)
9. [Deployments](#deployments)
10. [Code Review](#code-review)
11. [Access Management](#access-management)
12. [Integrations](#integrations)
13. [API and Automation](#api-and-automation)
14. [Bitbucket Server/Data Center](#bitbucket-serverdata-center)
15. [Troubleshooting](#troubleshooting)
16. [Best Practices](#best-practices)
17. [Practical Examples](#practical-examples)

---

## Introduction

### What is Bitbucket?

Bitbucket is a Git-based source code repository hosting service owned by Atlassian. It provides:
- Git repository hosting
- Built-in CI/CD (Pipelines)
- Code review (Pull Requests)
- Issue tracking (Jira integration)
- Team collaboration features

### Bitbucket Versions

**Bitbucket Cloud**: SaaS solution hosted by Atlassian
- URL: https://bitbucket.org
- Integrated with Atlassian Cloud products
- Pay per user pricing

**Bitbucket Server/Data Center**: Self-hosted solution
- On-premises deployment
- More control and customization
- Enterprise features

### Key Features

```
Repository Features:
├── Git repository hosting
├── Branch permissions
├── Merge checks
├── Code search
└── LFS support

CI/CD:
├── Bitbucket Pipelines
├── Docker-based builds
├── Deployment environments
└── Pipeline caching

Collaboration:
├── Pull Requests
├── Code review
├── Inline comments
└── Task management

Integration:
├── Jira integration
├── Trello boards
├── Slack notifications
└── Third-party apps
```

---

## Getting Started

### Account Setup

```bash
# 1. Sign up at https://bitbucket.org

# 2. Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 3. Set up SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"
cat ~/.ssh/id_ed25519.pub

# 4. Add SSH key to Bitbucket
# Personal settings → SSH keys → Add key

# 5. Test SSH connection
ssh -T git@bitbucket.org
```

### Creating Your First Repository

```bash
# Via Bitbucket Web UI:
# Repositories → Create repository

# Via Git CLI:
# 1. Create local repository
mkdir myproject
cd myproject
git init

# 2. Add remote
git remote add origin git@bitbucket.org:username/myproject.git

# 3. Create first commit
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"

# 4. Push to Bitbucket
git push -u origin main
```

### Installing Bitbucket CLI

```bash
# Install using pip
pip install bitbucket-cli

# Configure CLI
bb configure

# Basic commands
bb repo list
bb repo create myrepo
bb pr list
bb pr create
```

### Workspace Setup

```bash
# Workspaces are the top-level container in Bitbucket Cloud

# Navigate to workspace
https://bitbucket.org/workspace-name

# Workspace structure:
Workspace (username or team)
├── Projects
│   └── Repositories
└── Settings
    ├── Members
    ├── Groups
    └── Permissions
```

---

## Repository Management

### Creating Repositories

```bash
# Web UI: Repositories → Create repository

# Settings:
# - Repository name: myproject
# - Access level: Private/Public
# - Include README: Yes
# - Include .gitignore: (select language)
# - Version control: Git
# - Project: (optional grouping)

# Clone repository
git clone git@bitbucket.org:username/myproject.git
cd myproject
```

### Repository Settings

```bash
# Repository details
Repository settings → Details
- Name
- Description
- Project
- Language
- Website

# Access control
Repository settings → Access management
- User permissions
- Group permissions
- Public/Private toggle

# Branch permissions
Repository settings → Branch permissions
- Prevent deletion
- Prevent rewriting history
- Require pull requests
- Require approvals
- Require passing builds

# Merge checks
Repository settings → Merge checks
- Minimum approvals
- Default reviewer
- All tasks resolved
- All builds successful
```

### Repository Variables

```bash
# Repository settings → Repository variables

# Add variables for Pipelines
Name: API_KEY
Value: your-secret-key
Secured: ✓ (encrypted)

# Access in pipeline:
script:
  - echo $API_KEY
```

### Branch Management

```bash
# List branches
git branch -a

# Create branch
git checkout -b feature/new-feature

# Push branch
git push -u origin feature/new-feature

# Delete branch (local)
git branch -d feature/new-feature

# Delete branch (remote)
git push origin --delete feature/new-feature

# Set default branch (via Web UI)
# Repository settings → Repository details → Main branch
```

### Branch Permissions

```bash
# Repository settings → Branch permissions

# Prevent deletion of main/master
Branch pattern: main
Type: Prevent deletion
Exempt users: (admins)

# Require pull requests for protected branches
Branch pattern: main
Type: Prevent changes without a pull request
Minimum approvals: 2

# Require builds to pass
Branch pattern: main
Type: Require builds to pass
Build: bitbucket-pipelines.yml

# Allow only specific users to merge
Branch pattern: release/*
Type: Restrict who can merge
Users: release-managers group
```

### Repository Templates

```bash
# Create .gitignore
# Repository settings → .gitignore template

# Common templates:
# - Node
# - Python
# - Java
# - Go
# - Ruby
# - .NET

# Example .gitignore for Node.js:
node_modules/
npm-debug.log
.env
dist/
build/
*.log
.DS_Store

# Example .gitignore for Python:
__pycache__/
*.py[cod]
*$py.class
.env
venv/
*.egg-info/
dist/
build/
```

### Repository Webhooks

```bash
# Repository settings → Webhooks

# Create webhook
URL: https://your-server.com/webhook
Active: ✓
Events:
- Repository push
- Pull request created
- Pull request merged
- Build status updated

# Webhook payload example:
{
  "repository": {
    "name": "myrepo",
    "full_name": "username/myrepo"
  },
  "push": {
    "changes": [
      {
        "new": {
          "name": "main",
          "target": {
            "hash": "abc123"
          }
        }
      }
    ]
  }
}
```

---

## Branching Strategies

### Git Flow

```bash
# Main branches
main        # Production-ready code
develop     # Integration branch

# Supporting branches
feature/*   # New features
release/*   # Release preparation
hotfix/*    # Production fixes

# Workflow:

# 1. Start feature
git checkout develop
git checkout -b feature/user-authentication

# 2. Work on feature
git add .
git commit -m "Add user authentication"
git push -u origin feature/user-authentication

# 3. Create pull request: feature/user-authentication → develop

# 4. Start release
git checkout develop
git checkout -b release/1.0.0
# Bump version, update changelog
git commit -m "Prepare release 1.0.0"

# 5. Create pull requests:
# - release/1.0.0 → main (production)
# - release/1.0.0 → develop (merge back)

# 6. Hotfix
git checkout main
git checkout -b hotfix/security-patch
git commit -m "Fix security vulnerability"
# Create PRs to main and develop
```

### GitHub Flow (Simplified)

```bash
# Single main branch + feature branches

# 1. Create feature branch
git checkout -b feature/add-login
git push -u origin feature/add-login

# 2. Make changes and commit
git add .
git commit -m "Add login functionality"
git push

# 3. Create pull request → main

# 4. Review, test, merge

# 5. Deploy from main

# 6. Delete feature branch
git branch -d feature/add-login
git push origin --delete feature/add-login
```

### Trunk-Based Development

```bash
# Single trunk (main) branch
# Short-lived feature branches (max 1-2 days)
# Feature flags for incomplete features

# 1. Create short-lived branch
git checkout -b quick-fix

# 2. Make small changes
git add .
git commit -m "Quick fix for bug"

# 3. Push and create PR immediately
git push -u origin quick-fix

# 4. Fast review and merge (same day)

# 5. Use feature flags for incomplete work
if (featureFlags.newFeature) {
  // New code
} else {
  // Old code
}
```

### Release Flow

```bash
# Branches:
main           # Latest stable
release/*      # Release candidates
feature/*      # Features

# Workflow:

# 1. Feature development
git checkout -b feature/new-feature main
# Develop and create PR → main

# 2. Create release branch
git checkout -b release/v2.0.0 main

# 3. Release preparation (on release branch)
# - Version bump
# - Changelog
# - Bug fixes only

# 4. Tag release
git tag -a v2.0.0 -m "Release version 2.0.0"
git push origin v2.0.0

# 5. Hotfixes go to release branch, then cherry-pick to main
```

---

## Pull Requests

### Creating Pull Requests

```bash
# Via Web UI:
# 1. Navigate to repository
# 2. Click "Create pull request"
# 3. Select source and destination branches
# 4. Add title and description
# 5. Add reviewers
# 6. Create

# Via Git CLI:
git checkout -b feature/new-feature
git add .
git commit -m "Add new feature"
git push -u origin feature/new-feature

# Then create PR via Web UI or CLI:
bb pr create \
  --title "Add new feature" \
  --description "This PR adds..." \
  --source feature/new-feature \
  --destination main \
  --reviewers user1,user2
```

### Pull Request Template

```markdown
# Create .bitbucket/pull_request_template.md

## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex code
- [ ] Documentation updated
- [ ] No new warnings generated
- [ ] Tests added for new features
- [ ] All tests passing

## Related Issues
Closes #123
Related to #456

## Screenshots (if applicable)
[Add screenshots here]

## Additional Notes
[Any additional information]
```

### Reviewing Pull Requests

```bash
# Review checklist:
# 1. Read description and requirements
# 2. Check related issues/tickets
# 3. Review code changes
# 4. Add inline comments
# 5. Test locally (if needed)
# 6. Check pipeline status
# 7. Approve or request changes

# Add inline comment:
# Click on line number → Add comment

# Comment types:
# - General comment
# - Task (requires resolution)
# - Suggestion (can be applied directly)

# Review actions:
Approve          # Code looks good
Request changes  # Issues found
Comment          # General feedback

# Merge strategies:
Merge commit           # Preserves all commits
Squash                # Single commit
Fast-forward (if possible)
```

### Pull Request Tasks

```bash
# Add task in comment:
- [ ] Update documentation
- [ ] Add unit tests
- [ ] Update changelog

# Task syntax in description:
## Tasks
- [x] Implement feature
- [x] Add tests
- [ ] Update docs (reviewer can track)

# Require tasks resolved before merge:
Repository settings → Merge checks
→ Enable "All tasks resolved"
```

### Default Reviewers

```bash
# Repository settings → Default reviewers

# Add by path pattern:
Pattern: src/**/*.js
Reviewers: frontend-team

Pattern: api/**/*.py
Reviewers: backend-team

Pattern: docker/**
Reviewers: devops-team

Pattern: **
Reviewers: tech-lead
Min approvals: 1

# CODEOWNERS file (alternative):
# .bitbucket/CODEOWNERS
*.js @frontend-team
*.py @backend-team
/docs/ @docs-team
* @tech-lead
```

### Merge Strategies

```bash
# Merge commit (preserves history)
# Creates merge commit with all feature commits
git merge --no-ff feature/branch

# Squash (clean history)
# Combines all commits into single commit
git merge --squash feature/branch
git commit -m "Feature: Add user authentication"

# Fast-forward (when possible)
# No merge commit, linear history
git merge --ff-only feature/branch

# Configure default strategy:
Repository settings → Merge strategies
→ Merge commit / Squash / Fast-forward only
→ Delete source branch after merge ✓
```

---

## Bitbucket Pipelines

### Enabling Pipelines

```bash
# 1. Navigate to repository
# 2. Click "Pipelines" in left menu
# 3. Click "Enable Pipelines"
# 4. Choose language template or start from scratch

# Pipelines run in Docker containers
# Configuration file: bitbucket-pipelines.yml
# Located in repository root
```

### Basic Pipeline Configuration

```yaml
# bitbucket-pipelines.yml

image: node:16

pipelines:
  default:
    - step:
        name: Build and Test
        caches:
          - node
        script:
          - npm install
          - npm test
          - npm run build
        artifacts:
          - dist/**

  branches:
    main:
      - step:
          name: Build
          script:
            - npm install
            - npm run build
          artifacts:
            - dist/**
      - step:
          name: Deploy to Production
          deployment: production
          script:
            - ./deploy.sh production

    develop:
      - step:
          name: Deploy to Staging
          deployment: staging
          script:
            - ./deploy.sh staging

  pull-requests:
    '**':
      - step:
          name: Test Pull Request
          script:
            - npm install
            - npm test
            - npm run lint

  tags:
    'v*':
      - step:
          name: Create Release
          script:
            - npm install
            - npm run build
            - ./create-release.sh
```

### Pipeline Steps

```yaml
# Step structure
pipelines:
  default:
    - step:
        name: Step Name
        image: custom-image:tag  # Override default image
        size: 2x                 # Double resources (2x, 4x, 8x)
        caches:
          - node
          - docker
        services:
          - docker
          - postgres
        script:
          - command1
          - command2
        artifacts:
          - build/**
          - reports/**
        after-script:
          - cleanup-command

# Parallel steps
pipelines:
  default:
    - parallel:
      - step:
          name: Unit Tests
          script:
            - npm run test:unit
      - step:
          name: Integration Tests
          script:
            - npm run test:integration
      - step:
          name: Linting
          script:
            - npm run lint
```

### Caching

```yaml
# Define custom caches
definitions:
  caches:
    npm: ~/.npm
    composer: ~/.composer/cache
    gradle: ~/.gradle/caches

pipelines:
  default:
    - step:
        name: Build
        caches:
          - node          # Built-in cache
          - npm           # Custom cache
        script:
          - npm ci
          - npm run build

# Built-in caches:
# - node (node_modules)
# - npm (~/.npm)
# - pip (~/.cache/pip)
# - composer (vendor/)
# - gradle (~/.gradle)
# - maven (~/.m2)
# - docker (Docker layer cache)
```

### Artifacts

```yaml
pipelines:
  default:
    - step:
        name: Build
        script:
          - npm run build
        artifacts:
          - dist/**           # Files to pass to next steps
          - coverage/**

    - step:
        name: Deploy
        script:
          - ls dist/          # Access artifacts from previous step
          - ./deploy.sh

# Download artifacts:
# Pipeline result → Artifacts → Download

# Artifact retention: 14 days
```

### Services

```yaml
definitions:
  services:
    postgres:
      image: postgres:14
      environment:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass
    redis:
      image: redis:7-alpine
    mysql:
      image: mysql:8
      environment:
        MYSQL_DATABASE: testdb
        MYSQL_ROOT_PASSWORD: root

pipelines:
  default:
    - step:
        name: Test with Database
        services:
          - postgres
          - redis
        script:
          - npm install
          - npm test

# Access service:
# Host: localhost or 127.0.0.1
# Port: Default service port
```

### Docker in Pipelines

```yaml
pipelines:
  default:
    - step:
        name: Build Docker Image
        services:
          - docker
        caches:
          - docker
        script:
          - docker build -t myapp:${BITBUCKET_COMMIT} .
          - docker tag myapp:${BITBUCKET_COMMIT} myapp:latest

          # Login to registry
          - echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

          # Push image
          - docker push myapp:${BITBUCKET_COMMIT}
          - docker push myapp:latest

# Docker Buildx (multi-platform)
pipelines:
  default:
    - step:
        name: Build Multi-Platform
        services:
          - docker
        script:
          - docker buildx create --use
          - docker buildx build \
              --platform linux/amd64,linux/arm64 \
              -t myapp:latest \
              --push .
```

---

## Pipeline Configuration

### Environment Variables

```yaml
# Repository variables (encrypted)
# Repository settings → Repository variables
# Name: API_KEY, Value: secret, Secured: ✓

# Deployment variables
# Repository settings → Deployments → Environment
# Name: AWS_ACCESS_KEY_ID, Value: xxx, Secured: ✓

# Use in pipeline:
pipelines:
  default:
    - step:
        name: Deploy
        script:
          - echo $API_KEY
          - echo $BITBUCKET_COMMIT
          - aws s3 sync dist/ s3://bucket/

# Built-in variables:
# BITBUCKET_BRANCH
# BITBUCKET_COMMIT
# BITBUCKET_BUILD_NUMBER
# BITBUCKET_REPO_SLUG
# BITBUCKET_WORKSPACE
# BITBUCKET_TAG (for tag pipelines)
# BITBUCKET_PR_ID (for PR pipelines)
```

### Pipeline Triggers

```yaml
# Branch-specific
pipelines:
  branches:
    main:
      - step:
          script: echo "Main branch"
    develop:
      - step:
          script: echo "Develop branch"
    feature/*:
      - step:
          script: echo "Feature branch"

# Pull requests
  pull-requests:
    '**':  # All PRs
      - step:
          script: npm test
    'feature/*':  # Feature branch PRs only
      - step:
          script: npm run test:full

# Tags
  tags:
    'v*':  # All tags starting with 'v'
      - step:
          script: ./release.sh
    'v*.*.*':  # Semantic version tags
      - step:
          script: ./publish.sh

# Custom triggers
  custom:
    deployment-prod:
      - step:
          script: ./deploy.sh production

    rollback:
      - step:
          script: ./rollback.sh

# Trigger custom pipeline:
# Pipelines → Run pipeline → Custom → Select pipeline
```

### Conditions

```yaml
pipelines:
  default:
    - step:
        name: Build
        script:
          - npm run build

    - step:
        name: Deploy to Staging
        condition:
          changesets:
            includePaths:
              - "src/**"
              - "package.json"
        deployment: staging
        script:
          - ./deploy.sh staging

    - step:
        name: Deploy to Production
        trigger: manual  # Requires manual trigger
        deployment: production
        script:
          - ./deploy.sh production

# Condition types:
# - changesets (file changes)
# - trigger: manual
# - trigger: automatic
```

### Definitions and Reuse

```yaml
definitions:
  steps:
    - step: &build-step
        name: Build Application
        caches:
          - node
        script:
          - npm ci
          - npm run build
        artifacts:
          - dist/**

    - step: &test-step
        name: Run Tests
        caches:
          - node
        script:
          - npm ci
          - npm test

  services:
    postgres:
      image: postgres:14
      environment:
        POSTGRES_DB: testdb
        POSTGRES_PASSWORD: password

pipelines:
  default:
    - step: *build-step
    - step: *test-step

  branches:
    main:
      - step: *build-step
      - step:
          <<: *test-step
          name: Run Full Test Suite
          script:
            - npm ci
            - npm run test:all
      - step:
          name: Deploy
          deployment: production
          script:
            - ./deploy.sh
```

### Multi-Step Pipelines

```yaml
pipelines:
  default:
    # Stage 1: Build
    - step:
        name: Install Dependencies
        caches:
          - node
        script:
          - npm ci
        artifacts:
          - node_modules/**

    # Stage 2: Parallel Testing
    - parallel:
      - step:
          name: Unit Tests
          script:
            - npm run test:unit
      - step:
          name: Integration Tests
          services:
            - postgres
          script:
            - npm run test:integration
      - step:
          name: E2E Tests
          script:
            - npm run test:e2e
      - step:
          name: Lint & Format
          script:
            - npm run lint
            - npm run format:check

    # Stage 3: Build
    - step:
        name: Build Application
        script:
          - npm run build
        artifacts:
          - dist/**

    # Stage 4: Security Scan
    - step:
        name: Security Scan
        script:
          - npm audit
          - npx snyk test

    # Stage 5: Deploy
    - step:
        name: Deploy to Staging
        deployment: staging
        trigger: manual
        script:
          - ./deploy.sh staging
```

---

## Advanced Pipeline Features

### Docker Compose in Pipelines

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - postgres
      - redis
  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: testdb
      POSTGRES_PASSWORD: password
  redis:
    image: redis:7-alpine

# bitbucket-pipelines.yml
pipelines:
  default:
    - step:
        name: Test with Docker Compose
        services:
          - docker
        script:
          - docker-compose up -d
          - sleep 10
          - docker-compose exec -T app npm test
          - docker-compose down
```

### Building Multi-Stage Pipelines

```yaml
# Build → Test → Security → Deploy
definitions:
  steps:
    - step: &build
        name: Build
        script:
          - npm ci
          - npm run build
        artifacts:
          - dist/**

    - step: &unit-tests
        name: Unit Tests
        script:
          - npm run test:unit

    - step: &integration-tests
        name: Integration Tests
        services:
          - postgres
        script:
          - npm run test:integration

    - step: &security-scan
        name: Security Scan
        script:
          - npm audit --audit-level=moderate
          - npx snyk test

    - step: &deploy-staging
        name: Deploy to Staging
        deployment: staging
        script:
          - ./deploy.sh staging

    - step: &deploy-production
        name: Deploy to Production
        deployment: production
        trigger: manual
        script:
          - ./deploy.sh production

pipelines:
  default:
    - step: *build
    - parallel:
        - step: *unit-tests
        - step: *integration-tests
    - step: *security-scan

  branches:
    develop:
      - step: *build
      - parallel:
          - step: *unit-tests
          - step: *integration-tests
      - step: *security-scan
      - step: *deploy-staging

    main:
      - step: *build
      - parallel:
          - step: *unit-tests
          - step: *integration-tests
      - step: *security-scan
      - step: *deploy-staging
      - step: *deploy-production
```

### Pipeline Schedules

```bash
# Repository settings → Pipelines → Schedules

# Create schedule:
Name: Nightly Build
Branch: develop
Cron: 0 2 * * *  # 2 AM daily
Enabled: ✓

# Pipeline configuration:
pipelines:
  branches:
    develop:
      - step:
          name: Scheduled Build
          script:
            - npm run build
            - npm run test:full
            - ./generate-report.sh
```

### Pipeline Reports

```yaml
# Test reports
pipelines:
  default:
    - step:
        name: Test with Reports
        script:
          - npm test -- --coverage --reporters=default --reporters=jest-junit
        artifacts:
          - coverage/**
          - test-results/**

# Supported formats:
# - JUnit XML
# - Code coverage (Clover, Cobertura)
# - SARIF (security)

# Upload test results:
# Results appear in Pipeline → Test tab
```

### Pipeline Integrations

```yaml
# Slack notifications
pipelines:
  default:
    - step:
        name: Build and Notify
        script:
          - npm run build
        after-script:
          - |
            if [ $BITBUCKET_EXIT_CODE -eq 0 ]; then
              curl -X POST -H 'Content-type: application/json' \
                --data '{"text":"Build succeeded!"}' \
                $SLACK_WEBHOOK_URL
            else
              curl -X POST -H 'Content-type: application/json' \
                --data '{"text":"Build failed!"}' \
                $SLACK_WEBHOOK_URL
            fi

# Email notifications:
# User settings → Email notifications
# - Pipeline failed
# - Pipeline succeeded
# - Deployment succeeded/failed

# Jira integration:
# Commit message: "PROJECT-123 Fix bug"
# Automatically links commit to Jira issue
```

### Self-Hosted Runners

```bash
# Install runner on your infrastructure
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /tmp:/tmp \
  -e ACCOUNT_UUID={account-uuid} \
  -e RUNNER_UUID={runner-uuid} \
  -e OAUTH_CLIENT_ID={client-id} \
  -e OAUTH_CLIENT_SECRET={client-secret} \
  -e WORKING_DIRECTORY=/tmp \
  --name runner1 \
  docker-public.packages.atlassian.com/sox/atlassian/bitbucket-pipelines-runner

# Configure in repository:
# Repository settings → Pipelines → Runners

# Use in pipeline:
pipelines:
  default:
    - step:
        runs-on:
          - self.hosted
          - linux
        script:
          - ./build.sh
```

---

## Deployments

### Deployment Environments

```bash
# Repository settings → Deployments

# Create environments:
# - Test
# - Staging
# - Production

# Environment settings:
Name: production
Deployment permissions: (specific users/groups)
Environment variables: (encrypted)

# Environment variables:
AWS_ACCESS_KEY_ID: xxx (secured)
AWS_SECRET_ACCESS_KEY: xxx (secured)
AWS_REGION: us-east-1
```

### Deployment Pipeline

```yaml
pipelines:
  branches:
    main:
      - step:
          name: Build
          script:
            - npm run build
          artifacts:
            - dist/**

      - step:
          name: Deploy to Test
          deployment: test
          script:
            - ./deploy.sh test

      - step:
          name: Deploy to Staging
          deployment: staging
          script:
            - ./deploy.sh staging

      - step:
          name: Deploy to Production
          deployment: production
          trigger: manual  # Requires approval
          script:
            - ./deploy.sh production

# View deployments:
# Repository → Deployments
# Shows deployment history per environment
```

### Deployment Scripts

```bash
# deploy.sh
#!/bin/bash

ENVIRONMENT=$1

echo "Deploying to $ENVIRONMENT..."

case $ENVIRONMENT in
  test)
    S3_BUCKET="my-app-test"
    CLOUDFRONT_ID="TEST123"
    ;;
  staging)
    S3_BUCKET="my-app-staging"
    CLOUDFRONT_ID="STAGE123"
    ;;
  production)
    S3_BUCKET="my-app-prod"
    CLOUDFRONT_ID="PROD123"
    ;;
esac

# Deploy to S3
aws s3 sync dist/ s3://$S3_BUCKET/ --delete

# Invalidate CloudFront
aws cloudfront create-invalidation \
  --distribution-id $CLOUDFRONT_ID \
  --paths "/*"

echo "Deployment to $ENVIRONMENT completed!"
```

### Deployment with AWS

```yaml
# AWS deployment pipeline
pipelines:
  branches:
    main:
      - step:
          name: Build
          script:
            - npm ci
            - npm run build
          artifacts:
            - dist/**

      - step:
          name: Deploy to S3
          deployment: production
          script:
            # Sync to S3
            - pipe: atlassian/aws-s3-deploy:1.1.0
              variables:
                AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
                AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
                AWS_DEFAULT_REGION: 'us-east-1'
                S3_BUCKET: 'my-app-bucket'
                LOCAL_PATH: 'dist'

            # Invalidate CloudFront
            - pipe: atlassian/aws-cloudfront-invalidate:0.6.0
              variables:
                AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
                AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
                AWS_DEFAULT_REGION: 'us-east-1'
                DISTRIBUTION_ID: 'E1234567890'
```

### Deployment with Kubernetes

```yaml
pipelines:
  branches:
    main:
      - step:
          name: Build Docker Image
          services:
            - docker
          script:
            - docker build -t myapp:${BITBUCKET_BUILD_NUMBER} .
            - echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
            - docker push myapp:${BITBUCKET_BUILD_NUMBER}

      - step:
          name: Deploy to Kubernetes
          deployment: production
          image: atlassian/pipelines-kubectl
          script:
            # Set kubectl context
            - echo $KUBE_CONFIG | base64 -d > kubeconfig
            - export KUBECONFIG=kubeconfig

            # Update image in deployment
            - kubectl set image deployment/myapp \
                myapp=myapp:${BITBUCKET_BUILD_NUMBER} \
                -n production

            # Wait for rollout
            - kubectl rollout status deployment/myapp -n production
```

### Deployment with Helm

```yaml
pipelines:
  branches:
    main:
      - step:
          name: Deploy with Helm
          deployment: production
          image: alpine/helm:3.13.0
          script:
            # Configure kubectl
            - echo $KUBE_CONFIG | base64 -d > kubeconfig
            - export KUBECONFIG=kubeconfig

            # Deploy with Helm
            - helm upgrade --install myapp ./helm/myapp \
                --namespace production \
                --create-namespace \
                --set image.tag=${BITBUCKET_BUILD_NUMBER} \
                --set ingress.host=app.example.com \
                --wait \
                --timeout 5m
```

---

## Code Review

### Code Review Process

```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Make changes and commit
git add .
git commit -m "Add new feature"
git push -u origin feature/new-feature

# 3. Create pull request
# Source: feature/new-feature
# Destination: main
# Reviewers: team-members
# Description: Detailed explanation

# 4. Reviewers examine code:
# - Read changes
# - Add inline comments
# - Ask questions
# - Request changes or approve

# 5. Address feedback:
git add .
git commit -m "Address review feedback"
git push

# 6. Merge when approved
# (automatically or manually)

# 7. Delete feature branch
git branch -d feature/new-feature
git push origin --delete feature/new-feature
```

### Review Comments

```bash
# Types of comments:

# 1. General comment
# Non-blocking feedback

# 2. Task
- [ ] Please add error handling here
- [ ] Update the documentation

# 3. Suggestion (inline)
# Click "Add suggestion"
# Reviewer can propose exact code change
# Author can apply with one click

# 4. Blocker
# Critical issue that must be resolved
```

### Code Review Checklist

```markdown
# .bitbucket/pull_request_template.md

## Code Review Checklist

### Functionality
- [ ] Code works as intended
- [ ] Edge cases handled
- [ ] Error handling implemented
- [ ] No hardcoded values

### Code Quality
- [ ] Follows coding standards
- [ ] Well-structured and readable
- [ ] No code duplication
- [ ] Functions are focused and small
- [ ] Meaningful variable/function names

### Testing
- [ ] Unit tests added
- [ ] Integration tests added
- [ ] All tests passing
- [ ] Test coverage acceptable

### Security
- [ ] No sensitive data exposed
- [ ] Input validation implemented
- [ ] SQL injection prevented
- [ ] XSS protection in place
- [ ] Authentication/authorization checked

### Performance
- [ ] No obvious performance issues
- [ ] Database queries optimized
- [ ] Caching used appropriately
- [ ] No unnecessary API calls

### Documentation
- [ ] Code comments for complex logic
- [ ] README updated
- [ ] API documentation updated
- [ ] Changelog updated

### Dependencies
- [ ] New dependencies justified
- [ ] Dependency versions pinned
- [ ] No security vulnerabilities
```

### Inline Code Suggestions

```bash
# Reviewer adds suggestion:
# Original code:
function add(a, b) {
  return a + b;
}

# Suggestion:
function add(a, b) {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw new TypeError('Arguments must be numbers');
  }
  return a + b;
}

# Author can:
# - Accept suggestion (applies change automatically)
# - Reject suggestion with explanation
# - Modify and apply
```

### Merge Checks

```bash
# Repository settings → Merge checks

# Enable checks:
✓ Minimum number of approvals: 2
✓ Default reviewer must approve
✓ Check that all tasks are resolved
✓ Check that all builds pass
✓ Reset approvals when source branch changes

# Branch permissions:
# Require passing builds before merge
# Prevent changes without pull request
```

---

## Access Management

### User Permissions

```bash
# Workspace permissions:
Admins          # Full access
Members         # Can create repos
Collaborators   # Limited access (read-only)

# Repository permissions:
Admin           # Full control
Write           # Push and merge
Read            # Clone and view

# Add users:
# Repository settings → Access management → Add user

# User groups:
# Workspace settings → User groups
# Create groups for teams (frontend, backend, devops)
# Assign group permissions to repositories
```

### Branch Permissions

```bash
# Repository settings → Branch permissions

# Protect main branch:
Branch: main
Type: Prevent deletion
Exempt users: (none)

Branch: main
Type: Prevent rewriting history
Exempt users: (none)

Branch: main
Type: Require pull request approvals
Number of approvals: 2
Exempt users: (none)

Branch: main
Type: Require passing builds
Build: bitbucket-pipelines.yml

# Protect release branches:
Branch pattern: release/*
Type: Restrict who can push
Users/Groups: release-managers

# Pattern examples:
main                  # Exact match
release/*            # Wildcard
**/*-hotfix          # Any hotfix branch
```

### SSH Keys

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Add to Bitbucket:
# Personal settings → SSH keys → Add key
# Paste contents of ~/.ssh/id_ed25519.pub

# Repository access keys (read-only):
# Repository settings → Access keys
# Use for CI/CD systems to clone repo

# Test SSH connection:
ssh -T git@bitbucket.org
```

### Access Tokens

```bash
# Personal settings → App passwords

# Create app password:
Name: CI/CD Pipeline
Permissions:
- Repositories: Read, Write
- Pull requests: Read, Write
- Pipelines: Read, Write

# Use in scripts:
git clone https://username:app-password@bitbucket.org/workspace/repo.git

# Use in Pipelines:
script:
  - git clone https://$BITBUCKET_USERNAME:$BITBUCKET_APP_PASSWORD@bitbucket.org/workspace/repo.git
```

### IP Allowlisting

```bash
# Workspace settings → IP allowlisting

# Add allowed IPs:
10.0.0.0/8
192.168.1.0/24
203.0.113.5

# Applies to:
# - Git operations (push, pull)
# - Web UI access
# - API calls

# Exceptions:
# Bitbucket Pipelines (always allowed)
```

---

## Integrations

### Jira Integration

```bash
# Workspace settings → Integrations → Jira

# Enable integration:
1. Connect Jira workspace
2. Allow permissions

# Usage in commits:
git commit -m "PROJECT-123 Fix login bug"

# Automatic links:
# - Commits appear in Jira issues
# - Jira issues appear in commit details
# - Build status shown in Jira

# Smart commits:
git commit -m "PROJECT-123 #time 2h #comment Fixed bug"
git commit -m "PROJECT-123 #close"  # Close issue

# PR automation:
# - Link to Jira issue
# - Transition issue status
# - Add PR link to issue
```

### Slack Integration

```bash
# Workspace settings → Integrations → Slack

# Setup:
1. Install Bitbucket app in Slack
2. Connect workspace
3. Configure notifications

# Notifications:
# - Pull request created
# - Pull request merged
# - Build succeeded/failed
# - Deployment completed

# Channel notifications:
/bitbucket subscribe username/repository

# Unsubscribe:
/bitbucket unsubscribe username/repository

# View PRs in Slack:
/bitbucket prs

# Custom webhook:
script:
  - |
    curl -X POST \
      -H 'Content-type: application/json' \
      --data "{\"text\":\"Build ${BITBUCKET_BUILD_NUMBER} completed\"}" \
      $SLACK_WEBHOOK_URL
```

### Trello Integration

```bash
# Power-Up in Trello:
1. Open Trello board
2. Add Power-Up: Bitbucket
3. Connect workspace
4. Attach commits/PRs to cards

# Commit message:
git commit -m "Fix bug [Trello card ID]"

# Automatic attachment of commits to cards
```

### Microsoft Teams

```bash
# Install Bitbucket connector in Teams

# Configure notifications:
1. Teams channel → Connectors
2. Add Bitbucket connector
3. Configure repository
4. Select events

# Events:
# - Push
# - Pull request
# - Build status
# - Deployment
```

### Third-Party Integrations

```bash
# SonarQube
pipelines:
  default:
    - step:
        name: SonarQube Scan
        script:
          - pipe: sonarsource/sonarqube-scan:1.0.0
            variables:
              SONAR_HOST_URL: $SONAR_HOST_URL
              SONAR_TOKEN: $SONAR_TOKEN

# Snyk Security
pipelines:
  default:
    - step:
        name: Security Scan
        script:
          - pipe: snyk/snyk-scan:0.7.1
            variables:
              SNYK_TOKEN: $SNYK_TOKEN
              LANGUAGE: "node"

# Datadog
pipelines:
  default:
    - step:
        name: Deploy
        script:
          - ./deploy.sh
        after-script:
          - |
            curl -X POST "https://api.datadoghq.com/api/v1/events" \
              -H "Content-Type: application/json" \
              -H "DD-API-KEY: ${DD_API_KEY}" \
              -d @- << EOF
            {
              "title": "Deployment",
              "text": "Deployed build ${BITBUCKET_BUILD_NUMBER}",
              "tags": ["environment:production", "service:myapp"]
            }
            EOF
```

---

## API and Automation

### Bitbucket REST API

```bash
# Authentication with app password
curl -u username:app-password \
  https://api.bitbucket.org/2.0/repositories/workspace

# List repositories
curl -u username:app-password \
  https://api.bitbucket.org/2.0/repositories/workspace

# Get repository details
curl -u username:app-password \
  https://api.bitbucket.org/2.0/repositories/workspace/repo-slug

# List branches
curl -u username:app-password \
  https://api.bitbucket.org/2.0/repositories/workspace/repo-slug/refs/branches

# List pull requests
curl -u username:app-password \
  https://api.bitbucket.org/2.0/repositories/workspace/repo-slug/pullrequests

# Create pull request
curl -X POST -u username:app-password \
  -H "Content-Type: application/json" \
  -d '{
    "title": "New feature",
    "source": {"branch": {"name": "feature/new"}},
    "destination": {"branch": {"name": "main"}},
    "description": "Description here",
    "reviewers": [{"uuid": "{user-uuid}"}]
  }' \
  https://api.bitbucket.org/2.0/repositories/workspace/repo-slug/pullrequests

# Get pipeline status
curl -u username:app-password \
  https://api.bitbucket.org/2.0/repositories/workspace/repo-slug/pipelines/

# Trigger pipeline
curl -X POST -u username:app-password \
  -H "Content-Type: application/json" \
  -d '{
    "target": {
      "ref_type": "branch",
      "type": "pipeline_ref_target",
      "ref_name": "main"
    }
  }' \
  https://api.bitbucket.org/2.0/repositories/workspace/repo-slug/pipelines/
```

### Python API Client

```python
# Install library
# pip install atlassian-python-api

from atlassian.bitbucket import Cloud

# Initialize client
bitbucket = Cloud(
    url='https://api.bitbucket.org',
    username='username',
    password='app-password'
)

# List repositories
repos = bitbucket.workspaces.get('workspace-name').repositories.each()
for repo in repos:
    print(repo['name'])

# Get repository
repo = bitbucket.workspaces.get('workspace-name') \
    .repositories.get('repo-slug')

# List pull requests
prs = repo.pullrequests.each(state='OPEN')
for pr in prs:
    print(f"PR #{pr['id']}: {pr['title']}")

# Create pull request
pr = repo.pullrequests.create(
    title='New Feature',
    source_branch='feature/new',
    destination_branch='main',
    description='Detailed description',
    close_source_branch=True
)

# Approve pull request
pr.approve()

# Merge pull request
pr.merge()

# List commits
commits = repo.commits.each()
for commit in commits:
    print(f"{commit['hash']}: {commit['message']}")

# Trigger pipeline
pipeline = repo.pipelines.trigger(
    ref_type='branch',
    ref_name='main'
)
```

### Bash Automation Scripts

```bash
#!/bin/bash
# create-pr.sh - Create pull request via API

WORKSPACE="myworkspace"
REPO="myrepo"
SOURCE_BRANCH="feature/new-feature"
DEST_BRANCH="main"
TITLE="Add new feature"
DESCRIPTION="This PR adds a new feature"

USERNAME="username"
APP_PASSWORD="app-password"

curl -X POST \
  -u "$USERNAME:$APP_PASSWORD" \
  -H "Content-Type: application/json" \
  -d "{
    \"title\": \"$TITLE\",
    \"description\": \"$DESCRIPTION\",
    \"source\": {
      \"branch\": {
        \"name\": \"$SOURCE_BRANCH\"
      }
    },
    \"destination\": {
      \"branch\": {
        \"name\": \"$DEST_BRANCH\"
      }
    },
    \"close_source_branch\": true
  }" \
  "https://api.bitbucket.org/2.0/repositories/$WORKSPACE/$REPO/pullrequests"
```

### Webhooks Automation

```python
# Flask webhook receiver
from flask import Flask, request
import hmac
import hashlib

app = Flask(__name__)
WEBHOOK_SECRET = 'your-secret'

@app.route('/webhook', methods=['POST'])
def webhook():
    # Verify signature
    signature = request.headers.get('X-Hub-Signature')
    mac = hmac.new(
        WEBHOOK_SECRET.encode(),
        request.data,
        hashlib.sha256
    )

    if not hmac.compare_digest(signature, 'sha256=' + mac.hexdigest()):
        return 'Invalid signature', 403

    data = request.json
    event = request.headers.get('X-Event-Key')

    # Handle different events
    if event == 'repo:push':
        handle_push(data)
    elif event == 'pullrequest:created':
        handle_pr_created(data)
    elif event == 'pullrequest:fulfilled':
        handle_pr_merged(data)

    return 'OK', 200

def handle_push(data):
    branch = data['push']['changes'][0]['new']['name']
    print(f"Push to branch: {branch}")

def handle_pr_created(data):
    pr = data['pullrequest']
    print(f"PR created: #{pr['id']} - {pr['title']}")

def handle_pr_merged(data):
    pr = data['pullrequest']
    print(f"PR merged: #{pr['id']}")

if __name__ == '__main__':
    app.run(port=5000)
```

---

## Bitbucket Server/Data Center

### Installation (Server)

```bash
# System requirements:
# - Java 11 or 17
# - PostgreSQL, MySQL, or Oracle
# - 4GB RAM minimum

# Download Bitbucket Server
wget https://www.atlassian.com/software/bitbucket/downloads/binary/atlassian-bitbucket-8.0.0-x64.bin

# Install
chmod +x atlassian-bitbucket-8.0.0-x64.bin
sudo ./atlassian-bitbucket-8.0.0-x64.bin

# Configure database (PostgreSQL example)
sudo -u postgres psql
CREATE DATABASE bitbucket;
CREATE USER bitbucketuser WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE bitbucket TO bitbucketuser;

# Start Bitbucket
sudo systemctl start bitbucket
sudo systemctl enable bitbucket

# Access: http://localhost:7990
```

### Server Configuration

```bash
# bitbucket.properties
server.port=7990
server.context-path=/bitbucket
server.scheme=https
server.secure=true

# JVM settings (setenv.sh)
JVM_MINIMUM_MEMORY="2048m"
JVM_MAXIMUM_MEMORY="4096m"
JVM_SUPPORT_RECOMMENDED_ARGS="-Datlassian.plugins.enable.wait=300"

# HTTPS configuration
server.ssl.enabled=true
server.ssl.key-store=/path/to/keystore
server.ssl.key-store-password=changeit
```

### Server Administration

```bash
# Backup
cd /var/atlassian/application-data/bitbucket
./bitbucket-backup.sh

# Restore
./bitbucket-restore.sh /path/to/backup.zip

# User management
# Admin → Users → Create user
# Admin → Groups → Create group

# Project permissions
# Project settings → Permissions
# - Project admin
# - Project write
# - Project read

# Global permissions
# Admin → Global permissions
# - System admin
# - Admin
# - Project create

# License management
# Admin → License
# Upload new license
```

### Smart Mirroring

```bash
# Primary server: US
# Mirror server: EU (read-only)

# Configure mirror:
Admin → Bitbucket Data Center → Smart Mirroring
→ Add mirror

# Benefits:
# - Faster clone/fetch for geographically distributed teams
# - Read operations from nearest mirror
# - Write operations always go to primary
```

### Data Center (High Availability)

```bash
# Load balancer configuration
# HAProxy example:

frontend bitbucket_front
    bind *:443 ssl crt /etc/ssl/certs/bitbucket.pem
    default_backend bitbucket_back

backend bitbucket_back
    balance roundrobin
    option httpchk GET /status
    server bitbucket1 10.0.1.10:7990 check
    server bitbucket2 10.0.1.11:7990 check
    server bitbucket3 10.0.1.12:7990 check

# Shared filesystem (NFS)
# All nodes share:
# - Git repositories
# - Search index
# - Attachments

# Database clustering
# PostgreSQL with replication
# or
# Oracle RAC
```

---

## Troubleshooting

### Common Issues

```bash
# 1. Pipeline fails with "Permission denied"
# Solution: Check file permissions in repository
git update-index --chmod=+x script.sh
git commit -m "Make script executable"
git push

# 2. Pipeline timeout
# Solution: Increase step timeout
pipelines:
  default:
    - step:
        script:
          - long-running-command
        max-time: 120  # Minutes (default: 120)

# 3. Out of memory in pipeline
# Solution: Use larger build size
pipelines:
  default:
    - step:
        size: 2x  # 4GB RAM (default: 1x = 4GB)
        script:
          - memory-intensive-command

# 4. Docker service not starting
# Solution: Check service definition
definitions:
  services:
    postgres:
      image: postgres:14
      environment:
        POSTGRES_PASSWORD: password
      # Wait for service to be ready
pipelines:
  default:
    - step:
        services:
          - postgres
        script:
          - sleep 10  # Wait for postgres
          - npm test

# 5. Git LFS files not downloading
# Solution: Configure LFS in pipeline
pipelines:
  default:
    - step:
        script:
          - git lfs install
          - git lfs pull
          - npm run build

# 6. Artifacts not found in next step
# Solution: Check artifacts path
pipelines:
  default:
    - step:
        name: Build
        script:
          - npm run build
        artifacts:
          - dist/**  # Must match actual build output
    - step:
        name: Deploy
        script:
          - ls -la dist/  # Verify artifacts available
```

### Pipeline Debugging

```bash
# Enable debug mode
pipelines:
  default:
    - step:
        script:
          - set -x  # Enable bash debug output
          - npm install
          - npm test
          - set +x  # Disable

# Print environment variables
pipelines:
  default:
    - step:
        script:
          - env | sort
          - echo "Branch: $BITBUCKET_BRANCH"
          - echo "Commit: $BITBUCKET_COMMIT"

# Interactive debugging (local)
# Use Bitbucket Pipelines runner locally:
docker run -it \
  -v $(pwd):/app \
  -w /app \
  node:16 \
  bash

# Run pipeline steps manually:
npm install
npm test

# Check logs
# Pipeline → View logs
# Download logs for offline analysis
```

### Performance Issues

```bash
# 1. Slow clone times
# Solution: Use shallow clone
clone:
  depth: 1  # Shallow clone (faster)

# 2. Slow builds
# Solution: Use caching effectively
definitions:
  caches:
    npm: ~/.npm

pipelines:
  default:
    - step:
        caches:
          - node
          - npm
        script:
          - npm ci  # Use ci instead of install

# 3. Large repository
# Solution: Use Git LFS for large files
git lfs track "*.zip"
git lfs track "*.tar.gz"
git add .gitattributes
git commit -m "Track large files with LFS"

# 4. Parallel steps not running
# Solution: Check repository limits
# Settings → Pipelines → Settings
# Max concurrent steps per repository: 10
```

### Access Issues

```bash
# 1. SSH authentication failed
# Solution: Check SSH key
ssh-keygen -t ed25519 -C "email@example.com"
cat ~/.ssh/id_ed25519.pub
# Add to Personal settings → SSH keys

# Test connection:
ssh -T git@bitbucket.org

# 2. Permission denied (push)
# Solution: Check repository permissions
# Repository → Settings → Access management
# Verify user has Write access

# 3. Pipeline can't clone private repository
# Solution: Use SSH key or access token
pipelines:
  default:
    - step:
        script:
          - git clone git@bitbucket.org:workspace/private-repo.git
          # or
          - git clone https://x-token-auth:${BITBUCKET_ACCESS_TOKEN}@bitbucket.org/workspace/private-repo.git

# 4. Branch permission error
# Solution: Check branch permissions
# Repository → Settings → Branch permissions
# Verify user is not restricted
```

---

## Best Practices

### Repository Organization

```bash
# Workspace structure:
mycompany (workspace)
├── frontend (project)
│   ├── website
│   ├── admin-panel
│   └── mobile-app
├── backend (project)
│   ├── api-gateway
│   ├── user-service
│   └── payment-service
└── infrastructure (project)
    ├── terraform
    ├── kubernetes
    └── ansible

# Repository naming:
# Good: user-service, payment-api, frontend-web
# Bad: repo1, test, new-project

# README.md structure:
# Project Title
## Description
## Prerequisites
## Installation
## Usage
## Configuration
## Testing
## Deployment
## Contributing
## License
```

### Branching Best Practices

```bash
# Branch naming conventions:
feature/JIRA-123-add-login
bugfix/JIRA-456-fix-memory-leak
hotfix/critical-security-patch
release/v1.2.0
chore/update-dependencies

# Commit message format:
# <type>(<scope>): <subject>
#
# <body>
#
# <footer>

# Example:
feat(auth): add OAuth2 authentication

Implement OAuth2 authentication flow using Google provider.
Includes login, logout, and token refresh functionality.

Closes JIRA-123

# Types:
# feat: New feature
# fix: Bug fix
# docs: Documentation
# style: Formatting
# refactor: Code restructuring
# test: Tests
# chore: Maintenance
```

### Pipeline Best Practices

```yaml
# 1. Use caching effectively
definitions:
  caches:
    npm: ~/.npm
    pip: ~/.cache/pip

pipelines:
  default:
    - step:
        caches:
          - node
          - npm
        script:
          - npm ci

# 2. Fail fast
pipelines:
  default:
    - parallel:
      - step:
          name: Lint
          script:
            - npm run lint
      - step:
          name: Type Check
          script:
            - npm run type-check
    # Only continue if above passes
    - step:
        name: Build
        script:
          - npm run build

# 3. Use specific Docker images
image: node:16.20-alpine  # Specific version
# Not: node:latest

# 4. Keep secrets secure
# Use repository variables (encrypted)
# Never hardcode secrets

# 5. Optimize Docker builds
# Use .dockerignore
# Multi-stage builds
# Layer caching

# 6. Set resource limits
pipelines:
  default:
    - step:
        size: 2x
        max-time: 60
        script:
          - npm test
```

### Security Best Practices

```bash
# 1. Branch protection
# Enable branch permissions on main/master
# Require PR reviews
# Require passing builds

# 2. Secret management
# Use repository variables (encrypted)
# Use deployment environment variables
# Rotate secrets regularly
# Never commit secrets to repository

# 3. Access control
# Minimum necessary permissions
# Regular access audits
# Remove inactive users
# Use groups for team permissions

# 4. Code scanning
pipelines:
  default:
    - step:
        name: Security Scan
        script:
          - npm audit
          - npx snyk test
          - |
            if [ $? -ne 0 ]; then
              echo "Security vulnerabilities found!"
              exit 1
            fi

# 5. Dependency management
# Pin dependency versions
# Regular dependency updates
# Automated security patches

# 6. Code review
# Mandatory peer reviews
# Security-focused review checklist
# Automated security checks in CI
```

### Collaboration Best Practices

```bash
# 1. Pull request guidelines
# - Clear title and description
# - Link to issue/ticket
# - Self-review before requesting review
# - Keep PRs small and focused
# - Respond to feedback promptly

# 2. Code review standards
# - Review within 24 hours
# - Constructive feedback
# - Ask questions, don't demand
# - Approve when satisfied
# - Follow up on unresolved comments

# 3. Documentation
# - Update README for changes
# - Document complex logic
# - Keep API docs current
# - Maintain changelog

# 4. Testing
# - Write tests for new features
# - Maintain test coverage
# - Fix failing tests immediately
# - Don't skip CI checks

# 5. Communication
# - Use descriptive commit messages
# - Reference issues in commits
# - Update ticket status
# - Notify team of breaking changes
```

---

## Practical Examples

### Example 1: Node.js CI/CD Pipeline

```yaml
# bitbucket-pipelines.yml
image: node:16

definitions:
  caches:
    npm: ~/.npm

  steps:
    - step: &install
        name: Install Dependencies
        caches:
          - node
          - npm
        script:
          - npm ci
        artifacts:
          - node_modules/**

    - step: &lint
        name: Lint
        script:
          - npm run lint

    - step: &test
        name: Test
        script:
          - npm test -- --coverage
        artifacts:
          - coverage/**

    - step: &build
        name: Build
        script:
          - npm run build
        artifacts:
          - dist/**

    - step: &security
        name: Security Scan
        script:
          - npm audit --audit-level=moderate
          - npx snyk test --severity-threshold=high

pipelines:
  default:
    - step: *install
    - parallel:
        - step: *lint
        - step: *test
    - step: *security

  branches:
    develop:
      - step: *install
      - parallel:
          - step: *lint
          - step: *test
      - step: *security
      - step: *build
      - step:
          name: Deploy to Staging
          deployment: staging
          script:
            - npm run deploy:staging

    main:
      - step: *install
      - parallel:
          - step: *lint
          - step: *test
      - step: *security
      - step: *build
      - step:
          name: Deploy to Production
          deployment: production
          trigger: manual
          script:
            - npm run deploy:production

  pull-requests:
    '**':
      - step: *install
      - parallel:
          - step: *lint
          - step: *test
      - step: *security
```

### Example 2: Python Django Application

```yaml
# bitbucket-pipelines.yml
image: python:3.11

definitions:
  services:
    postgres:
      image: postgres:15
      environment:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass

    redis:
      image: redis:7-alpine

pipelines:
  default:
    - step:
        name: Test
        services:
          - postgres
          - redis
        caches:
          - pip
        script:
          # Install dependencies
          - pip install -r requirements.txt
          - pip install -r requirements-dev.txt

          # Wait for services
          - sleep 5

          # Run tests
          - python manage.py test
          - coverage run --source='.' manage.py test
          - coverage report

          # Linting
          - flake8 .
          - black --check .
          - mypy .
        artifacts:
          - htmlcov/**

  branches:
    main:
      - step:
          name: Test
          services:
            - postgres
            - redis
          caches:
            - pip
          script:
            - pip install -r requirements.txt
            - python manage.py test

      - step:
          name: Build Docker Image
          services:
            - docker
          script:
            - docker build -t myapp:${BITBUCKET_BUILD_NUMBER} .
            - echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
            - docker push myapp:${BITBUCKET_BUILD_NUMBER}

      - step:
          name: Deploy
          deployment: production
          script:
            - pipe: atlassian/ssh-run:0.4.1
              variables:
                SSH_USER: 'deploy'
                SERVER: $DEPLOY_SERVER
                SSH_KEY: $SSH_KEY
                COMMAND: './deploy.sh ${BITBUCKET_BUILD_NUMBER}'
```

### Example 3: Microservices Monorepo

```yaml
# bitbucket-pipelines.yml
image: node:16

definitions:
  steps:
    - step: &test-service
        name: Test Service
        caches:
          - node
        script:
          - cd $SERVICE_PATH
          - npm ci
          - npm test

    - step: &build-service
        name: Build Service
        services:
          - docker
        script:
          - cd $SERVICE_PATH
          - docker build -t $SERVICE_NAME:${BITBUCKET_BUILD_NUMBER} .
          - echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
          - docker push $SERVICE_NAME:${BITBUCKET_BUILD_NUMBER}

pipelines:
  default:
    - parallel:
        - step:
            <<: *test-service
            name: Test User Service
            environment:
              SERVICE_PATH: services/user
        - step:
            <<: *test-service
            name: Test Order Service
            environment:
              SERVICE_PATH: services/order
        - step:
            <<: *test-service
            name: Test Payment Service
            environment:
              SERVICE_PATH: services/payment

  branches:
    main:
      - parallel:
          - step:
              <<: *build-service
              name: Build User Service
              environment:
                SERVICE_PATH: services/user
                SERVICE_NAME: user-service
          - step:
              <<: *build-service
              name: Build Order Service
              environment:
                SERVICE_PATH: services/order
                SERVICE_NAME: order-service
          - step:
              <<: *build-service
              name: Build Payment Service
              environment:
                SERVICE_PATH: services/payment
                SERVICE_NAME: payment-service

      - step:
          name: Deploy All Services
          deployment: production
          script:
            - kubectl set image deployment/user-service user-service=user-service:${BITBUCKET_BUILD_NUMBER}
            - kubectl set image deployment/order-service order-service=order-service:${BITBUCKET_BUILD_NUMBER}
            - kubectl set image deployment/payment-service payment-service=payment-service:${BITBUCKET_BUILD_NUMBER}
```

### Example 4: Terraform Infrastructure

```yaml
# bitbucket-pipelines.yml
image: hashicorp/terraform:1.5

pipelines:
  branches:
    develop:
      - step:
          name: Terraform Plan (Staging)
          script:
            - cd terraform/
            - terraform init -backend-config=backend-staging.hcl
            - terraform workspace select staging || terraform workspace new staging
            - terraform plan -out=tfplan -var-file=staging.tfvars
          artifacts:
            - terraform/tfplan

      - step:
          name: Terraform Apply (Staging)
          deployment: staging
          trigger: manual
          script:
            - cd terraform/
            - terraform init -backend-config=backend-staging.hcl
            - terraform workspace select staging
            - terraform apply tfplan

    main:
      - step:
          name: Terraform Plan (Production)
          script:
            - cd terraform/
            - terraform init -backend-config=backend-prod.hcl
            - terraform workspace select production || terraform workspace new production
            - terraform plan -out=tfplan -var-file=production.tfvars
          artifacts:
            - terraform/tfplan

      - step:
          name: Terraform Apply (Production)
          deployment: production
          trigger: manual
          script:
            - cd terraform/
            - terraform init -backend-config=backend-prod.hcl
            - terraform workspace select production
            - terraform apply tfplan

  custom:
    destroy-staging:
      - step:
          name: Destroy Staging Environment
          script:
            - cd terraform/
            - terraform init -backend-config=backend-staging.hcl
            - terraform workspace select staging
            - terraform destroy -auto-approve -var-file=staging.tfvars
```

### Example 5: Mobile App (React Native)

```yaml
# bitbucket-pipelines.yml
image: node:16

definitions:
  caches:
    bundler: vendor/bundle
    cocoapods: Pods

pipelines:
  default:
    - step:
        name: Install Dependencies
        caches:
          - node
        script:
          - npm ci

    - parallel:
        - step:
            name: Lint
            script:
              - npm run lint
        - step:
            name: Type Check
            script:
              - npm run type-check
        - step:
            name: Test
            script:
              - npm test

  branches:
    main:
      - step:
          name: Build Android
          image: mingc/android-build-box:latest
          script:
            - npm ci
            - cd android
            - ./gradlew assembleRelease
          artifacts:
            - android/app/build/outputs/**/*.apk

      - step:
          name: Build iOS
          image: macos-11-xcode-12
          script:
            - npm ci
            - cd ios
            - pod install
            - xcodebuild -workspace MyApp.xcworkspace \
                -scheme MyApp \
                -configuration Release \
                -archivePath MyApp.xcarchive \
                archive
          artifacts:
            - ios/MyApp.xcarchive/**

      - step:
          name: Deploy to App Store
          deployment: production
          trigger: manual
          script:
            - fastlane ios release
```

---

## Quick Reference

### Essential Commands

```bash
# Repository
git clone git@bitbucket.org:workspace/repo.git
git remote add origin git@bitbucket.org:workspace/repo.git
git push -u origin main

# Branches
git checkout -b feature/new-feature
git push -u origin feature/new-feature
git branch -d feature/new-feature
git push origin --delete feature/new-feature

# Pull Requests
# Create via Web UI or:
bb pr create --title "Title" --source feature --destination main

# Pipeline
# Trigger: git push
# Manual: Pipelines → Run pipeline
# Custom: Pipelines → Run pipeline → Custom

# API
curl -u username:app-password \
  https://api.bitbucket.org/2.0/repositories/workspace/repo

# Webhooks
# Repository settings → Webhooks → Add webhook
```

### Pipeline Syntax Quick Reference

```yaml
image: node:16

definitions:
  caches:
    custom: path/to/cache
  services:
    postgres:
      image: postgres:14

pipelines:
  default:
    - step:
        name: Step Name
        image: custom:tag
        size: 2x
        caches: [node]
        services: [postgres]
        script:
          - command
        artifacts:
          - files/**

  branches:
    main:
      - step:
          script: echo "main"

  pull-requests:
    '**':
      - step:
          script: echo "PR"

  tags:
    'v*':
      - step:
          script: echo "Tag"

  custom:
    name:
      - step:
          script: echo "Custom"
```

---

**End of Bitbucket Complete Guide**

For more information, visit:
- Official Documentation: https://support.atlassian.com/bitbucket-cloud/
- Bitbucket Pipelines: https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/
- API Documentation: https://developer.atlassian.com/cloud/bitbucket/rest/
- Community: https://community.atlassian.com/t5/Bitbucket/ct-p/bitbucket
