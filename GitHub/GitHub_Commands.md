# GitHub & GitHub Actions CLI Commands Reference Guide

**Version:** 1.0  
**Last Updated:** October 2025  
**Audience:** DevOps Engineers & Development Teams

---

## Table of Contents

1. [GitHub CLI Installation & Setup](#github-cli-installation--setup)
2. [Authentication](#authentication)
3. [Repository Management](#repository-management)
4. [Issue Management](#issue-management)
5. [Pull Request Management](#pull-request-management)
6. [Branch Management](#branch-management)
7. [Release Management](#release-management)
8. [GitHub Actions Basics](#github-actions-basics)
9. [Workflow Files](#workflow-files)
10. [GitHub Actions CLI Commands](#github-actions-cli-commands)
11. [Secrets & Environment Variables](#secrets--environment-variables)
12. [Runner Management](#runner-management)
13. [Common Workflows](#common-workflows)
14. [Advanced Actions Techniques](#advanced-actions-techniques)
15. [Troubleshooting](#troubleshooting)

---

## GitHub CLI Installation & Setup

### Install GitHub CLI (macOS)

```bash
brew install gh
```

Installs GitHub CLI using Homebrew on macOS.

### Install GitHub CLI (Ubuntu/Debian)

```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```

Installs GitHub CLI on Ubuntu/Debian with package verification.

### Install GitHub CLI (Windows)

```bash
choco install gh
# or
scoop install gh
```

Installs GitHub CLI using Chocolatey or Scoop on Windows.

### Check GitHub CLI Version

```bash
gh version
```

Displays installed GitHub CLI version and check for updates.

### Get GitHub CLI Help

```bash
gh help
gh help <command>
gh repo --help
```

Displays help documentation for GitHub CLI and subcommands.

### Update GitHub CLI

```bash
brew upgrade gh
# or platform-specific update method
```

Updates GitHub CLI to the latest version.

---

## Authentication

### Login to GitHub

```bash
gh auth login
```

Interactive authentication process to connect GitHub CLI with your account. Prompts for host, protocol, and credentials.

### Login with HTTPS Token

```bash
gh auth login --with-token < token.txt
```

Authenticates using a personal access token from stdin or file.

### Login to GitHub Enterprise

```bash
gh auth login --hostname github.company.com
```

Authenticates with GitHub Enterprise Server or GitHub Enterprise Cloud.

### Check Authentication Status

```bash
gh auth status
```

Displays current authentication status and authenticated accounts.

### List Authenticated Accounts

```bash
gh auth status --show-token
```

Shows all authenticated accounts and their authentication type (OAuth, PAT, etc.).

### Logout

```bash
gh auth logout
```

Removes authentication credentials for the current host.

### Logout from Specific Host

```bash
gh auth logout --hostname github.company.com
```

Removes authentication for specific GitHub instance.

### Create Personal Access Token

```bash
gh auth refresh --scopes repo,workflow,gist
```

Updates OAuth token scopes for enhanced permissions.

### Export Authentication Token

```bash
gh auth token
```

Displays the current authentication token for use in scripts or CI/CD.

### Use GitHub CLI with SSH

```bash
gh auth login --with-token < token.txt
gh ssh-key add ~/.ssh/id_ed25519.pub --title "DevOps Workstation"
```

Sets up SSH key for secure authentication.

### Configure Git Credential Helper

```bash
gh auth setup-git
```

Configures Git to use GitHub CLI for credential management.

---

## Repository Management

### Create New Repository

```bash
gh repo create my-repo
gh repo create my-repo --public --description "My project"
```

Creates a new repository interactively or with specified options.

### Create Private Repository

```bash
gh repo create my-repo --private
```

Creates a new private repository.

### Create Repository in Organization

```bash
gh repo create org/my-repo --public
```

Creates a repository in specified organization.

### Clone Repository

```bash
gh repo clone owner/repo
gh repo clone owner/repo ./local-path
```

Clones repository using GitHub CLI. Simpler than `git clone`.

### List Repositories

```bash
gh repo list
gh repo list owner
gh repo list owner --limit 100
```

Lists repositories for authenticated user or specified owner.

### List with Filters

```bash
gh repo list --language python --sort stars
gh repo list --visibility public --topic devops
```

Lists repositories with various filters (language, topic, visibility, etc.).

### List Forks

```bash
gh repo list --fork
```

Shows only forked repositories.

### View Repository Information

```bash
gh repo view owner/repo
```

Displays detailed information about a repository.

### View Repository in Browser

```bash
gh repo view --web owner/repo
```

Opens repository on GitHub in default web browser.

### Fork Repository

```bash
gh repo fork owner/repo
gh repo fork owner/repo --clone
```

Creates fork of a repository. `--clone` option automatically clones it.

### Delete Repository

```bash
gh repo delete owner/repo
```

Deletes a repository (requires confirmation).

### Set Repository Description

```bash
gh repo edit owner/repo --description "New description"
```

Updates repository description.

### Set Repository Homepage

```bash
gh repo edit owner/repo --homepage "https://example.com"
```

Sets repository homepage URL.

### Enable/Disable Features

```bash
gh repo edit owner/repo --enable-issues
gh repo edit owner/repo --disable-wiki
```

Enables or disables repository features (issues, wiki, discussions, etc.).

### Set Default Branch

```bash
gh repo edit owner/repo --default-branch main
```

Changes the default branch for the repository.

### Make Repository Template

```bash
gh repo edit owner/repo --template
```

Converts repository into a template for creating new repositories.

### Archive Repository

```bash
gh repo archive owner/repo
```

Archives a repository, making it read-only.

### Unarchive Repository

```bash
gh repo unarchive owner/repo
```

Unarchives a previously archived repository.

### Get Repository Clone URL

```bash
gh repo view owner/repo --json url
```

Returns clone URL as JSON for scripting.

### Sync Fork with Upstream

```bash
gh repo sync owner/repo --branch main
```

Synchronizes fork with upstream repository.

### List Repository Deployments

```bash
gh deployment list -R owner/repo
```

Shows all deployments for a repository.

### Create Repository From Template

```bash
gh repo create my-new-repo --template owner/template-repo --public
```

Creates new repository using template repository.

---

## Issue Management

### Create Issue

```bash
gh issue create
gh issue create --title "Bug: Login not working" --body "Description of the bug"
```

Creates a new issue interactively or with specified options.

### Create Issue with Assignment

```bash
gh issue create --title "Feature request" --assignee @me
gh issue create --title "Bug fix" --assignee username
```

Creates issue and assigns to self or specific user.

### Create Issue with Labels

```bash
gh issue create --title "Bug" --label bug,critical
```

Creates issue and applies labels.

### Create Issue with Milestone

```bash
gh issue create --title "Task" --milestone "v1.0"
```

Creates issue and assigns to milestone.

### Create Issue as Draft

```bash
gh issue create --title "WIP: New feature" --draft
```

Creates issue in draft state for later completion.

### List Issues

```bash
gh issue list
gh issue list --state open
gh issue list --state closed
```

Lists issues with default filters or specific state.

### List Issues with Advanced Filters

```bash
gh issue list --assignee @me --label bug --state open
gh issue list --author username --label critical
gh issue list --search "broken" --state open
```

Lists issues with multiple filter combinations.

### List Issues by Milestone

```bash
gh issue list --milestone "v1.0"
```

Shows issues for specific milestone.

### Search Issues

```bash
gh issue list --search "authentication"
gh issue list --search "label:bug state:open"
```

Searches issues by keywords or structured queries.

### View Issue Details

```bash
gh issue view 42
gh issue view 42 --comments
```

Shows detailed information about specific issue with optional comments.

### View Issue in Browser

```bash
gh issue view 42 --web
```

Opens issue in web browser.

### Add Comment to Issue

```bash
gh issue comment 42 --body "This is resolved"
```

Adds comment to an issue.

### Close Issue

```bash
gh issue close 42
gh issue close 42 --reason "completed"
```

Closes an issue with optional reason.

### Reopen Issue

```bash
gh issue reopen 42
```

Reopens a closed issue.

### Edit Issue

```bash
gh issue edit 42 --title "New title"
gh issue edit 42 --body "Updated description"
```

Edits issue title and/or body.

### Lock Issue

```bash
gh issue lock 42 --reason "off-topic"
```

Locks issue preventing further comments.

### Unlock Issue

```bash
gh issue unlock 42
```

Unlocks a previously locked issue.

### Transfer Issue

```bash
gh issue transfer 42 new-owner/new-repo
```

Moves issue to different repository.

### Delete Issue Comment

```bash
gh issue comment-delete comment-id
```

Removes a comment from an issue.

### Edit Issue Comment

```bash
gh issue comment-edit comment-id --body "Updated comment"
```

Modifies existing issue comment.

### React to Issue

```bash
gh issue reaction add 42 --emoji "+1"
```

Adds emoji reaction to issue.

### Add Assignee

```bash
gh issue edit 42 --add-assignee username
```

Adds assignee to existing issue.

### Remove Assignee

```bash
gh issue edit 42 --remove-assignee username
```

Removes assignee from issue.

### Bulk Update Issues

```bash
gh issue list --state open | while read id; do
  gh issue close $id
done
```

Closes multiple issues using scripting.

---

## Pull Request Management

### Create Pull Request

```bash
gh pr create
gh pr create --title "Feature: New API" --body "Implements new REST API endpoint"
```

Creates new pull request interactively or with specified options.

### Create Pull Request to Specific Base

```bash
gh pr create --base main --head feature-branch
```

Creates PR from feature-branch to main branch.

### Create with Draft Status

```bash
gh pr create --draft --title "WIP: Feature implementation"
```

Creates pull request in draft state for work-in-progress.

### Create with Reviewers

```bash
gh pr create --reviewer @me --reviewer user1,user2
```

Creates PR and requests reviewers.

### Create with Assignees

```bash
gh pr create --assignee @me --assignee user1
```

Creates PR and assigns to users.

### Create with Labels

```bash
gh pr create --label enhancement,documentation
```

Creates PR with labels.

### Create with Template

```bash
gh pr create --template
```

Creates PR using template from `.github/pull_request_template.md`.

### List Pull Requests

```bash
gh pr list
gh pr list --state open
gh pr list --state closed
```

Lists pull requests with state filters.

### List PRs with Filters

```bash
gh pr list --author @me
gh pr list --reviewer @me
gh pr list --search "frontend" --state open
```

Lists PRs by author, reviewer, or search query.

### List PRs Awaiting Review

```bash
gh pr list --review-requested @me
```

Shows PRs requesting your review.

### View PR Details

```bash
gh pr view 42
gh pr view 42 --comments
```

Shows detailed PR information with optional comments.

### View PR in Browser

```bash
gh pr view 42 --web
```

Opens PR on GitHub in browser.

### Checkout PR Branch

```bash
gh pr checkout 42
```

Checks out PR branch locally for testing or modifications.

### Review Pull Request

```bash
gh pr review 42 --approve
gh pr review 42 --request-changes --body "Needs these changes"
gh pr review 42 --comment --body "Great work!"
```

Reviews PR with approval, request changes, or comment.

### Comment on PR

```bash
gh pr comment 42 --body "This looks good"
```

Adds comment to PR.

### Request Review

```bash
gh pr review-requests 42 add user1 user2
```

Requests review from specific users.

### Remove Review Request

```bash
gh pr review-requests 42 remove user1
```

Cancels review request from user.

### Merge Pull Request

```bash
gh pr merge 42
gh pr merge 42 --merge
gh pr merge 42 --squash
gh pr merge 42 --rebase
```

Merges PR. Options specify merge strategy.

### Merge with Auto Delete

```bash
gh pr merge 42 --auto --delete-branch
```

Automatically merges when ready and deletes branch.

### Close Pull Request

```bash
gh pr close 42
```

Closes PR without merging.

### Reopen Pull Request

```bash
gh pr reopen 42
```

Reopens closed PR.

### Edit PR Title

```bash
gh pr edit 42 --title "New title"
```

Updates PR title.

### Edit PR Description

```bash
gh pr edit 42 --body "Updated description"
```

Modifies PR body/description.

### Add Labels to PR

```bash
gh pr edit 42 --add-label bug,enhancement
```

Adds labels to PR.

### Remove Labels from PR

```bash
gh pr edit 42 --remove-label documentation
```

Removes specific labels from PR.

### Check PR Status

```bash
gh pr checks 42
```

Shows status checks for PR (CI/CD results).

### Diff Pull Request

```bash
gh pr diff 42
```

Shows differences introduced by PR.

### Lock PR

```bash
gh pr lock 42 --reason "resolved"
```

Locks PR preventing further discussion.

### Add Assignees

```bash
gh pr edit 42 --add-assignee user1,user2
```

Adds assignees to PR.

### Bulk PR Management

```bash
gh pr list --state open | while read id; do
  gh pr comment $id --body "Auto-comment"
done
```

Performs bulk operations on multiple PRs using scripting.

---

## Branch Management

### List Branches

```bash
gh api repos/{owner}/{repo}/branches
```

Lists all branches using API.

### Create Branch

```bash
git checkout -b feature-new
git push -u origin feature-new
```

Creates branch (using Git, not direct GitHub CLI command).

### Delete Branch

```bash
gh api -X DELETE repos/{owner}/{repo}/git/refs/heads/branch-name
```

Deletes a branch via API.

### Protect Branch

```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  -f required_status_checks='{"strict": true, "contexts": ["ci"]}' \
  -f enforce_admins=true \
  -f required_pull_request_reviews='{"dismiss_stale_reviews": true}'
```

Configures branch protection rules.

### List Protected Branches

```bash
gh api repos/{owner}/{repo}/branches \
  --jq '.[] | select(.protected) | .name'
```

Shows all protected branches.

### Check Branch Protections

```bash
gh api repos/{owner}/{repo}/branches/main/protection
```

Displays protection rules for specific branch.

### Require Status Checks

```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  -f required_status_checks='{"strict": true, "contexts": ["ci", "lint"]}'
```

Requires specific status checks before merge.

### Require Review Approval

```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  -f required_pull_request_reviews='{"required_approving_review_count": 2}'
```

Requires minimum review approvals before merge.

### Dismiss Stale Reviews

```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  -f required_pull_request_reviews='{"dismiss_stale_reviews": true}'
```

Automatically dismisses reviews after new commits.

---

## Release Management

### Create Release

```bash
gh release create v1.0.0
gh release create v1.0.0 --title "Version 1.0.0" --notes "Release notes here"
```

Creates new release/tag with optional title and notes.

### Create Pre-release

```bash
gh release create v1.0.0-rc1 --prerelease --title "Release Candidate 1"
```

Creates pre-release version.

### Create Draft Release

```bash
gh release create v1.0.0 --draft --notes "Release notes"
```

Creates release in draft state for later publication.

### Create Release with Assets

```bash
gh release create v1.0.0 ./dist/app-1.0.0.tar.gz ./dist/app-1.0.0.exe
```

Creates release and uploads build artifacts.

### List Releases

```bash
gh release list
gh release list --limit 10
```

Shows all releases with optional limit.

### View Release Details

```bash
gh release view v1.0.0
```

Displays detailed information about specific release.

### View Release in Browser

```bash
gh release view v1.0.0 --web
```

Opens release on GitHub in browser.

### Edit Release

```bash
gh release edit v1.0.0 --notes "Updated release notes"
gh release edit v1.0.0 --title "New title"
```

Modifies release title or notes.

### Delete Release

```bash
gh release delete v1.0.0
```

Removes a release.

### Upload Assets to Release

```bash
gh release upload v1.0.0 ./dist/app-1.0.0.tar.gz
```

Adds artifact to existing release.

### Download Release Assets

```bash
gh release download v1.0.0
gh release download v1.0.0 -p "*.exe"
```

Downloads release assets, optionally filtered by pattern.

### Mark Release as Latest

```bash
gh release edit v1.0.0 --latest
```

Sets specific release as the latest.

### Publish Draft Release

```bash
gh release edit v1.0.0 --draft=false
```

Publishes previously drafted release.

### Create Release from Tag

```bash
gh release create v1.0.0 --target main
```

Creates release from specific branch or commit.

### Automated Release Creation

```bash
VERSION=$(git describe --tags)
gh release create $VERSION \
  --title "Release $VERSION" \
  --notes "Auto-generated release"
```

Scripts release creation with version detection.

---

## GitHub Actions Basics

### What are GitHub Actions

GitHub Actions is GitHub's built-in CI/CD platform enabling automated workflows triggered by repository events. Workflows execute jobs with steps on runners to automate building, testing, and deploying code.

### Action Components

- **Workflow**: YAML file defining automation logic
- **Event**: Trigger (push, pull_request, schedule, etc.)
- **Job**: Unit of work with multiple steps
- **Step**: Individual command or action
- **Action**: Reusable unit of code
- **Runner**: Server executing jobs

### Create Workflow File

```bash
mkdir -p .github/workflows
touch .github/workflows/ci.yml
```

Sets up directory structure for GitHub Actions workflows.

### View Workflow Runs

```bash
gh run list
gh run list --status failure
gh run list --workflow ci.yml
```

Lists workflow runs with optional filters for status or specific workflow.

### View Specific Run

```bash
gh run view 1234567
gh run view 1234567 --log
```

Shows details of specific run with optional log output.

### Download Logs

```bash
gh run download 1234567
```

Downloads all logs from specific run as archive.

### View Run in Browser

```bash
gh run view 1234567 --web
```

Opens run details on GitHub in browser.

### Cancel Run

```bash
gh run cancel 1234567
```

Stops running workflow execution.

### Retry Run

```bash
gh run rerun 1234567
```

Re-executes failed workflow run.

### Delete Run

```bash
gh run delete 1234567
```

Removes workflow run and associated logs.

### Wait for Run Completion

```bash
gh run watch 1234567
```

Monitors run status, updating until completion.

### List Failed Runs

```bash
gh run list --status failure
```

Shows all failed workflow runs.

### View Job Details

```bash
gh run view 1234567 --exit-status
```

Shows exit status of run jobs.

---

## Workflow Files

### Basic Workflow Structure

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: npm test
```

Minimal workflow with trigger event and single job.

### Trigger on Push

```yaml
on:
  push:
    branches:
      - main
      - develop
    paths:
      - 'src/**'
      - 'package.json'
    tags:
      - 'v*'
```

Workflow triggered on push to specific branches with path filters.

### Trigger on Pull Request

```yaml
on:
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]
    paths:
      - 'src/**'
```

Workflow triggered on PR events with specific activity types.

### Scheduled Workflow

```yaml
on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight UTC
    - cron: '0 2 * * 0'  # Weekly on Sunday at 2 AM UTC
```

Cron-triggered workflow for scheduled tasks.

### Workflow Dispatch (Manual Trigger)

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
```

Allows manual workflow trigger with input parameters.

### Multiple Event Triggers

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  release:
    types: [published]
  workflow_dispatch:
```

Workflow triggered by multiple event types.

### Matrix Strategy

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.8', '3.9', '3.10', '3.11']
        os: [ubuntu-latest, windows-latest, macos-latest]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - run: pytest
```

Runs job across multiple configurations simultaneously.

### Conditional Step Execution

```yaml
steps:
  - name: Step A
    run: echo "Always runs"
  
  - name: Step B
    if: github.event_name == 'push'
    run: echo "Only on push"
  
  - name: Step C
    if: failure()
    run: echo "Only if previous step failed"
  
  - name: Step D
    if: success()
    run: echo "Only if all previous steps succeeded"
```

Conditional execution based on event type or job status.

### Environment Variables

```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    env:
      NODE_ENV: production
    steps:
      - run: echo ${{ env.REGISTRY }}
      - run: echo $NODE_ENV
```

Setting environment variables at workflow and job level.

### Outputs and Artifacts

```yaml
jobs:
  build:
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    steps:
      - uses: actions/checkout@v4
      - name: Set output
        id: meta
        run: echo "tags=myimage:latest" >> $GITHUB_OUTPUT
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: build-output
          path: dist/

  deploy:
    needs: build
    steps:
      - run: echo "Using ${{ needs.build.outputs.image-tag }}"
      - uses: actions/download-artifact@v3
```

Job outputs and artifact passing between jobs.

### Secrets in Workflow

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: |
          echo "Deploying with credentials"
          # Use secrets in deployment script
```

Accessing repository secrets in workflow steps.

### Workflow Permissions

```yaml
permissions:
  contents: read
  pull-requests: write
  deployments: write
  checks: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
```

Setting minimum permissions for workflow operations.

### Timeout Configuration

```yaml
jobs:
  long-running:
    runs-on: ubuntu-latest
    timeout-minutes: 120
    steps:
      - uses: actions/checkout@v4
      - name: Run long task
        run: ./long-running-script.sh
```

Sets job execution timeout limit.

### Continue on Error

```yaml
steps:
  - name: Run tests
    continue-on-error: true
    run: npm test
  
  - name: Generate report
    if: always()
    run: npm run report
```

Steps continue even if previous step fails.

### Concurrency Control

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
```

Limits concurrent workflow runs and cancels queued runs.

### Required Status Checks

```yaml
# In branch protection settings
# Require successful completion of:
# - build (workflow name)
# - test (workflow name)
```

Enforces workflow completion before PR merge.

---

## GitHub Actions CLI Commands

### List Workflows

```bash
gh workflow list
gh workflow list --all
```

Shows available workflows in repository.

### View Workflow

```bash
gh workflow view ci.yml
```

Displays workflow file content and details.

### Enable Workflow

```bash
gh workflow enable ci.yml
gh workflow enable 1234567
```

Re-enables disabled workflow.

### Disable Workflow

```bash
gh workflow disable ci.yml
```

Disables workflow from running.

### Trigger Workflow Manually

```bash
gh workflow run ci.yml
gh workflow run ci.yml --ref main --field environment=production
```

Manually triggers workflow with optional inputs.

### Run Workflow with Inputs

```bash
gh workflow run deploy.yml \
  --ref main \
  --field environment=production \
  --field version=1.0.0
```

Triggers workflow with specific parameter values.

### View Workflow Runs

```bash
gh run list --workflow ci.yml
gh run list --status completed
```

Lists runs for specific workflow with status filters.

### Get Latest Run

```bash
gh run list --limit 1 --workflow ci.yml
```

Shows most recent workflow run.

### View Run Log

```bash
gh run view 1234567 --log
```

Displays complete log output from run.

### Stream Run Logs

```bash
gh run watch 1234567
```

Watches run in real-time as it executes.

### Download Run Artifacts

```bash
gh run download 1234567
gh run download 1234567 -n build-artifact
```

Downloads artifacts from completed run.

### View Step Logs

```bash
gh run view 1234567 --job 5678
```

Shows logs for specific job within run.

### Check Last Commit Status

```bash
gh run list --limit 1 --branch main --status completed
```

Finds latest run status for specific branch.

### Monitor Specific Branch

```bash
while true; do
  gh run list --branch main --limit 1 --status in_progress
  sleep 30
done
```

Continuously monitors workflow status on branch.

### Get Run Conclusion

```bash
gh run view 1234567 --json conclusion
```

Returns run result (success, failure, cancelled, etc.).

### Create Deployment from Workflow

```bash
gh run view 1234567 --json deployments
```

Views deployments created by workflow run.

---

## Secrets & Environment Variables

### List Repository Secrets

```bash
gh secret list
```

Shows all secrets configured for repository.

### Create/Update Secret

```bash
gh secret set DB_PASSWORD
# Enter secret value when prompted

echo "my-secret-value" | gh secret set MY_SECRET
```

Creates or updates repository secret.

### Create Secret from File

```bash
gh secret set DEPLOY_KEY < ~/.ssh/deploy_key
```

Creates secret from file contents.

### Delete Secret

```bash
gh secret delete DB_PASSWORD
```

Removes repository secret.

### List Organization Secrets

```bash
gh secret list --org myorg
```

Shows secrets available to organization.

### Create Organization Secret

```bash
gh secret set SHARED_SECRET --org myorg
```

Creates secret at organization level.

### Create Environment Secret

```bash
gh secret set DB_PASSWORD --env production
```

Creates secret specific to deployment environment.

### List Environment Secrets

```bash
gh secret list --env production
```

Shows secrets for specific environment.

### Create Organization Variable

```bash
gh variable set NODE_ENV --org myorg --body "production"
```

Creates organization-wide configuration variable.

### List Organization Variables

```bash
gh variable list --org myorg
```

Shows organization-level variables.

### Create Repository Variable

```bash
gh variable set DEPLOY_REGION --body "us-east-1"
```

Creates repository-level variable for workflows.

### List Repository Variables

```bash
gh variable list
```

Shows all repository variables.

### Delete Variable

```bash
gh variable delete DEPLOY_REGION
```

Removes repository or organization variable.

### Use Secrets in Workflow

```yaml
jobs:
  deploy:
    env:
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
      NODE_ENV: ${{ vars.NODE_ENV }}
    steps:
      - run: ./deploy.sh
```

References secrets and variables in workflow steps.

### Secret Management Best Practices

```bash
# Never log secrets
gh secret set SECRET_VALUE  # Prompts interactively

# Use specific secret names
gh secret set GITHUB_TOKEN  # Standard GitHub token

# Rotate secrets regularly
gh secret delete OLD_SECRET
gh secret set NEW_SECRET

# Use fine-grained credentials
# Create PAT with minimal required scopes
```

Best practices for managing secrets securely.

---

## Runner Management

### List Organization Runners

```bash
gh run-list --org myorg
gh api orgs/myorg/actions/runners
```

Shows all self-hosted runners in organization.

### List Repository Runners

```bash
gh api repos/owner/repo/actions/runners
```

Shows runners available to specific repository.

### Add Organization Runner Group

```bash
gh api orgs/myorg/actions/runner-groups \
  -f name=production \
  -f visibility=private
```

Creates group for organizing runners.

### Add Repository to Runner Group

```bash
gh api repos/owner/repo/actions/secrets/repository-secrets \
  -f name=RUNNER_GROUP \
  -f value=production
```

Assigns repository to specific runner group.

### View Runner Details

```bash
gh api orgs/myorg/actions/runners/1234567
```

Shows detailed runner information and capabilities.

### Remove Runner

```bash
gh api -X DELETE orgs/myorg/actions/runners/1234567
```

Unregisters self-hosted runner.

### Enable/Disable Runner

```bash
gh api orgs/myorg/actions/runners/1234567 \
  -f status=offline
```

Changes runner availability status.

### View Runner Jobs

```bash
gh api orgs/myorg/actions/runners/1234567/jobs
```

Shows jobs executed on specific runner.

### Generate Runner Token

```bash
gh api orgs/myorg/actions/runners/registration-token
```

Creates token for registering new runner.

---

## Common Workflows

### Continuous Integration Pipeline

```yaml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      - run: npm install
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      - run: npm install
      - run: npm test
      - uses: codecov/codecov-action@v3
        if: always()

  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      - run: npm install
      - run: npm run build
      - uses: actions/upload-artifact@v3
        with:
          name: build
          path: dist/
```

Complete CI pipeline with linting, testing, and building.

### Docker Build and Push

```yaml
name: Build and Push Docker Image

on:
  push:
    tags: ['v*']

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v2
      - uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.ref_name }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
```

Builds and publishes Docker image to container registry.

### Deploy to Production

```yaml
name: Deploy to Production

on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.ref_name }}
      
      - name: Configure credentials
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.DEPLOY_KEY }}" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H ${{ secrets.DEPLOY_HOST }} >> ~/.ssh/known_hosts
      
      - name: Deploy application
        run: |
          ssh -i ~/.ssh/deploy_key user@${{ secrets.DEPLOY_HOST }} \
            'cd /app && git pull && npm install && npm run build && pm2 restart app'
      
      - name: Create deployment
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.repos.createDeployment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              ref: context.ref,
              environment: 'production',
              auto_merge: false,
              production_environment: true,
              required_contexts: []
            });
```

Production deployment with environment protection.

### Release Creation

```yaml
name: Create Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      
      - name: Build artifacts
        run: npm run build
      
      - name: Create release
        uses: softprops/action-gh-release@v1
        with:
          files: dist/**
          body_path: CHANGELOG.md
          draft: false
          prerelease: ${{ contains(github.ref, 'rc') || contains(github.ref, 'alpha') }}
```

Automatically creates release with build artifacts.

### Schedule Cleanup Job

```yaml
name: Scheduled Maintenance

on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Clean up old artifacts
        run: |
          # Run cleanup scripts
          ./scripts/cleanup-artifacts.sh
      
      - name: Generate report
        run: |
          # Generate and send reports
          ./scripts/generate-report.sh
```

Scheduled maintenance tasks and reporting.

### Multi-OS Testing

```yaml
name: Multi-OS Testing

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python-version: ['3.8', '3.9', '3.10', '3.11']
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Install dependencies
        run: pip install -r requirements.txt
      
      - name: Run tests
        run: pytest
```

Tests application across multiple OS and Python versions.

### Comment PR with Results

```yaml
name: Test Results Comment

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test -- --json --outputFile=test-results.json
      
      - name: Comment PR with results
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const results = JSON.parse(fs.readFileSync('test-results.json', 'utf8'));
            
            const comment = `## Test Results
            - Tests: ${results.numPassedTests}/${results.numTotalTests} passed
            - Coverage: ${results.testResults[0].coverage}%`;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

Posts test results as PR comment.

### Notify Slack on Failure

```yaml
name: Notify Slack

on:
  workflow_run:
    workflows: ['CI Pipeline']
    types: [completed]

jobs:
  notify:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'failure' }}
    steps:
      - uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "Workflow failed in ${{ github.repository }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Workflow Failure*\nRepository: ${{ github.repository }}\nBranch: ${{ github.ref_name }}"
                  }
                }
              ]
            }
```

Sends Slack notification on workflow failure.

---

## Advanced Actions Techniques

### Custom Action with Inputs/Outputs

```yaml
name: Custom Deploy Action
description: Deploys application to server

inputs:
  host:
    description: Deployment host
    required: true
  key:
    description: SSH private key
    required: true
  environment:
    description: Deployment environment
    required: true
    default: staging

outputs:
  deployment-url:
    description: URL of deployed application
    value: ${{ steps.deploy.outputs.url }}

runs:
  using: composite
  steps:
    - id: deploy
      run: |
        echo "Deploying to ${{ inputs.host }}"
        ./scripts/deploy.sh "${{ inputs.host }}" "${{ inputs.environment }}"
        echo "url=https://myapp.example.com" >> $GITHUB_OUTPUT
      shell: bash
```

Creates reusable custom action with inputs and outputs.

### Docker Action

```dockerfile
FROM node:18-alpine

RUN apk add --no-cache git openssh-client

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```

Dockerfile-based custom action.

### JavaScript Action

```javascript
// action.js
const core = require('@actions/core');
const github = require('@actions/github');

async function run() {
  try {
    const inputValue = core.getInput('my-input');
    console.log(`Processing: ${inputValue}`);
    
    core.setOutput('result', 'success');
  } catch (error) {
    core.setFailed(error.message);
  }
}

run();
```

Node.js/JavaScript based custom action.

### Composite Action Example

```yaml
name: Setup and Build
description: Sets up environment and builds project

inputs:
  node-version:
    description: Node.js version
    default: 18
  build-args:
    description: Build arguments
    default: ''

outputs:
  build-path:
    description: Path to build output
    value: ${{ steps.build.outputs.path }}

runs:
  using: composite
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
    
    - run: npm install
      shell: bash
    
    - id: build
      run: |
        npm run build ${{ inputs.build-args }}
        echo "path=./dist" >> $GITHUB_OUTPUT
      shell: bash
```

Composite action combining multiple steps.

### Reusable Workflow

```yaml
# .github/workflows/reusable-deploy.yml
name: Reusable Deploy Workflow

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      version:
        required: true
        type: string
    secrets:
      deploy-key:
        required: true
      deploy-host:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        env:
          DEPLOY_KEY: ${{ secrets.deploy-key }}
          DEPLOY_HOST: ${{ secrets.deploy-host }}
        run: ./scripts/deploy.sh ${{ inputs.version }}
```

Reusable workflow called from other workflows.

### Call Reusable Workflow

```yaml
name: Deploy Application

on:
  release:
    types: [published]

jobs:
  deploy-staging:
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: staging
      version: ${{ github.ref_name }}
    secrets:
      deploy-key: ${{ secrets.STAGING_DEPLOY_KEY }}
      deploy-host: ${{ secrets.STAGING_DEPLOY_HOST }}
  
  deploy-production:
    needs: deploy-staging
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: production
      version: ${{ github.ref_name }}
    secrets:
      deploy-key: ${{ secrets.PROD_DEPLOY_KEY }}
      deploy-host: ${{ secrets.PROD_DEPLOY_HOST }}
```

Uses reusable workflow with different parameters.

### Context and Expression Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Print contexts
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Actor: ${{ github.actor }}"
          echo "Event: ${{ github.event_name }}"
          echo "Commit: ${{ github.sha }}"
          echo "Ref: ${{ github.ref }}"
      
      - name: Conditional logic
        run: |
          if [[ "${{ github.ref_name }}" == "main" ]]; then
            echo "Running on main branch"
          fi
```

Using GitHub context variables in workflows.

### Dynamic Job Matrix

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - id: set-matrix
        run: |
          echo "matrix={\"include\":[{\"os\":\"ubuntu-latest\",\"node\":\"18\"},{\"os\":\"windows-latest\",\"node\":\"18\"}]}" >> $GITHUB_OUTPUT
  
  test:
    needs: setup
    runs-on: ${{ matrix.os }}
    strategy:
      matrix: ${{ fromJson(needs.setup.outputs.matrix) }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
```

Dynamically generates job matrix based on previous step output.

### Cache Dependencies

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Cache node modules
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
      
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      
      - run: npm install
      - run: npm run build
```

Caches dependencies between workflow runs.

### Artifact Management

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-${{ github.run_number }}
          path: dist/
          retention-days: 30
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v3
        with:
          name: build-${{ github.run_number }}
      
      - name: Deploy
        run: ./scripts/deploy.sh
```

Passes build artifacts between jobs.

---

## Troubleshooting

### Workflow Not Triggering

```bash
# Check workflow file syntax
gh workflow view ci.yml

# Verify event trigger
# Check branch/path filters match
# Ensure workflow is enabled
gh workflow enable ci.yml

# View recent runs
gh run list --workflow ci.yml
```

Diagnosing workflow trigger issues.

### Job Fails with Permission Error

```yaml
jobs:
  deploy:
    permissions:
      contents: write
      packages: write
      deployments: write
    steps:
      - uses: actions/checkout@v4
      - run: ./deploy.sh
```

Fixing insufficient permissions for job operations.

### Secret Not Available in Workflow

```bash
# Verify secret exists
gh secret list

# Check secret name matches exactly (case-sensitive)
# Verify repository secret vs organization secret

# Recreate secret if needed
gh secret delete SECRET_NAME
gh secret set SECRET_NAME
```

Troubleshooting secret access issues.

### Runner Connection Issues

```bash
# Check runner status
gh api repos/owner/repo/actions/runners

# View runner logs
# Verify network connectivity from runner
# Check firewall rules
# Ensure runner process is running
```

Debugging self-hosted runner problems.

### Long Workflow Execution

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 60
    steps:
      - uses: actions/checkout@v4
      - name: Long-running task
        timeout-minutes: 30
        run: ./long-task.sh
```

Addressing workflow timeout issues.

### Out of Disk Space During Build

```yaml
steps:
  - uses: actions/checkout@v4
  - name: Free disk space
    run: |
      sudo apt-get clean
      docker system prune -a -f
  - run: npm run build
```

Managing disk space in workflow runners.

### Debugging Failed Workflow

```bash
# Get run details
gh run view <run-id> --log

# Download full logs
gh run download <run-id>

# Check specific job
gh run view <run-id> --json jobs

# Re-run with verbose logging
ACTIONS_RUNNER_DEBUG=true gh run rerun <run-id>
```

Diagnosing workflow failures.

### YAML Syntax Errors

```bash
# Validate workflow syntax
gh workflow view ci.yml

# Check indentation
# Verify quotes and special characters
# Use YAML validator: yamllint ci.yml
```

Fixing YAML configuration issues.

### Environment Variables Not Set

```yaml
env:
  GLOBAL_VAR: value

jobs:
  build:
    env:
      JOB_VAR: value
    steps:
      - name: Step with env
        env:
          STEP_VAR: value
        run: |
          echo "Global: $GLOBAL_VAR"
          echo "Job: $JOB_VAR"
          echo "Step: $STEP_VAR"
```

Scope and precedence of environment variables.

### Matrix Job Not Creating Variants

```yaml
strategy:
  matrix:
    include:
      - os: ubuntu-latest
        python: '3.8'
      - os: windows-latest
        python: '3.9'
      - os: macos-latest
        python: '3.10'
```

Fixing matrix configuration issues.

### Action Version Compatibility

```yaml
# Use pinned versions
- uses: actions/setup-node@v4.0.1

# Or major version
- uses: actions/setup-node@v4

# Avoid 'latest' tag
- uses: actions/setup-node@latest  # ❌ Not recommended
```

Managing action versions in workflows.

---

## Best Practices

Use specific action versions instead of `@latest` for reproducibility. Pin to major versions like `@v4` for regular updates.

Set appropriate concurrency limits to prevent duplicate workflow runs. Use `concurrency` with `cancel-in-progress: true`.

Store sensitive information in secrets, never in workflows or code. Use environment-specific secrets for different environments.

Organize workflows logically with clear naming and documentation. Add comments explaining complex logic.

Use reusable workflows to reduce duplication across multiple workflows.

Cache dependencies to speed up workflow execution. Cache package managers, build artifacts, and external dependencies.

Implement proper error handling with `if: always()`, `if: failure()`, and `continue-on-error`.

Set appropriate timeouts to prevent stuck workflows. Use both workflow and job-level timeouts.

Use matrix strategies to test across multiple configurations simultaneously.

Monitor workflow performance and optimize long-running steps. Consider parallelization.

Implement branch protection requiring successful status checks before merge.

Use workflow permissions minimally. Grant only required permissions per job.

Version control all workflow files. Include them in code reviews.

Test workflows thoroughly before deploying to production.

Document workflow purpose, triggers, and manual inputs in README or workflow descriptions.

---

## Resources

- GitHub CLI Documentation: https://cli.github.com/
- GitHub Actions Documentation: https://docs.github.com/en/actions
- GitHub Actions Examples: https://github.com/actions
- GitHub Script Action: https://github.com/actions/github-script
- Act - Run workflows locally: https://github.com/nektos/act
- GitHub Actions Best Practices: https://github.com/marketplace?type=actions
- Workflow Syntax Reference: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions

