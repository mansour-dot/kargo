---
marp: true
theme: default
paginate: true
header: 'Git Strategies — Practical Examples'
footer: 'Common branching models explained simply'
style: |
  section { font-size: 26px; }
  h1 { color: #2563eb; }
  h2 { color: #1e40af; }
  code { background: #f1f5f9; font-size: 21px; }
  table { font-size: 21px; }
---

# Git Strategies
## Common Types Explained Simply

**Practical examples for teams of any size**

---

# Who Is This For?

This guide is for you if:

- You hear words like **GitFlow**, **GitHub Flow**, **trunk-based**
- Your team is confused about **which branch to use**
- You want a **simple comparison** with real examples

**No expert Git knowledge required.**

---

# What Is a Git Strategy?

A **Git strategy** (branching model) answers:

1. Which branches do we keep?
2. Where do developers create changes?
3. How do we merge and release?
4. How do we fix production bugs?

Without a strategy: messy branches, broken `main`, slow releases.

---

# Big Picture: 6 Common Strategies

| Strategy | Simple idea |
|----------|-------------|
| **Feature Branch** | One branch per feature + PR |
| **GitHub Flow** | Short branches → merge to `main` → deploy |
| **Git Flow** | Long-lived `develop` + release/hotfix branches |
| **GitLab Flow** | Environment branches (`staging`, `production`) |
| **Trunk-Based** | Tiny branches, frequent merge to `main` |
| **Forking** | Contributors work in their own fork |

---

# Strategy 1: Feature Branch Workflow

**Idea:** every change gets its own branch.

```
main
 └── feature/login-page
 └── feature/payment-api
 └── bugfix/cart-total
```

1. Create branch from `main`
2. Commit your work
3. Open Pull Request / Merge Request
4. Review → merge → delete branch

---

# Example: Feature Branch

```bash
git checkout main
git pull
git checkout -b feature/add-search

# ... code changes ...
git add .
git commit -m "Add product search box"
git push -u origin feature/add-search
```

Then open a PR: `feature/add-search` → `main`

**Best for:** small/medium teams learning Git collaboration.

---

# Strategy 2: GitHub Flow

**Idea:** `main` is always deployable.

```
main (always ready to deploy)
 └── feature/xyz   (short-lived)
```

Rules:
1. Create a branch from `main`
2. Add commits
3. Open Pull Request
4. Review + CI checks
5. Merge to `main`
6. Deploy immediately (or almost)

---

# Example: GitHub Flow Day

**09:00** — Start `feature/dark-mode`
**11:00** — Open PR, CI runs tests
**12:00** — Reviewer approves
**12:10** — Merge to `main`
**12:15** — Auto-deploy to production

**Best for:** web apps with continuous delivery.
**Avoid if:** you need long release cycles and versioned releases.

---

# Strategy 3: Git Flow

**Idea:** clear roles for each long-lived branch.

```
main        = production-ready releases
develop     = integration branch for next release
feature/*   = new work (from develop)
release/*   = prepare a release
hotfix/*    = urgent production fix
```

Created by Vincent Driessen (popular classic model).

---

# Git Flow: Branch Roles

| Branch | Purpose |
|--------|---------|
| `main` | Stable production code + version tags |
| `develop` | Latest completed features for next release |
| `feature/*` | One feature in progress |
| `release/*` | Freeze features, fix bugs, bump version |
| `hotfix/*` | Emergency fix from `main` |

---

# Example: Git Flow Feature

```bash
git checkout develop
git pull
git checkout -b feature/user-profile

# work...
git commit -m "Add user profile page"
git checkout develop
git merge --no-ff feature/user-profile
git branch -d feature/user-profile
```

Feature merges into **`develop`**, not directly into `main`.

---

# Example: Git Flow Release

```bash
git checkout develop
git checkout -b release/1.2.0

# bump version, fix small bugs only
git commit -m "Bump version to 1.2.0"

git checkout main
git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Release 1.2.0"

git checkout develop
git merge --no-ff release/1.2.0
git branch -d release/1.2.0
```

Release goes to **both** `main` and `develop`.

---

# Example: Git Flow Hotfix

Production bug found on `v1.2.0`:

```bash
git checkout main
git checkout -b hotfix/1.2.1

# fix the bug
git commit -m "Fix login crash"

git checkout main
git merge --no-ff hotfix/1.2.1
git tag -a v1.2.1 -m "Hotfix 1.2.1"

git checkout develop
git merge --no-ff hotfix/1.2.1
```

**Best for:** scheduled releases, versioned products, mobile/app stores.

---

# Strategy 4: GitLab Flow

**Idea:** combine feature branches with **environment branches**.

```
main
 └── feature/xyz   → merge to main
main ───────────────▶ staging ───────────────▶ production
```

Or:

```
main → pre-production → production
```

Promotion between environments is done by merging branches.

---

# Example: GitLab Flow Promotion

```bash
# feature merged to main (after PR)
git checkout main
git merge feature/checkout-button

# promote to staging
git checkout staging
git merge main

# after QA passes, promote to production
git checkout production
git merge staging
```

**Best for:** teams that map branches to environments (staging/prod).
**Note:** environment branches can become hard to keep in sync.

---

# Strategy 5: Trunk-Based Development

**Idea:** everyone merges to one trunk (`main`) many times per day.

```
main (trunk)
 ├── short-lived-branch-1  (hours, not weeks)
 ├── short-lived-branch-2
 └── short-lived-branch-3
```

Rules:
- Branches live **hours/days**, not weeks
- Prefer small changes
- Use feature flags for incomplete work
- Strong CI is required

---

# Example: Trunk-Based + Feature Flag

```bash
git checkout -b add-new-checkout
# implement behind flag NEW_CHECKOUT=false
git commit -m "Add checkout v2 behind feature flag"
# merge same day to main
```

In production config:
```text
NEW_CHECKOUT=false   # hidden
NEW_CHECKOUT=true    # enable for 10% users later
```

**Best for:** high-performing DevOps teams, continuous delivery.
**Hard if:** weak tests / no CI / large long-running branches.

---

# Strategy 6: Forking Workflow

**Idea:** each contributor has their own copy (fork) of the repo.

```
upstream (company repo)
 └── your-fork
      └── feature/fix-docs
           └── Pull Request back to upstream
```

Common in open source (Linux, Kubernetes, many GitHub projects).

---

# Example: Forking Workflow

```bash
# 1) Fork on GitHub UI, then clone YOUR fork
git clone https://github.com/you/project.git
cd project

# 2) Add original repo as upstream
git remote add upstream https://github.com/org/project.git

# 3) Create branch and push to YOUR fork
git checkout -b docs/fix-readme
git push -u origin docs/fix-readme

# 4) Open PR: your-fork → org/project
```

**Best for:** open source and external contributors.

---

# Quick Comparison Table

| Strategy | Speed | Complexity | Typical use |
|----------|-------|------------|-------------|
| Feature Branch | Medium | Low | Most teams starting out |
| GitHub Flow | Fast | Low | SaaS / continuous deploy |
| Git Flow | Medium/Slow | High | Versioned releases |
| GitLab Flow | Medium | Medium | Env-based promotion |
| Trunk-Based | Very fast | Medium | Elite CD teams |
| Forking | Medium | Medium | Open source |

---

# Same Feature in 3 Strategies

Feature: **Add discount coupon**

### GitHub Flow
`feature/coupon` → PR → `main` → deploy

### Git Flow
`feature/coupon` → `develop` → `release/x.y` → `main` + tag

### Trunk-Based
tiny PR to `main` same day, feature flag off until ready

---

# Practical Example: Small Startup

**Team:** 5 developers, deploy many times/week

**Choose:** GitHub Flow or Trunk-Based

```
main
 └── feature/* (short)
```

Why?
- simple
- fast feedback
- less branch maintenance

---

# Practical Example: Mobile App Company

**Team:** releases every 2–4 weeks to App Store

**Choose:** Git Flow

```
main / develop / release/* / hotfix/*
```

Why?
- clear release versions (`v2.3.0`)
- stabilize on release branch
- hotfix path for store emergencies

---

# Practical Example: Open Source Library

**Team:** maintainers + outside contributors

**Choose:** Forking + Feature Branch / GitHub Flow

```
upstream/main
 ← PR from contributor forks
```

Why?
- outsiders cannot push directly
- maintainers review every change

---

# Naming Conventions (Useful Everywhere)

| Branch type | Example names |
|-------------|---------------|
| Feature | `feature/add-search`, `feat/user-avatar` |
| Bugfix | `bugfix/login-crash`, `fix/cart-total` |
| Hotfix | `hotfix/1.2.1-payment-timeout` |
| Release | `release/2.0.0` |
| Chore | `chore/upgrade-node-20` |

Keep names short, clear, and consistent.

---

# Pull Request Best Practices

Good PR:
- small and focused
- clear title: `Add coupon validation`
- linked issue
- tests updated
- CI green before merge

Bad PR:
- 40 files mixed with refactor + feature + typo fixes
- no description
- “please merge ASAP”

---

# Commit Message Tips

Simple useful format:

```text
feat: add coupon code validation
fix: correct tax calculation for carts
docs: explain branching strategy in README
chore: upgrade CI Node version to 20
```

Why?
- readable history
- easy changelog generation
- faster debugging

---

# Common Mistakes

| Mistake | Better approach |
|---------|-----------------|
| Long-lived feature branches (weeks) | Smaller PRs, merge often |
| Commit directly to `main` | Always use PR reviews |
| No branch naming rules | Agree on `feature/`, `fix/` |
| Choosing Git Flow “because famous” | Choose based on release style |
| No CI on PRs | Block merge until tests pass |
| Hotfixing on random branches | Use clear hotfix path |

---

# Decision Guide

Ask these questions:

1. Do we deploy **many times/day**? → GitHub Flow / Trunk-Based
2. Do we ship **versioned releases**? → Git Flow
3. Do branches map to **environments**? → GitLab Flow
4. Do outsiders contribute? → Forking Workflow
5. Are we a small team learning Git? → Feature Branch / GitHub Flow

**Pick the simplest model that fits your release style.**

---

# Migration Tip

Do not jump from chaos → full Git Flow overnight.

Recommended path:
1. Protect `main` (PR required)
2. Use feature branches + reviews
3. Add CI checks
4. Then choose GitHub Flow **or** Git Flow based on release needs

---

# Cheat Sheet

```
Feature Branch = one branch per change + PR
GitHub Flow    = short branches, main always deployable
Git Flow       = main + develop + release + hotfix
GitLab Flow    = promote via environment branches
Trunk-Based    = tiny frequent merges to main
Forking        = work in your fork, PR upstream
```

---

# Practice Questions

1. Which strategy uses `develop` + `release/*`?
2. Which strategy keeps `main` always deployable with short branches?
3. When is forking the best choice?
4. What is the main risk of long-lived feature branches?
5. Which strategy often uses feature flags?

---

# Answer Key

1. **Git Flow**
2. **GitHub Flow** (also Trunk-Based)
3. **Open source / external contributors**
4. **Merge conflicts, delayed feedback, hard reviews**
5. **Trunk-Based Development**

---

# Final Summary

There is no single “best” Git strategy.

- Choose based on **team size**, **release speed**, and **risk**
- Keep rules simple and written down
- Protect `main`, use PRs, run CI
- Start simple; add complexity only when needed

A clear Git strategy makes delivery predictable for everyone.

---

# Thank You

Useful references:
- GitHub Flow: https://docs.github.com/en/get-started/using-github/github-flow
- Git Flow (original): https://nvie.com/posts/a-successful-git-branching-model/
- Trunk-Based Development: https://trunkbaseddevelopment.com/
