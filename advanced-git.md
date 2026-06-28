### Advanced Git — Summary

**Git Internals**

The four object types stored in `.git/objects/`:

| Object | Contains | Identified by |
|---|---|---|
| blob | Raw file content only — no filename | Hash of content |
| tree | Directory listing — maps filenames to blob/tree hashes | Hash of listing |
| commit | Pointer to a tree + parent hash + author + message | Hash of all above |
| tag | Named pointer to a specific commit | Hash of tag data |

The pointer chain every commit follows:
```
HEAD → refs/heads/master → commit → tree → blobs
```

Key commands for inspecting internals:
- `git cat-file -t <hash>` — show the type of any object (commit/tree/blob)
- `git cat-file -p <hash>` — pretty-print the contents of any object
- `cat .git/HEAD` — see which branch HEAD points to
- `cat .git/refs/heads/master` — see the raw commit hash a branch points to
- `ls .git/objects/` — see the full object store (first 2 chars = folder, remaining 38 = filename)

How Git stays efficient: unchanged files between commits share the same blob hash — Git never duplicates content, it reuses existing objects.

---

**GitHub Actions — CI/CD**

Workflows live in `.github/workflows/*.yml` and trigger automatically on GitHub events.

```yaml
name: CI                          # workflow name shown on GitHub

on:                               # what triggers this workflow
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:                          # job name
    runs-on: ubuntu-latest        # fresh VM GitHub spins up per run

    steps:
      - uses: actions/checkout@v4        # clone the repo into the VM
      - uses: actions/setup-python@v5   # install Python
        with:
          python-version: '3.11'
      - run: python hello.py             # run your command
```

Key concepts:
- Each run gets a **fresh, isolated VM** — destroyed after the job completes
- A non-zero exit code from any `run:` step **fails the pipeline**
- In real projects, `run:` steps are `pytest`, `npm test`, linters, build commands
- Failed pipelines can **block PR merges** — broken code can't enter the codebase silently

---

**Git Hooks**

Scripts that run automatically on your local machine when Git events fire. Live in `.git/hooks/` — never committed or pushed (local only).

| Hook | Fires | Common use |
|---|---|---|
| `pre-commit` | Before commit is created | Block debug code, run linter |
| `commit-msg` | After message is typed | Enforce message format |
| `pre-push` | Before push to remote | Run tests locally first |

Activating a hook — remove `.sample` and make executable:
```bash
chmod +x .git/hooks/pre-commit
```

Sharing hooks with a team — store in a versioned folder and point Git to it:
```bash
# Store hooks in .githooks/ (committed to repo)
git config core.hooksPath .githooks
```

Exit codes control everything: `exit 0` = allow Git to proceed, `exit 1` = block the operation.