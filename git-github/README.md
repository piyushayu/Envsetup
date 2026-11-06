# Git & GitHub — Setup & Commands

> Quick commands: [CHEATSHEET.md](CHEATSHEET.md)

Git is the version control system; GitHub hosts Git repositories. This guide covers first-time setup, SSH auth, the daily command loop, and how to undo almost anything.

---

## 1. Install Git (Windows)

```powershell
winget install Git.Git

# Verify (restart the terminal first)
git --version
```

Or download: https://git-scm.com/download/win

During the installer, the defaults are fine. Two prompts worth choosing deliberately:

| Prompt | Recommendation |
| --- | --- |
| Default editor | VS Code (`code --wait`) or Notepad++ |
| Adjusting PATH | "Git from the command line and also from 3rd-party software" |
| Line endings | **"Checkout Windows-style, commit Unix-style line endings"** (`core.autocrlf=true`) |
| Credential helper | Git Credential Manager (default) |

**macOS:** `brew install git` or `xcode-select --install`
**Linux:** `sudo apt install git`

---

## 2. First-time configuration

```bash
# Identity — must match your GitHub account email if you want commits linked
git config --global user.name "Piyush"
git config --global user.email "you@example.com"

# Default branch name
git config --global init.defaultBranch main

# Line endings (Windows)
git config --global core.autocrlf true

# Better diffs / pull behavior
git config --global pull.rebase false
git config --global core.editor "code --wait"

# Check everything
git config --global --list
```

**macOS/Linux:** use `core.autocrlf input` instead.

> If you use multiple GitHub accounts, set `user.name`/`user.email` per repo (drop `--global`) inside each project.

---

## 3. GitHub authentication

### Option A — SSH key (recommended)

```bash
# 1. Generate a key (press Enter for the default path, no passphrase is OK for local dev)
ssh-keygen -t ed25519 -C "you@example.com"

# 2. Start the agent and add the key
# Windows PowerShell (run as Administrator once):
Set-Service ssh-agent -StartupType Automatic
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519

# macOS/Linux:
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

```bash
# 3. Copy the public key
# Windows:
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | clip
# macOS:
pbcopy < ~/.ssh/id_ed25519.pub
# Linux:
xclip -selection clipboard < ~/.ssh/id_ed25519.pub
```

4. GitHub → **Settings → SSH and GPG keys → New SSH key** → paste → Add.
5. Test:

```bash
ssh -T git@github.com
# Hi <username>! You've successfully authenticated...
```

### Option B — HTTPS + token

GitHub no longer accepts account passwords over HTTPS. Create a **Personal Access Token**:
GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic) → scope `repo`.

Use the token as the password when Git prompts. Let Git Credential Manager store it:

```bash
git config --global credential.helper manager
```

> `Support for password authentication was removed` means you typed your account password — use a token or SSH.

---

## 4. Start a repository

### New local project → GitHub

```bash
cd my-project
git init
git add .
git commit -m "Initial commit"

# Create an empty repo on GitHub first, then:
git remote add origin git@github.com:<username>/<repo>.git
git branch -M main
git push -u origin main
```

### Existing GitHub repo → local

```bash
git clone git@github.com:<username>/<repo>.git
# or HTTPS
git clone https://github.com/<username>/<repo>.git

cd <repo>
npm install        # if it's a Node project
```

### Add an existing repo as a remote

```bash
git remote add origin <url>
git remote -v
```

---

## 5. The daily loop

```bash
git status                     # what changed
git diff                       # unstaged changes
git diff --staged              # staged changes

git add <file>                 # stage one file
git add .                      # stage everything
git add -p                     # stage hunk by hunk (recommended)

git commit -m "feat: add login page"

git pull                       # get remote changes first
git push                       # publish

git log --oneline --graph --decorate -20
```

### Commit message convention (Conventional Commits)

```
<type>(<scope>): <subject>

feat: add checkout page
fix: prevent crash on empty cart
docs: update README
refactor: extract auth helper
test: add payment tests
chore: bump dependencies
```

Keep the subject under ~72 characters, imperative mood ("add", not "added").

---

## 6. Branching and merging

```bash
git branch                          # list local branches
git branch -a                       # include remote branches
git switch -c feature/login         # create + switch
git switch main                     # switch
git branch -d feature/login         # delete after merge

# Merge a branch into main
git switch main
git pull
git merge feature/login

