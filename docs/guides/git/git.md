---
tags:
  - git
  - service
  - config
  - cheat_sheet
---
# Git Reference & Version Control

## 1. Command Cheat Sheet

_Quick reference_

|**Action**|**Command**|
|---|---|
|**Check State**|`git status` (Always run this first)|
|**View Changes**|`git diff`|
|**Stage Files**|`git add .` (Stage all) or `git add <filename>`|
|**Commit**|`git commit -m "Your message here"`|
|**Push**|`git push origin main`|
|**Pull Updates**|`git pull`|
|**Undo File Edit**|`git restore <file>` (Discards local changes)|
|**Unstage File**|`git restore --staged <file>` (Keeps changes, removes from commit)|
|**View History**|`git log --oneline --graph --all`|

### 1.1. Emergency Fixes

_Commands to fix mistakes immediately._

|**Scenario**|**Workflow / Command**|
|---|---|
|**Typos in last commit**|`git add .` $\to$ `git commit --amend -m "New Message"` $\to$ `git push -f`|
|**Leaked Secret**|`git rm --cached .env` $\to$ `echo ".env" >> .gitignore` $\to$ `git commit -m "rm secret"`|
|**Reset to Remote**|`git fetch origin` $\to$ `git reset --hard origin/main` (Destructive: makes local match cloud)|

---

## 2. Initialization (One-Time Setup)

_Run this only when setting up the M920q for the first time._

### 2.1. Configure Identity

Bash

```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
# Set default branch to main to match GitHub modern standards
git config --global init.defaultBranch main 
```

### 2.2. Initialize Repo

Bash

```
cd ~/homelab/docker/
git init
```

### 2.3. The Ignore Rules (Critical)

_Prevent secrets and system junk from uploading._

Create `.gitignore` in the root (`~/homelab/docker/.gitignore`):

Bash

```
# --- Secrets (NEVER COMMIT) ---
**/.env
**/.env.*
**/secrets/
id_rsa
*.pem

# --- System Junk ---
.DS_Store
Thumbs.db
**/.git/   # Ignore nested git repos (prevent submodule errors)

# --- Docker Data (If mapped locally) ---
**/data/
**/mysql/
**/influxdb/ 

# --- Custom Ignores ---
# Files named .pastebin anywhere
**/.pastebin
# Folders named .pastebin anywhere
**/.pastebin/
```

### 2.4. Connect to GitHub

Bash

```
# 1. Link Remote
git remote add origin https://github.com/YourUser/homelab-backup.git

# 2. First Push
git add .
git commit -m "Initial homelab commit"
git push -u origin main
```

---

## 3. Daily Workflows (CLI)

### 3.1. The "Config Change" Routine

_Run this every time you edit a `compose.yaml` or config file._

1. **Check Status:**
    
    Bash
    
    ```
    git status
    ```
    
    - _Verify no `.env` files are red (untracked) or green (staged)._
        
2. **Stage & Commit:**
    
    Bash
    
    ```
    git add .
    git commit -m "Added Tautulli to media stack"
    ```
    
3. **Push:**
    
    Bash
    
    ```
    git push
    ```
    

### 3.2. Disaster Recovery (Rebuild)

_Use this if the M920q OS drive fails and you reinstall Ubuntu._

1. **Install Git:**
    
    Bash
    
    ```
    sudo apt update && sudo apt install git -y
    ```
    
2. **Clone Configs:**
    
    Bash
    
    ```
    git clone https://github.com/YourUser/homelab-backup.git ~/homelab/docker
    ```
    
3. **Restore Secrets (Manual):**
    
    - Since `.env` files are ignored, you must manually recreate them using your password manager (Bitwarden) data.
        
    
    Bash
    
    ```
    cd ~/homelab/docker/core
    nano .env 
    # Paste API keys/passwords here
    ```
    
4. **Launch:**
    
    Bash
    
    ```
    docker compose up -d
    ```
    

---

## 4. Troubleshooting

### 4.1. "Submodule" Error (Gray Folder on GitHub)

**Symptom:** You `git clone`d a project (like a theme) inside your main repo. GitHub sees the `.git` folder inside it and treats it as a dead link.

**Fix:**

Bash

