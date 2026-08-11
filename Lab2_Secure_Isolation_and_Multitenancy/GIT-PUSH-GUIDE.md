# Git Push Guide - PowerShell

## Quick Reference Guide for Pushing Lab 2 to GitHub

---

## Prerequisites

Before you start, ensure you have:
- ✅ Git installed on your system
- ✅ GitHub account created
- ✅ Repository created on GitHub (or ready to create one)

---

## Step-by-Step Guide

### Step 1: Initialize Git Repository (First Time Only)

Open PowerShell in your Lab2 directory and run:

```powershell
# Navigate to your Lab2 directory
cd C:\Users\Surya\Desktop\Lab2

# Initialize git repository
git init
```

**Output:**
```
Initialized empty Git repository in C:/Users/Surya/Desktop/Lab2/.git/
```

---

### Step 2: Configure Git (First Time Only)

Set your Git username and email:

```powershell
# Set your username
git config --global user.name "Your Name"

# Set your email (use your GitHub email)
git config --global user.email "your.email@example.com"

# Verify configuration
git config --list
```

---

### Step 3: Add Files to Staging Area

```powershell
# Add all files in the current directory
git add .

# OR add specific files
git add README.md
git add *.png

# Check status to see what will be committed
git status
```

**Output:**
```
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   README.md
        new file:   Setup Cluster with policy Enforcement 1.png
        ...
```

---

### Step 4: Commit Changes

```powershell
# Commit with a descriptive message
git commit -m "Add Lab 2: Secure Isolation & Multi-Tenancy report and screenshots"

# OR multi-line commit message
git commit -m "Add Lab 2 complete report" -m "- Added comprehensive lab report with all tasks" -m "- Included screenshots for evidence" -m "- Completed all security isolation tasks"
```

**Output:**
```
[main (root-commit) abc1234] Add Lab 2: Secure Isolation & Multi-Tenancy report and screenshots
 16 files changed, 500 insertions(+)
 create mode 100644 README.md
 ...
```

---

### Step 5: Create GitHub Repository

Two options:

#### Option A: Using GitHub Web Interface

