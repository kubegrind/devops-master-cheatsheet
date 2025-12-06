# Git Commands Reference Guide

## Table of Contents

1. [Configuration & Setup](#configuration--setup)
2. [Inspection & Status](#inspection--status)
3. [Branching](#branching)
4. [Staging & Committing](#staging--committing)
5. [Remote Operations](#remote-operations)
6. [Undoing Changes](#undoing-changes)
7. [Rebasing & Merging](#rebasing--merging)
8. [Stashing](#stashing)
9. [Debugging & History](#debugging--history)
10. [Advanced Operations](#advanced-operations)
11. [Common Workflows](#common-workflows)
12. [Troubleshooting](#troubleshooting)

---

## Configuration & Setup

### Configure User Identity

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@company.com"
```

Sets the author name and email for all commits on this system. Use `--local` instead of `--global` to set only for the current repository.

### View Configuration

```bash
git config --global --list
```

Displays all global Git configuration settings. Add `--local` to see repository-specific settings.

### Configure Default Editor

```bash
git config --global core.editor "vim"
```

Sets the default editor for commit messages and interactive rebases. Common options: `vim`, `nano`, `code`, `subl`.

### Configure Line Endings

```bash
git config --global core.autocrlf true
```

On Windows, convert CRLF to LF on commit and LF to CRLF on checkout. On macOS/Linux, use `input` to convert only on commit.

### Configure Aliases

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.unstage 'restore --staged'
```

Creates shortcuts for frequently used commands. Access them with `git st`, `git co`, etc.

---

## Inspection & Status

### Check Current Status

```bash
git status
```

Shows your current branch, staged changes, unstaged changes, and untracked files. Essential for understanding your working directory state before committing.

### View Commit History

```bash
git log
```

Displays full commit history with hash, author, date, and message. Press `q` to exit the pager view.

### Condensed Commit History

```bash
git log --oneline
```

Shows one commit per line with abbreviated hash and message. Use `git log --oneline --graph --all` to visualize branch structure.

### View Specific Number of Commits

```bash
git log -5
```

Shows the last 5 commits. Combine with options: `git log -10 --oneline --author="name"`.

### Show Specific Commit Details

```bash
git show <commit-hash>
```

Displays full details including the diff. Use abbreviated hash or: `git show HEAD` or `git show HEAD~2`.

### View Changes in Working Directory

```bash
git diff
```

Shows unstaged changes in all tracked files. Compares working directory against the staging area.

### View Staged Changes

```bash
git diff --staged
```

Shows differences between staged changes and the last commit. Verify before committing.

### View Changes Between Branches

```bash
git diff main feature-branch
```

Compares two branches showing all differences.

### View Changes in Specific File

```bash
git diff path/to/file.js
```

Shows changes only for that specific file. Works with `--staged` flag as well.

### View File at Specific Commit

```bash
git show <commit>:path/to/file
```

Displays the content of a file as it existed at a specific commit.

### Compact Status

```bash
git status -s
```

Compact short format showing file status with two-letter codes (M, A, D, etc.).

---

## Branching

### List Local Branches

```bash
git branch
```

Shows all local branches with asterisk marking the current branch.

### List All Branches

```bash
git branch -a
```

Includes both local and remote-tracking branches.

### Create New Branch

```bash
git branch feature-new-api
```

Creates a new branch locally without switching to it.

### Create Branch from Specific Commit

```bash
git branch hotfix-bug abc1234
```

Creates a branch starting from a specific commit hash.

### Switch to Branch

```bash
git checkout feature-new-api
```

Switches your working directory to the specified branch.

### Create and Switch to New Branch

```bash
git checkout -b feature-new-api
```

Creates and switches to a new branch in one command.

### Modern Alternative (Git 2.23+)

```bash
git switch feature-new-api
git switch -c feature-new-api
```

The newer `switch` command is more intuitive. `-c` creates and switches.

### Delete Branch Locally

```bash
git branch -d feature-old-api
```

Deletes a branch safely. Use `-D` to force delete.

### Delete Remote Branch

```bash
git push origin --delete feature-old-api
```

Removes a branch from the remote repository.

### Rename Branch

```bash
git branch -m old-name new-name
```

Renames a branch locally.

### Rename and Push

```bash
git branch -m old-name new-name
git push origin -u new-name
git push origin --delete old-name
```

Renames locally, pushes new name with upstream tracking, then deletes old name.

### Set Upstream Branch

```bash
git branch -u origin/main
git branch --set-upstream-to=origin/main
```

Sets the upstream branch for tracking.

### View Branch Details

```bash
git branch -v
```

Shows branches with their last commit hash and message.

### List Merged Branches

```bash
git branch --merged
```

Shows branches that have been merged into the current branch.

### List Unmerged Branches

```bash
git branch --no-merged
```

Shows branches with commits not yet merged into current branch.

---

## Staging & Committing

### Stage Specific File

```bash
git add src/main.js
```

Stages a specific file for commit.

### Stage Multiple Files

```bash
git add src/ config/
```

Stages all changes in specified directories.

### Stage All Changes

```bash
git add .
```

Stages all modified and new files in the repository.

### Stage All Changes Including Deletions

```bash
git add -A
```

Stages modifications, additions, and deletions.

### Interactive Staging

```bash
git add -p
```

Opens interactive mode to review and selectively stage hunks. Options: y, n, s, e, q.

### Commit Staged Changes

```bash
git commit -m "Add user authentication module"
```

Creates a commit with staged changes.

### Commit with Detailed Message

```bash
git commit -m "Add user authentication" -m "Implements JWT token validation and refresh logic"
```

First message is subject, subsequent `-m` flags add body paragraphs.

### Stage and Commit Tracked Files

```bash
git commit -am "Fix login validation logic"
```

Stages all modifications to tracked files and commits in one command.

### Amend Last Commit

```bash
git commit --amend -m "New message"
```

Modifies the last commit's message and/or includes additional staged changes.

### Amend Without Changing Message

```bash
git commit --amend --no-edit
```

Adds staged changes to the last commit without opening an editor.

### Unstage File

```bash
git restore --staged src/main.js
```

Removes a file from staging area without discarding changes.

### Unstage All Files

```bash
git restore --staged .
```

Unstages all files while preserving changes.

### View What Will Be Committed

```bash
git diff --staged
```

Preview exactly what your commit will contain.

### Create Empty Commit

```bash
git commit --allow-empty -m "Deploy v1.0.0"
```

Creates a commit with no file changes. Useful for deployment markers.

---

## Remote Operations

### List Remote Repositories

```bash
git remote -v
```

Shows all configured remotes with their URLs.

### Add Remote Repository

```bash
git remote add origin https://github.com/company/repo.git
```

Adds a new remote.

### Change Remote URL

```bash
git remote set-url origin https://github.com/company/new-repo.git
```

Updates the URL for an existing remote.

### Remove Remote

```bash
git remote remove origin
```

Deletes a remote configuration.

### Fetch Updates from Remote

```bash
git fetch
```

Downloads new commits and branches from all configured remotes.

### Fetch Specific Remote

```bash
git fetch origin
```

Fetches only from the specified remote.

### Fetch Specific Branch

```bash
git fetch origin feature-branch
```

Downloads a specific branch without fetching everything.

### Pull (Fetch + Merge)

```bash
git pull
```

Fetches and merges remote changes into the current branch.

### Pull with Rebase

```bash
git pull --rebase
```

Fetches and rebases your local commits on top of remote changes.

### Push Changes to Remote

```bash
git push
```

Uploads your commits to the remote tracking branch.

### Push to Specific Branch

```bash
git push origin feature-new-api
```

Pushes your commits to the specified remote branch.

### Push and Set Upstream

```bash
git push -u origin feature-new-api
```

Pushes to remote and sets it as the upstream branch for tracking.

### Force Push (Use Cautiously)

```bash
git push --force
```

Overwrites remote history with your local history. Dangerous on shared branches.

### Force Push Safely

```bash
git push --force-with-lease
```

Safer alternative to `--force`. Only pushes if remote hasn't changed.

### Delete Remote Branch

```bash
git push origin --delete feature-old-api
```

Removes a branch from the remote repository.

### Push All Branches

```bash
git push --all
```

Pushes all local branches to their remote counterparts.

### Push All Tags

```bash
git push --tags
```

Pushes all local tags to the remote.

### Fetch and Remove Deleted Remote Branches

```bash
git fetch --prune
```

Removes local remote-tracking branches for branches deleted on the remote.

---

## Undoing Changes

### Discard Unstaged Changes in File

```bash
git restore src/main.js
```

Discards all unstaged changes, reverting to the last committed version.

### Discard All Unstaged Changes

```bash
git restore .
```

Reverts all modified files to their last committed state.

### Unstage File (Preserve Changes)

```bash
git restore --staged src/main.js
```

Removes file from staging area without discarding changes.

### View Undo History

```bash
git reflog
```

Shows all HEAD movements for recovery purposes.

### Recover Lost Commits

```bash
git reflog
git checkout abc1234
```

Use reflog to find the commit hash, then checkout to create a recovery branch.

### Reset to Previous Commit (Soft)

```bash
git reset --soft HEAD~1
```

Moves HEAD back one commit and stages the changes.

### Reset to Previous Commit (Mixed)

```bash
git reset --mixed HEAD~1
```

Moves HEAD back one commit without staging changes. Default behavior.

### Reset to Previous Commit (Hard)

```bash
git reset --hard HEAD~1
```

Moves HEAD back one commit and discards all changes.

### Reset to Remote State

```bash
git reset --hard origin/main
```

Discards all local changes and aligns with remote branch.

### Revert Specific Commit

```bash
git revert abc1234
```

Creates a new commit that undoes the changes from the specified commit.

### Revert Last Commit

```bash
git revert HEAD
```

Creates a new commit that reverses the last commit.

### Clean Untracked Files

```bash
git clean -fd
```

Removes untracked files and directories.

### Preview Cleanup

```bash
git clean -fdn
```

Shows what would be deleted without actually removing anything.

### Remove File from Git Tracking

```bash
git rm --cached src/secrets.js
```

Stops tracking a file without deleting it from disk.

### Create Backup Before Destructive Operation

```bash
git branch backup-branch
git reset --hard origin/main
```

Creates a safety branch before major resets.

---

## Rebasing & Merging

### Merge Branch into Current Branch

```bash
git merge feature-new-api
```

Integrates changes from the specified branch.

### Rebase Current Branch onto Another

```bash
git rebase main
```

Replays your commits on top of the target branch.

### Interactive Rebase

```bash
git rebase -i HEAD~3
```

Opens an editor to reorder, squash, edit, or drop the last 3 commits.

### Squash Commits Before Merging

```bash
git rebase -i HEAD~5
# Mark commits as 'squash' except the first
git push -u origin feature-branch
```

Combines multiple commits into one.

### Abort Failed Rebase

```bash
git rebase --abort
```

Cancels an ongoing rebase and returns to the original state.

### Continue After Resolving Conflicts

```bash
git add .
git rebase --continue
```

After resolving merge conflicts, continue the rebase process.

### Skip Conflicting Commit

```bash
git rebase --skip
```

Ignores the current commit during rebase.

### Merge Without Creating Merge Commit

```bash
git merge --squash feature-branch
```

Combines all commits from the branch into staged changes without a merge commit.

### Merge Specific Commit

```bash
git cherry-pick abc1234
```

Applies a specific commit from another branch to the current branch.

### Cherry-pick Range of Commits

```bash
git cherry-pick abc1234..def5678
```

Applies commits from `abc1234` to `def5678` (exclusive of `abc1234`).

### Abort Cherry-pick

```bash
git cherry-pick --abort
```

Cancels an ongoing cherry-pick operation.

### Merge with Custom Merge Strategy

```bash
git merge -X theirs feature-branch
```

Performs merge using the specified strategy (ours or theirs).

---

## Stashing

### Temporarily Save Changes

```bash
git stash
```

Saves uncommitted changes and reverts the working directory.

### Stash with Description

```bash
git stash push -m "WIP: authentication feature"
```

Saves changes with a descriptive message.

### List All Stashes

```bash
git stash list
```

Shows all saved stashes with their indices.

### View Stash Contents

```bash
git stash show stash@{0}
```

Shows which files are included in the stash.

### View Stash Diff

```bash
git stash show -p stash@{0}
```

Displays the full diff of changes in the stash.

### Apply Stash Without Removing

```bash
git stash apply
```

Restores the most recent stash. Stash remains for future use.

### Apply Specific Stash

```bash
git stash apply stash@{2}
```

Restores a specific stash by index.

### Pop Stash (Apply and Remove)

```bash
git stash pop
```

Restores most recent stash and removes it from the stash list.

### Delete Stash

```bash
git stash drop stash@{0}
```

Permanently removes a specific stash.

### Create Branch from Stash

```bash
git stash branch feature-from-stash
```

Creates a new branch and applies the stash to it.

---

## Debugging & History

### Show Blame for File

```bash
git blame src/main.js
```

Shows which commit modified each line of the file.

### Show Blame with Line Range

```bash
git blame -L 10,20 src/main.js
```

Shows blame only for lines 10 through 20.

### Find Commit That Introduced Bug

```bash
git bisect start
git bisect bad HEAD
git bisect good abc1234
```

Binary search through commits to identify which one introduced a regression.

### Bisect Workflow

```bash
git bisect bad      # Mark current commit as bad
git bisect good     # Mark as good
git bisect reset    # Exit bisect mode
```

Efficiently narrows down the problematic commit.

### View Reference Log

```bash
git reflog
```

Displays all HEAD movements for the last 30 days (default).

### Create Annotated Tag

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
```

Creates a permanent tag with metadata.

### Create Lightweight Tag

```bash
git tag v1.0.0
```

Creates a simple reference to a commit.

### List Tags

```bash
git tag -l
git tag -l "v1.*"
```

Shows all tags. Second example filters by pattern.

### Push Tag to Remote

```bash
git push origin v1.0.0
```

Uploads a specific tag. Use `--tags` for all tags.

### Delete Tag

```bash
git tag -d v1.0.0
git push origin --delete v1.0.0
```

Deletes locally and remotely.

### Show Tag Info

```bash
git show v1.0.0
```

Displays tag details and the associated commit.

### Search Commit Messages

```bash
git log --grep="authentication"
```

Finds commits with matching text in the message.

### Search Code Changes

```bash
git log -S "functionName" -p
```

Finds commits where "functionName" was added or removed.

### Search by Author

```bash
git log --author="John Doe"
```

Shows commits by a specific author.

### View Statistics

```bash
git log --stat
```

Shows commit history with file change statistics.

---

## Advanced Operations

### Cherry-pick Multiple Commits

```bash
git cherry-pick abc1234 def5678 ghi9012
```

Applies multiple specific commits in sequence.

### Rebase onto Different Branch

```bash
git rebase --onto main develop feature-branch
```

Replays commits from `develop` to `feature-branch` onto `main`.

### Rewrite History (Destructive)

```bash
git filter-branch --env-filter '
if [ "$GIT_COMMITTER_EMAIL" = "old@example.com" ]
then
    export GIT_COMMITTER_NAME="New Name"
    export GIT_COMMITTER_EMAIL="new@example.com"
fi' -- --all
```

Changes commit metadata across entire history. Consider `git-filter-repo` instead.

### Garbage Collection

```bash
git gc
```

Optimizes repository by packing objects and cleaning up unused files.

### Verify Repository Integrity

```bash
git fsck
```

Checks repository for corruption or broken objects.

### Create Shallow Clone

```bash
git clone --depth 1 https://github.com/company/repo.git
```

Clones only the latest commit history.

### Deepen Shallow Clone

```bash
git fetch --unshallow
```

Converts a shallow clone to a full clone.

### Change Commit Date

```bash
git commit --amend --date="2 hours ago" --no-edit
```

Modifies the commit date without changing the message.

### Sign Commits with GPG

```bash
git config --global commit.gpgsign true
git config --global user.signingkey <KEY_ID>
```

Enables GPG signing for all commits.

### View Repository Size

```bash
du -sh .git
```

Shows the size of the Git database.

---

## Common Workflows

### Feature Branch Workflow

```bash
git checkout -b feature/user-authentication
# Make changes
git add .
git commit -m "Add user authentication module"
git push -u origin feature/user-authentication
# Create pull request
# After approval:
git checkout main
git pull origin main
git merge feature/user-authentication
git push origin main
```

Standard workflow for developing new features in isolation.

### Hotfix for Production

```bash
git checkout -b hotfix/critical-bug main
# Fix the bug
git add .
git commit -m "Fix critical authentication bug"
git push -u origin hotfix/critical-bug
# Merge to main and develop
git checkout main
git merge --no-ff hotfix/critical-bug
git push origin main
```

Quick fix for production issues.

### Syncing Fork with Upstream

```bash
git remote add upstream https://github.com/original/repo.git
git fetch upstream
git rebase upstream/main
git push origin main --force-with-lease
```

Keeps your fork up-to-date.

### Squashing Commits Before PR

```bash
git rebase -i main
# Mark all but first commit as 'squash'
git push origin feature-branch --force-with-lease
```

Cleans up commit history.

### Recovering Deleted Branch

```bash
git reflog
git checkout -b recovered-branch abc1234
```

Uses reflog to find and restore accidentally deleted branches.

### Undoing Pushed Commits

```bash
git revert abc1234 def5678
git push origin main
```

Creates new commits that undo previous work.

---

## Troubleshooting

### Commit on Wrong Branch

```bash
git log --oneline -1
git checkout correct-branch
git cherry-pick abc1234
git checkout wrong-branch
git reset --hard HEAD~1
```

Moves a commit to the correct branch.

### Undo Last Pushed Commit

```bash
git revert HEAD
git push origin main
```

Creates new commit that undoes the previous one.

### Merge Conflicts

```bash
# Look for conflict markers in files
<<<<<<< HEAD
your changes
=======
their changes
>>>>>>> branch-name
# Edit files to resolve
git add resolved-files
git commit -m "Resolve merge conflicts"
```

Manually edit conflicted files and stage resolution.

### Accidentally Committed Secret

```bash
git reset --soft HEAD~1
git restore --staged secrets.txt
git commit -m "Remove sensitive file"
git push -u origin main --force-with-lease
```

Removes file from latest commit.

### Detached HEAD State

```bash
git branch recovery-branch
git checkout main
```

Recovers from detached HEAD.

### Pull with Conflicting Changes

```bash
git stash
git pull
git stash pop
# Resolve conflicts if any
```

Saves local changes, updates from remote, then reapplies.

### Large Repository Clone Speed

```bash
git clone --depth 1 --single-branch https://repo.git
git fetch --unshallow  # Later if needed
```

Creates shallow clone for faster initial clone.

### Branch Divergence from Remote

```bash
git fetch origin
git status
git pull --rebase
```

Resynchronizes local branch with remote.

---

## Best Practices

Write clear, descriptive commit messages following a consistent format using imperative mood.

Commit frequently with logical, atomic changes rather than large bulk commits.

Use branches for all development work. Never commit directly to main.

Rebase before pushing to keep history clean and linear.

Always pull before pushing to avoid conflicts. Use `git pull --rebase` for cleaner integration.

Delete merged branches to reduce clutter. Regularly prune with `git fetch --prune`.

Sign commits with GPG for production workflows.

Review changes before committing using `git diff` and `git diff --staged`.

Back up important work by pushing to remote frequently.

Document your workflow and conventions as a team.

---

## Resources

- Official Git Documentation: https://git-scm.com/doc
- Interactive Git Learning: https://learngitbranching.js.org/
- Atlassian Git Tutorials: https://www.atlassian.com/git/tutorials

