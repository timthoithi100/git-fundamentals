### Monorepos, Submodules & Large Repo Strategies

**The problem these solve**

As projects grow, you face a structural question: how do you organize code across multiple related projects? Three common approaches exist, each with tradeoffs.

---

### Monorepo

A **monorepo** (monolithic repository) stores multiple distinct projects in a single Git repository. For example, a company might keep their frontend, backend, mobile app, and shared libraries all in one repo:

```
my-company/
├── apps/
│   ├── web/          ← React frontend
│   ├── api/          ← Node.js backend
│   └── mobile/       ← React Native app
├── packages/
│   ├── ui-components/  ← shared component library
│   └── utils/          ← shared utilities
└── package.json
```

**Advantages:**
- One place for all code — easy cross-project changes in a single commit
- Shared code (libraries, utilities) without publishing npm packages
- Unified CI/CD pipeline
- Easier to keep dependencies in sync

**Disadvantages:**
- Repo gets large and slow over time
- `git clone` downloads everything even if you only need one app
- Requires tooling (Nx, Turborepo, Lerna) to manage builds efficiently

Companies that use monorepos: Google, Meta, Microsoft, Vercel.

---

### Git Submodules

A **submodule** is a Git repo nested inside another Git repo. The parent repo stores a reference to a specific commit in the child repo — not the child's files directly.

```bash
# Add a submodule
git submodule add https://github.com/someuser/shared-lib.git libs/shared-lib

# Clone a repo that has submodules
git clone --recurse-submodules <url>

# If you already cloned without --recurse-submodules
git submodule init
git submodule update

# Update a submodule to its latest commit
git submodule update --remote
```

After adding a submodule, two things appear in your repo:
- A `.gitmodules` file recording the submodule URL and path
- A special entry in your tree pointing to a specific commit in the child repo

**When to use submodules:**
- You want to include another team's repo as a dependency at a pinned version
- The dependency is a separate project with its own release cycle

**Why submodules are painful:**
- Teammates must remember to run `git submodule update` after pulling
- Easy to accidentally leave submodules in a detached HEAD state
- Updates require explicit commands — nothing is automatic

Submodules have a reputation for being confusing. Many teams avoid them in favor of package managers or monorepo tooling.

---

### Large Repo Strategies

When a repo accumulates years of history, large binary files, or thousands of files, standard Git starts slowing down. Several strategies exist:

**Git LFS (Large File Storage)**

Git is not designed for large binary files (images, videos, datasets, compiled artifacts). Every version of a binary is stored in full in `.git/objects` — a 100MB file changed 50 times means 5GB in your repo.

Git LFS solves this by replacing large files in your repo with tiny pointer files, while storing the actual content on a separate LFS server:

```bash
# Install Git LFS
git lfs install

# Track all PNG files with LFS
git lfs track "*.png"

# This creates/updates .gitattributes — commit it
git add .gitattributes
git commit -m "Track PNGs with Git LFS"

# From here, git add/commit works normally
# Git LFS handles the rest transparently
```

**Shallow clones**

When you only need recent history (common in CI/CD pipelines):

```bash
# Clone only the last 1 commit — fastest possible clone
git clone --depth 1 <url>

# Clone only the last 10 commits
git clone --depth 10 <url>
```

GitHub Actions uses shallow clones by default (`actions/checkout@v4` clones with `--depth 1`) because CI pipelines rarely need full history.

**Sparse checkout**

When you only need a subset of files from a large monorepo:

```bash
git clone --no-checkout <url>
cd repo
git sparse-checkout init
git sparse-checkout set apps/web    # only checkout the web app folder
git checkout master
```

Only the files under `apps/web` will be downloaded to your working directory.

---

### Summary

| Strategy | Use when |
|---|---|
| Monorepo | Multiple related projects, shared code, one team owns everything |
| Submodules | Pinning an external repo dependency at a specific version |
| Git LFS | Repo contains large binary files (images, video, datasets) |
| Shallow clone | CI/CD pipelines, speed matters, full history not needed |
| Sparse checkout | Working in one section of a large monorepo |