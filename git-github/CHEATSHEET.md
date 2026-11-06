# Git & GitHub — Cheat Sheet

Full guide: [README.md](README.md)

## First time

```bash
git config --global user.name "Piyush"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global core.autocrlf true   # Windows
```

## SSH key (one time)

```bash
ssh-keygen -t ed25519 -C "you@example.com"
ssh -T git@github.com
```

Add the contents of `~/.ssh/id_ed25519.pub` to GitHub → Settings → SSH and GPG keys.

## New repo → GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:<user>/<repo>.git
git branch -M main
git push -u origin main
```

## Clone

```bash
git clone git@github.com:<user>/<repo>.git
```

## Daily

```bash
git status
git add .
git add -p                 # stage hunk by hunk
git commit -m "feat: add login page"
git pull
git push
git log --oneline --graph -20
```

## Branches

```bash
git switch -c feature/login
git switch main
git merge feature/login
git branch -d feature/login
git push -u origin feature/login
```

## Undo

```bash
git restore <file>                   # discard changes in a file
git restore --staged <file>          # unstage
git commit --amend -m "new message"  # fix last commit
git reset --soft HEAD~1              # undo commit, keep changes
git revert <hash>                    # safe undo on shared branch
git stash                            # park changes
git stash pop                        # restore
git reflog                           # recover lost commits
```

## `.gitignore`

```gitignore
node_modules/
.next/
dist/
.env
.env.local
```

## Conventional commits

```
feat: add checkout page
fix: prevent crash on empty cart
docs: update README
refactor: extract auth helper
chore: bump dependencies
```

## Health check

```bash
git --version && ssh -T git@github.com
```