# Rebase instead (linear history)
git switch feature/login
git rebase main
```

### Pull request flow (GitHub)

```bash
git switch -c feature/login
# ... work ...
git add .
git commit -m "feat: add login page"
git push -u origin feature/login
```

Then on GitHub → **Compare & pull request** → review → merge → delete the branch.

```bash
# Sync your local main after the PR is merged
git switch main
git pull
git branch -d feature/login
```

### Merge conflicts

```bash
git status                  # lists conflicted files
# edit the files, remove <<<<<<< ======= >>>>>>> markers
git add <resolved-file>
git commit                  # completes the merge
# or abort:
git merge --abort
```

---

## 7. Undo reference

| Goal | Command | Safe? |
| --- | --- | --- |
| Discard changes in one file | `git restore <file>` | ⚠️ loses edits |
| Unstage a file (keep edits) | `git restore --staged <file>` | ✅ |
| Amend the last commit message | `git commit --amend -m "new message"` | ⚠️ rewrites history |
| Undo last commit, keep changes staged | `git reset --soft HEAD~1` | ✅ |
| Undo last commit, keep changes unstaged | `git reset HEAD~1` | ✅ |
| Undo last commit, **discard** changes | `git reset --hard HEAD~1` | ❌ destructive |
| Revert a pushed commit (new commit) | `git revert <hash>` | ✅ safe on shared branches |
| Stash work in progress | `git stash push -m "wip"` | ✅ |
| Restore stashed work | `git stash pop` | ✅ |
| List stashes | `git stash list` | ✅ |
| See who changed a line | `git blame <file>` | ✅ |
| Find the commit that broke something | `git bisect start` | ✅ |
| Recover a deleted branch | `git reflog` then `git switch -c <name> <hash>` | ✅ |

> Rule: **never `reset --hard` or force-push a shared branch.** Use `revert` on `main`.

### Force-push safely (your own branch only)

```bash
git push --force-with-lease
```

---

## 8. GitHub CLI (optional but handy)

```powershell
winget install GitHub.cli
gh auth login

gh repo create my-app --private --source=. --push
gh pr create --fill
gh pr list
gh pr checkout 42
gh issue list
```

---

## 9. `.gitignore` templates

**Node / React / Next.js**

```gitignore
node_modules/
dist/
build/
.next/
out/
coverage/

.env
.env.local
.env.*.local

*.log
npm-debug.log*
.DS_Store
Thumbs.db
.vscode/*
!.vscode/extensions.json
.idea/

# Prisma
/generated
```

**Python**

```gitignore
__pycache__/
*.py[cod]
.venv/
venv/
.env
.pytest_cache/
.mypy_cache/
```

> `.env.example` **should** be committed; `.env` should not.
> If a file was already committed, `.gitignore` won't hide it: `git rm --cached <file>`.

---

## 10. Gotchas

| Symptom | Cause / fix |
| --- | --- |
| `fatal: not a git repository` | You're outside the project, or `git init` was never run. `cd` into it. |
| `Permission denied (publickey)` | SSH key not added to GitHub/agent. `ssh -T git@github.com` to diagnose. |
| `Support for password authentication was removed` | Use a PAT or SSH — not your account password. |
| `Updates were rejected because the remote contains work you do not have` | `git pull` first (or `git pull --rebase`), then push. |
| `fatal: refusing to merge unrelated histories` | Cloned a repo that already had commits: `git pull origin main --allow-unrelated-histories`. |
| Every line shows as changed | Line-ending mismatch. Set `core.autocrlf true` (Windows) / `input` (mac/linux), then renormalize: `git add --renormalize .`. |
| `detached HEAD` | You checked out a commit/hash. `git switch -c new-branch` to keep work, or `git switch main` to leave. |
| Committed a secret | Rotate the secret immediately. Removing it from history requires `git filter-repo` — assume it's compromised. |
| Large file rejected (>100 MB) | Use Git LFS: `git lfs install` then `git lfs track "*.psd"`, commit `.gitattributes`. |
| Wrong author on commits | Fix `user.name`/`user.email`, then `git commit --amend --reset-author` (last commit) or rebase for older ones. |
| `src refspec main does not match any` | No commit yet, or the branch is named `master`. `git branch -M main` then push. |
| CRLF warnings on every commit | Harmless, but fix with a `.gitattributes` file (below). |

### `.gitattributes` (normalize line endings)

```gitattributes
* text=auto eol=lf
*.bat text eol=crlf
*.cmd text eol=crlf
*.ps1 text eol=crlf
```

---

## 11. Official docs

- Git: https://git-scm.com/doc
- Pro Git book (free): https://git-scm.com/book/en/v2
- GitHub Docs: https://docs.github.com/
- SSH troubleshooting: https://docs.github.com/en/authentication/troubleshooting-ssh
- Conventional Commits: https://www.conventionalcommits.org/