```
# 1. Remove the nested .git folder
rm -rf path/to/subfolder/.git

# 2. Remove the folder from Git's index (cache) only
git rm --cached path/to/subfolder

# 3. Re-add the folder as standard files
git add .
git commit -m "Converted submodule to standard files"
```

### 4.2. Credentials Issues (HTTPS)

**Symptom:** Git asks for username/password every single time you push.

**Fix (Cache Credentials):**

Bash

```
# Cache credentials in memory for 1 hour (3600 seconds)
git config --global credential.helper 'cache --timeout=3600'
```

_Alternatively, set up SSH keys for password-less auth._

---

## 5. Security: Detecting Secrets Before Staging

_Objective: Prevent accidental leakage of passwords, API keys, or `.env` contents into the Git history._

### 5.1. The Native Way (Visual Inspection)

|**Command**|**Description**|
|---|---|
|`git diff`|Shows line-by-line changes for **unstaged** files. Read this to ensure you aren't adding a hardcoded password.|
|`git diff --staged`|Shows changes for files you have **already added** (staged) but not yet committed.|

### 5.2. The "Grep" Sweep (Quick Check)

Run this command in your project root to scan for common danger words in modified files.

Bash

```
# Search recursively for "password", "secret", "key", or "token"
# Excludes the .git folder to prevent noise
grep -rnE --exclude-dir=.git "(password|secret|key|token|=)" .
```

- **Output:** Lists filename and line number of matches.
    
- **Action:** If you see a hit in a file that isn't `.env`, **do not commit it.**
    

### 5.3. The "Pro" Way (Gitleaks via Docker)

Since Docker is installed, use **Gitleaks** to scan your directory for high-entropy strings (random characters that look like API keys) without installing new software on the host.

Bash

```
# Run this inside your ~/homelab/docker directory
docker run --rm \
  -v $(pwd):/path \
  zricethezav/gitleaks:latest \
  detect --source="/path" -v
```

- **Success:** "No leaks found."
    
- **Failure:** It prints the specific secret and file location.
    

### 5.4. Safe Workflow Summary

1. **Edit Configs:** Make changes to `compose.yaml`.
    
2. **Scan:**
    
    - _Fast:_ `git diff` (Look for added credentials).
        
    - _Deep:_ Run the Gitleaks docker command.
        
3. **Stage:** `git add .`
    
4. **Verify:** `git status` (Ensure no `.env` files are green).
    
5. **Commit:** `git commit -m "update"`
    

---

## 6. VS Code Integration

### 6.1. Visualizing Diff in VS Code

_How to make `git diff` output readable in the editor._

**Method A: Pipe to Editor (Quick)**

Bash

```
git diff | code -
```

**Method B: Configure as Default Tool (Permanent)**

Bash

```
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'
# Usage:
git difftool
```

### 6.2. VS Code Explorer Indicators

_Understanding the badges and colors in the file explorer._

| **Badge**  | **Color** <font color="#f79646">(gruvbox theme)                             </font> | **Meaning**   | **Action Needed**                    |
| ---------- | ----------------------------------------------------------------------------------- | ------------- | ------------------------------------ |
| **U**      | **<font color="#9bbb59">Green**                                                     | **Untracked** | File is</font> new. Needs `git add`. |
| **M**      | **<font color="#4f81bd">Blue</font>**                                               | **Modified**  | File is edited but not staged.       |
| **A**      | **<font color="#9bbb59">Green</font>**                                              | **Added**     | File is staged and ready to commit.  |
| **D**      | **<font color="#c0504d">Red**                           </font>                     | **Deleted**   | File removed locally.                |
| **(None)** | **<font color="#7f7f7f">Gray**                           </font>                    | **Ignored**   | Matches `.gitignore` rule.           |

- **Yellow Folder:** A file inside this folder has modifications.
    
- **Dot (•):** Unsaved changes in editor (Git doesn't see these yet).
    

---

## 7. VS Code GUI Workflow

_The "Happy Path" using the Source Control Tab (Ctrl+Shift+G)._

1. **Edit & Save:** Make changes and save (`Ctrl+S`).
    
2. **Review:** Click the file under **Changes** to see the Side-by-Side Diff.
    
3. **Stage:** Click the **`+` (Plus)** icon next to the file.
    
4. **Commit:** Type message in the input box $\to$ Click **Commit**.
    
5. **Sync:** Click **Sync Changes** (Auto Pulls & Pushes).