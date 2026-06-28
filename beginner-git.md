### Beginner Git — Summary

**Core concept**

Git is a version control tool that runs locally. GitHub is a hosting service for Git repositories. They are separate things — Git works without the internet.

---

**The three areas — the most important mental model**

```
Working Directory  →  Staging Area  →  Repository
  (your files)        (git add)        (git commit)
```

- **Working Directory** — where you edit files
- **Staging Area** — a preparation zone; you choose exactly what goes into the next commit
- **Repository** — permanent history; every commit is a snapshot saved forever

---

**First-time setup**

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor "code --wait"
git config --global --list        # verify
```

---

**Core commands**

| Command | What it does |
|---|---|
| `git init` | Create a new repo in the current folder |
| `git status` | Show the state of working directory and staging area |
| `git diff` | Show unstaged changes line by line |
| `git add <file>` | Move file to staging area |
| `git add .` | Stage everything changed in current directory |
| `git commit -m "message"` | Save staged changes as a permanent snapshot |
| `git log` | Full commit history with author, date, message |
| `git log --oneline` | Compact one-line history view |

---

**Undoing things**

| Command | What it undoes | Loses content? |
|---|---|---|
| `git restore <file>` | Discard unstaged working directory change | Yes — permanent |
| `git restore --staged <file>` | Unstage a file (keeps change in file) | No |
| `git revert <hash>` | Create a new commit that undoes a previous one | No — history preserved |

---

**GitHub setup — SSH (Linux/Mac) vs HTTPS (corporate/Windows)**

```bash
# SSH key generation (Fedora/Linux)
ssh-keygen -t ed25519 -C "you@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub        # copy this to GitHub → Settings → SSH keys
ssh -T git@github.com            # test connection

# HTTPS with Personal Access Token (Windows/corporate network)
# Use token as password when Git prompts during push/pull
```

---

**Syncing local and remote**

```bash
git remote add origin <url>      # connect local repo to GitHub
git remote -v                    # verify remote URL
git push -u origin master        # first push; sets upstream
git push                         # subsequent pushes
git pull origin master           # fetch + merge remote changes
git clone <url>                  # download a repo fresh
git remote prune origin          # remove stale remote branch references
```

---

**Reading `git log --oneline`**

```
9f58434 (HEAD -> master, origin/master) Merge edit-readme
251f94a Update README description on master
c1655ac Update README description on edit-readme branch
```

- **Hash** — unique 7-char ID for each commit
- **HEAD -> master** — you are here; master is your current branch
- **origin/master** — where GitHub's master last synced to