1. Go to [https://github.com](https://github.com)
2. Click the **"+"** icon in top right
3. Select **"New repository"**
4. Repository name: `IKB42603-Lab2` (or your preferred name)
5. Description: "Lab 2 - Secure Isolation & Multi-Tenancy"
6. Choose **Public** or **Private**
7. **DO NOT** initialize with README (we already have files)
8. Click **"Create repository"**

#### Option B: Using GitHub CLI (if installed)

```powershell
# Create repository using gh CLI
gh repo create IKB42603-Lab2 --public --source=. --remote=origin
```

---

### Step 6: Add Remote Repository

Copy the repository URL from GitHub and add it as remote:

```powershell
# Add remote repository (HTTPS)
git remote add origin https://github.com/YOUR-USERNAME/IKB42603-Lab2.git

# OR using SSH (if you have SSH keys set up)
git remote add origin git@github.com:YOUR-USERNAME/IKB42603-Lab2.git

# Verify remote was added
git remote -v
```

**Output:**
```
origin  https://github.com/YOUR-USERNAME/IKB42603-Lab2.git (fetch)
origin  https://github.com/YOUR-USERNAME/IKB42603-Lab2.git (push)
```

---

### Step 7: Rename Branch to 'main' (if needed)

GitHub uses 'main' as the default branch name:

```powershell
# Check current branch
git branch

# Rename master to main (if needed)
git branch -M main
```

---

### Step 8: Push to GitHub

```powershell
# Push to GitHub (first time)
git push -u origin main

# For subsequent pushes, simply use:
git push
```

**You may be prompted for authentication:**

1. **Username:** Your GitHub username
2. **Password:** Your GitHub Personal Access Token (NOT your password)

**Output:**
```
Enumerating objects: 18, done.
Counting objects: 100% (18/18), done.
Delta compression using up to 8 threads
Compressing objects: 100% (17/17), done.
Writing objects: 100% (18/18), 2.5 MiB | 1.2 MiB/s, done.
Total 18 (delta 2), reused 0 (delta 0), pack-reused 0
To https://github.com/YOUR-USERNAME/IKB42603-Lab2.git
 * [new branch]      main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

---

## Common Workflows

### Making Changes and Pushing Updates

```powershell
# After making changes to files

# 1. Check what changed
git status

# 2. Add changes
git add .

# 3. Commit with message
git commit -m "Update lab report with additional analysis"

# 4. Push to GitHub
git push
```

---

### Creating a .gitignore File

To exclude certain files from Git:

```powershell
# Create .gitignore file
New-Item -Path .gitignore -ItemType File

# Add content to .gitignore
@"
# Ignore temporary files
*.tmp
*.log
~*

# Ignore system files
.DS_Store
Thumbs.db
desktop.ini

# Ignore sensitive data
*.key
*.pem
secrets/
"@ | Out-File -FilePath .gitignore -Encoding utf8

# Add and commit .gitignore
git add .gitignore
git commit -m "Add .gitignore file"
git push
```

---

## GitHub Personal Access Token (PAT)

If you need to create a Personal Access Token for authentication:

### Steps to Create PAT:

1. Go to GitHub → **Settings**
2. Click **Developer settings** (bottom left)
3. Click **Personal access tokens** → **Tokens (classic)**
4. Click **Generate new token** → **Generate new token (classic)**
5. Note: "Git operations for Lab2"
6. Select scopes:
   - ✅ **repo** (Full control of private repositories)
7. Click **Generate token**
8. **Copy the token immediately** (you won't see it again!)
9. Use this token as your password when pushing

---

## Complete Example Workflow

Here's the complete workflow from start to finish:

```powershell
# 1. Navigate to your directory
cd C:\Users\Surya\Desktop\Lab2

# 2. Initialize git (first time only)
git init

# 3. Configure git (first time only)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 4. Add all files
git add .

# 5. Check status
git status

# 6. Commit
git commit -m "Add Lab 2: Secure Isolation & Multi-Tenancy complete report"

# 7. Add remote repository
git remote add origin https://github.com/YOUR-USERNAME/IKB42603-Lab2.git

# 8. Rename branch to main
git branch -M main

# 9. Push to GitHub
git push -u origin main

# 10. Enter credentials when prompted
# Username: YOUR-USERNAME
# Password: YOUR-PERSONAL-ACCESS-TOKEN
```

---

## Troubleshooting

### Error: "remote origin already exists"

```powershell
# Remove existing remote
git remote remove origin

# Add the correct remote
git remote add origin https://github.com/YOUR-USERNAME/IKB42603-Lab2.git
```

### Error: "failed to push some refs"

```powershell
# Pull latest changes first
git pull origin main --rebase

# Then push
git push origin main
```

### Error: Authentication failed

```powershell
# Make sure you're using Personal Access Token, not password
# Generate new token at: https://github.com/settings/tokens

# Or use SSH instead of HTTPS
git remote set-url origin git@github.com:YOUR-USERNAME/IKB42603-Lab2.git
```

### Error: Large files

```powershell
# Check file sizes
Get-ChildItem -Recurse | Where-Object {$_.Length -gt 50MB} | Select-Object FullName, @{Name="Size(MB)";Expression={[math]::Round($_.Length/1MB,2)}}

# If files are too large, consider using Git LFS
git lfs install
git lfs track "*.png"
git add .gitattributes
git commit -m "Configure Git LFS for images"
```

---

## Useful Git Commands

```powershell
# View commit history
git log

# View short commit history
git log --oneline

# View changes before staging
git diff

# View changes after staging
git diff --staged

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo changes to a file
git checkout -- filename.txt

# View remote repository URL
git remote -v

# Check current branch
git branch

# Create and switch to new branch
git checkout -b feature-branch

# Switch back to main branch
git checkout main

# Merge branch into main
git merge feature-branch

# Delete branch
git branch -d feature-branch
```

---

## Best Practices

1. ✅ **Commit often** with meaningful messages
2. ✅ **Pull before push** to avoid conflicts
3. ✅ **Use .gitignore** to exclude unnecessary files
4. ✅ **Write descriptive commit messages**
5. ✅ **Never commit sensitive data** (passwords, API keys, tokens)
6. ✅ **Keep commits atomic** (one logical change per commit)
7. ✅ **Review changes** before committing (`git status`, `git diff`)

---

## Example Commit Messages

**Good commit messages:**
```
✅ Add Lab 2 complete report with all tasks
✅ Fix Task 4 NetworkPolicy configuration
✅ Update README with verification results
✅ Add screenshots for Session B tasks
```

**Poor commit messages:**
```
❌ update
❌ fix
❌ changes
❌ asdfgh
```

---

## Quick Command Reference

| Command | Description |
|---------|-------------|
| `git init` | Initialize repository |
| `git add .` | Stage all changes |
| `git add <file>` | Stage specific file |
| `git commit -m "message"` | Commit with message |
| `git status` | Check repository status |
| `git log` | View commit history |
| `git remote add origin <url>` | Add remote repository |
| `git push -u origin main` | Push to remote (first time) |
| `git push` | Push to remote |
| `git pull` | Pull from remote |
| `git clone <url>` | Clone repository |
| `git branch` | List branches |
| `git checkout -b <branch>` | Create and switch to branch |

---

## Summary

1. **Initialize** repository: `git init`
2. **Add** files: `git add .`
3. **Commit** changes: `git commit -m "message"`
4. **Add remote**: `git remote add origin <url>`
5. **Push** to GitHub: `git push -u origin main`

---

**Created by:** Kiro AI Assistant  
**For:** IKB42603 Cloud Computing Security Essentials  
**Lab:** Lab 2 - Secure Isolation & Multi-Tenancy  
**Date:** 2026

---

## Additional Resources

- [Git Documentation](https://git-scm.com/doc)
- [GitHub Guides](https://guides.github.com/)
- [GitHub CLI](https://cli.github.com/)
- [Git LFS](https://git-lfs.github.com/)
- [Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials)
