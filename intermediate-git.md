### Intermediate Git — Summary

**Branching strategy**

Two approaches — know both, use GitHub Flow for solo and small team work:

| | Git Flow | GitHub Flow |
|---|---|---|
| Permanent branches | `main` + `develop` | `main` only |
| Best for | Large teams, scheduled releases | Continuous delivery, solo |
| Rule | Never commit directly to `main` | Never commit directly to `main` |

```bash
git switch -c feature/<name>     # create and switch to feature branch
git switch master                # go back to main branch
git branch                       # list all branches
git branch -d <name>             # delete merged branch
git branch -D <name>             # force delete unmerged branch
```

---

**Merging vs rebasing**

```bash
# Merge — joins histories, creates a merge commit
git switch master
git merge feature/my-feature

# Rebase — replays your commits on top of another branch, linear history
git switch feature/my-feature
git rebase master                # rebase feature onto latest master
git switch master
git merge feature/my-feature    # now a clean fast-forward
```

| | Merge | Rebase |
|---|---|---|
| History | Preserves true branching | Clean linear history |
| Creates | Merge commit | Replayed commits with new hashes |
| Safe on shared branches | Yes | Never — rewrites history |
| Best used | Default choice | Before opening a PR to clean up |

**Golden rule:** never rebase commits already pushed to a remote others may have pulled.

---

**Resolving merge conflicts**

```bash
git merge <branch>               # if conflict occurs:
# 1. Open the conflicted file — edit out the markers manually
# 2. Keep what's correct, delete <<<<<<< ======= >>>>>>> lines
git add <file>                   # mark as resolved
git commit                       # finalize the merge
git merge --abort                # escape hatch — cancel the whole merge
```

Conflict markers explained:
```
<<<<<<< HEAD
your current branch version
=======
incoming branch version
>>>>>>> feature-branch
```

---

**Pull Request workflow**

```bash
# 1. Create and push a feature branch
git switch -c feature/<name>
# ... do work, commit ...
git push origin feature/<name>

# 2. Open PR on GitHub
#    Base: master ← Compare: feature/<name>
#    Add title, description, review the Files Changed tab

# 3. After merge on GitHub
git switch master
git pull origin master           # sync local master
git remote prune origin          # remove stale remote branch refs
git branch -d feature/<name>     # delete local branch
```

PR merge options on GitHub:
| Option | What it does |
|---|---|
| Merge commit | Preserves all commits + adds merge commit |
| Squash and merge | Combines all PR commits into one |
| Rebase and merge | Replays commits linearly, no merge commit |

---

**.gitignore**

```bash
# Common patterns
.env                             # secrets — never commit
node_modules/                    # dependencies — reinstallable
dist/                            # build output
*.log                            # any log file
Thumbs.db                        # Windows OS clutter
.DS_Store                        # Mac OS clutter
.vscode/                         # editor settings
```

Key rules:
- `.gitignore` only works on **untracked** files
- If a file is already committed, untrack it first: `git rm --cached <file>`
- The `.gitignore` file itself **should be committed** so the whole team shares the same rules

---

**git stash**

```bash
git stash push -u -m "description"   # stash everything including untracked (-u)
git stash list                        # see all stashes
git stash pop                         # restore most recent stash, remove from stack
git stash drop stash@{n}             # delete a specific stash without restoring
```

Without `-u`, new untracked files are left behind. Always use `-u` to stash everything.

---

**git cherry-pick**

```bash
git cherry-pick <hash>           # copy a single commit onto current branch
```

- The copied commit gets a **new hash** — same change, different parent
- The original commit stays where it was
- Use when you need one specific fix from a branch without merging everything else

---

**reset vs revert**

```bash
git revert <hash>                # safe — adds a new undo commit, history preserved
git reset --soft <hash>          # remove commits, keep changes staged
git reset --mixed <hash>         # remove commits, keep changes unstaged (default)
git reset --hard <hash>          # remove commits, DELETE all changes permanently
```

| | revert | reset --soft | reset --mixed | reset --hard |
|---|---|---|---|---|
| Commits | Adds new | Removed | Removed | Removed |
| Staged changes | Preserved | Kept staged | Unstaged | **Gone** |
| Working directory | Preserved | Untouched | Untouched | **Gone** |
| Safe on pushed commits | ✅ Yes | ❌ Never | ❌ Never | ❌ Never |

**The rule:** if a commit has been pushed to a shared remote, always use `revert`. Reserve `reset` for local-only commits.