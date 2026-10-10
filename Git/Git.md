# Git and GitHub Notes

## 1. Software Installation

Install and configure the following tools before starting:

- **Git for Windows:** https://git-scm.com/download/win
- **GitHub account:** https://github.com/
- **Visual Studio Code:** https://code.visualstudio.com/

### Verify Git installation

Open Command Prompt or the VS Code terminal and run:

```bash
git --version
```

Example output:

```text
git version 2.56.0.windows.1
```

The version number may be different on your computer.

## 2. Open a Project in Visual Studio Code

Move into your project directory:

```bat
cd cloudtrain-git-github-04-10-2026
```

List the files and folders:

```bat
dir
```

Open the current folder in VS Code:

```bat
code .
```

If `code .` is not recognized, open the folder from VS Code using **File → Open Folder**.

## 3. Check Repository Status

```bash
git status
```

This shows the current branch and whether files are modified, staged, or committed.

Example:

```text
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  modified: README.md
```

## 4. Configure Your Git Identity

Set the name and email that Git should associate with your commits:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

Replace the example values with your own name and email. To check the settings:

```bash
git config --global --list
```

## 5. Stage, Commit, and Push Changes

### Step 1: Stage a file

```bash
git add README.md
```

To stage all changes in the current directory:

```bash
git add .
```

### Step 2: Verify staged changes

```bash
git status
```

Staged files appear under **Changes to be committed**.

### Step 3: Commit the changes

```bash
git commit -m "Updated readme file"
```

A commit saves a snapshot of the staged changes in your local repository.

### Step 4: Push commits to GitHub

```bash
git push
```

If the branch has no upstream configured yet, use:

```bash
git push -u origin feature-1
```

### Step 5: Verify the result

```bash
git status
```

A clean repository may show:

```text
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean
```

**Remember:** `git add` stages changes, `git commit` saves them locally, and `git push` uploads commits to the remote repository.

## 6. Create and Switch Branches

A branch lets you work on a change without directly modifying another branch.

### Create a branch

```bash
git branch feature-1
```

### List local branches

```bash
git branch
```

The `*` marks the current branch.

### Switch to the new branch

```bash
git checkout feature-1
```

Or create and switch to a branch in one command:

```bash
git checkout -b feature-1
```

The newer equivalent for switching branches is:

```bash
git switch feature-1
```

To return to `main`:

```bash
git switch main
```

### Publish a new branch to GitHub

```bash
git push -u origin feature-1
```

## 7. Common Git Commands

| Command | Purpose |
|---|---|
| `git --version` | Check the installed Git version |
| `git status` | Check repository and working-tree status |
| `git branch` | List local branches |
| `git branch feature-1` | Create a branch |
| `git switch feature-1` | Switch branches |
| `git add README.md` | Stage a specific file |
| `git add .` | Stage changes in the current directory |
| `git commit -m "message"` | Commit staged changes |
| `git push` | Push commits to the remote |
| `git pull` | Fetch and integrate remote changes |
| `git log --oneline` | View a compact commit history |
| `git diff` | View unstaged changes |
| `git remote -v` | Show configured remote URLs |

## 8. GitHub Actions

GitHub Actions automates tasks such as testing, building, and deploying applications. A workflow is a YAML file stored in the repository under:

```text
.github/workflows/
```

For example, create `.github/workflows/ci.yml` with the following content:

```yaml
# A basic GitHub Actions workflow
name: CI

# Run on pushes and pull requests targeting main,
# or when manually started from the Actions tab.
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
  workflow_dispatch:

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest

    steps:
      # Download repository contents to the runner.
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Run a one-line script
        run: echo "Hello, world!"

      - name: Run multiple commands
        run: |
          echo "Add build steps here."
          echo "Add test steps here."

  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Deployment placeholder
        run: echo "Add deployment steps here."
```

### Understand the workflow

- **`name: CI`** — names the workflow.
- **`on`** — defines the events that trigger it.
- **`push`** — runs when changes are pushed to `main`.
- **`pull_request`** — runs for pull requests targeting `main`.
- **`workflow_dispatch`** — allows manual execution from GitHub Actions.
- **`jobs`** — defines jobs in the workflow.
- **`runs-on: ubuntu-latest`** — uses a GitHub-hosted Ubuntu runner.
- **`steps`** — lists the actions and commands in a job.
- **`actions/checkout@v4`** — checks out the repository so the job can access its files.
- **`needs: build`** — makes the `deploy` job wait for the `build` job to succeed.

**Important:** This example's deploy job only prints a message; it does not deploy an application. Add your actual deployment commands and configure required credentials as GitHub Actions secrets. Never commit passwords, access tokens, private keys, or cloud credentials to the repository.

### Add a workflow to your repository

1. Create the `.github/workflows` directory if it does not exist.
2. Save the YAML as `.github/workflows/ci.yml`.
3. Stage and commit the file:

   ```bash
   git add .github/workflows/ci.yml
   git commit -m "Add GitHub Actions CI workflow"
   git push
   ```

4. Open your repository on GitHub and select **Actions** to view workflow runs.

## 9. Typical Git Workflow

```text
Edit files
   ↓
git status
   ↓
git add .
   ↓
git commit -m "Describe your change"
   ↓
git push
   ↓
Verify on GitHub
```

For feature development, create a branch, commit and push your changes, then open a pull request to merge them into `main`.

## 10. Troubleshooting Tips

- **Git command not recognized:** Install Git for Windows and reopen the terminal.
- **`code .` not recognized:** Open the project through VS Code's **File → Open Folder** menu, or enable the `code` command in your installation.
- **Nothing to commit:** Run `git status` and check whether you have changed or staged any files.
- **Push rejected:** Run `git pull` and resolve any conflicts before pushing again. Follow your team's branch and merge policy.
- **Authentication failed:** Sign in using an approved GitHub authentication method; do not put credentials directly in commands or files.
- **Workflow not running:** Check that the YAML file is under `.github/workflows/`, the trigger branch matches, and the workflow has no YAML syntax errors.
