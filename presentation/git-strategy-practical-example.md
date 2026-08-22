---
marp: true
theme: default
paginate: true
header: 'Common Git Strategies'
footer: 'Clear examples for each type'
style: |
  section { font-size: 26px; }
  h1 { color: #2563eb; }
  h2 { color: #1e40af; }
  code { background: #f1f5f9; font-size: 20px; }
  table { font-size: 20px; }
---

# Common Git Strategies
## Clear Explanation + One Example for Each Type

**Same story in every example: add a Discount Coupon to an online shop**

---

# What You Will Learn

By the end you can answer:

1. What is a Git strategy?
2. What are the 6 common types?
3. How does each one handle the **same feature**?
4. Which one should **your team** choose?

---

# What Is a Git Strategy?

A Git strategy is a **team agreement** about branches.

It answers 4 simple questions:

| Question | Example answer |
|----------|----------------|
| Where is stable code? | `main` |
| Where do I work? | `feature/coupon` |
| How do we release? | merge PR, then deploy |
| How do we fix production? | `hotfix/*` branch |

---

# One Story for All Examples

**App:** Online shop  
**Team:** 4 developers  
**Task:** Add **Discount Coupon** feature

We will show this same task in:

1. Feature Branch
2. GitHub Flow
3. Git Flow
4. GitLab Flow
5. Trunk-Based
6. Forking

So you can compare them easily.

---

# The 6 Types at a Glance

| Type | In one sentence |
|------|-----------------|
| **1. Feature Branch** | One branch per change, then merge with a PR |
| **2. GitHub Flow** | Short branch → merge to `main` → deploy now |
| **3. Git Flow** | `develop` for work, `main` for releases, extra release/hotfix branches |
| **4. GitLab Flow** | Promote code by merging: `main` → `staging` → `production` |
| **5. Trunk-Based** | Tiny daily merges to `main`, hide unfinished work with flags |
| **6. Forking** | Work in your own copy of the repo, then send a PR |

---

# Type 1: Feature Branch Workflow

## What it is

Every new change lives on its **own branch**.  
Nobody commits directly to `main`.  
A Pull Request (PR) is required.

```
main
 └── feature/discount-coupon
 └── feature/new-cart
 └── fix/login-bug
```

---

# Type 1: When to use

**Use when:**
- Team is learning Git
- You want simple reviews
- You do not need complex release versions yet

**Do not use alone when:**
- You need strict version numbers like `v1.4.0`
- Many environments must be promoted in order

---

# Type 1: Clear Example (Discount Coupon)

**Goal:** Sara adds a coupon box on checkout.

```bash
# 1. Start from latest main
git checkout main
git pull

# 2. Create a branch for this one feature
git checkout -b feature/discount-coupon

# 3. Write code, then save it
git add .
git commit -m "Add discount coupon field on checkout"

# 4. Publish the branch
git push -u origin feature/discount-coupon
```

Then Sara opens a PR: **`feature/discount-coupon` → `main`**

---

# Type 1: What happens next

1. CI runs tests
2. Ahmed reviews the PR
3. Sara fixes review comments
4. PR is merged into `main`
5. Branch `feature/discount-coupon` is deleted

**Result:** `main` now has the coupon feature.

This is the base of almost every other strategy.

---

# Type 2: GitHub Flow

## What it is

Same as Feature Branch, plus one extra rule:

> **`main` is always ready to go to production.**

```
main  (can deploy at any time)
 └── feature/discount-coupon   (lives only 1 day or a few days)
```

Merge to `main` = you can deploy.

---

# Type 2: Rules (very simple)

1. Create a short-lived branch from `main`
2. Open a PR early (even before finished)
3. CI must be green
4. Review + merge
5. Deploy `main` immediately

No `develop` branch.  
No `release` branch.

---

# Type 2: Clear Example (same coupon)

**Monday 09:00** Sara starts:

```bash
git checkout main
git pull
git checkout -b feature/discount-coupon
```

**Monday 11:00** she opens a PR (CI starts).

**Monday 12:30** Ahmed approves.

**Monday 12:40** merge to `main`.

**Monday 12:45** production auto-deploys coupon feature.

---

# Type 2: Why teams like it

| Good | Watch out |
|------|-----------|
| Very simple | Needs good tests |
| Fast delivery | Incomplete features can reach `main` if you are not careful |
| Easy for new developers | Not ideal for App Store versioned releases |

**Best for:** websites and APIs that deploy often.

---

# Type 3: Git Flow

## What it is

Git Flow uses **two long-lived branches**:

| Branch | Meaning |
|--------|---------|
| `main` | Production history + version tags (`v1.2.0`) |
| `develop` | Next release being prepared |

Plus extra branches:

- `feature/*` from `develop`
- `release/*` to freeze a version
- `hotfix/*` for production emergencies

---

# Type 3: Picture

```
feature/discount-coupon
        │
        ▼
     develop  ──► release/1.3.0 ──► main (tag v1.3.0)
                                      │
                                      ▼
                                  hotfix/1.3.1
```

**Important:** features merge to `develop`, **not** to `main`.

---

# Type 3: Example A — Build the feature

Sara does **not** branch from `main`. She branches from `develop`.

```bash
git checkout develop
git pull
git checkout -b feature/discount-coupon

git add .
git commit -m "Add discount coupon on checkout"
git push -u origin feature/discount-coupon
```

PR is: **`feature/discount-coupon` → `develop`**

After merge, coupon is in `develop` only. Production is still old until a release.

---

# Type 3: Example B — Release version 1.3.0

The team wants to ship coupon + other finished work.

```bash
git checkout develop
git checkout -b release/1.3.0

# only version bump and small bug fixes here
git commit -m "Prepare release 1.3.0"

# 1) put it on production branch
git checkout main
git merge --no-ff release/1.3.0
git tag -a v1.3.0 -m "Release 1.3.0"

# 2) copy release fixes back to develop
git checkout develop
git merge --no-ff release/1.3.0
git branch -d release/1.3.0
```

Now production is `v1.3.0`.

---

# Type 3: Example C — Production is broken

Users cannot apply the coupon. Fix without waiting for next big release.

```bash
git checkout main
git checkout -b hotfix/1.3.1

git commit -m "Fix coupon code case-sensitivity"
git checkout main
git merge --no-ff hotfix/1.3.1
git tag -a v1.3.1 -m "Hotfix 1.3.1"

git checkout develop
git merge --no-ff hotfix/1.3.1
```

Hotfix goes to **production first**, then back to `develop`.

---

# Type 3: When to use Git Flow

**Use when:**
- You release every 2–4 weeks
- You need versions: `v1.3.0`, `v1.3.1`
- Mobile apps / desktop apps / products with a release date

**Avoid when:**
- You deploy many times every day
- Team is small and Git Flow feels too heavy

---

# Type 4: GitLab Flow

## What it is

Use feature branches, then **promote the same code through environment branches**.

```
feature/discount-coupon
        │
        ▼
      main  ──merge──►  staging  ──merge──►  production
      (dev)              (QA test)            (live users)
```

The branch name = the environment.

---

# Type 4: Clear Example (same coupon)

**Step 1 — build**
```bash
git checkout main
git checkout -b feature/discount-coupon
git commit -m "Add discount coupon"
# PR into main
```

**Step 2 — send to QA**
```bash
git checkout staging
git merge main
# now QA tests shop.staging.company.com
```

**Step 3 — send to live**
```bash
git checkout production
git merge staging
# now shop.company.com is updated
```

---

# Type 4: Why this is easy to explain

You can say in a meeting:

- “Is it on `main`?” → developers have it
- “Is it on `staging`?” → QA can test it
- “Is it on `production`?” → customers have it

**Best for:** teams that think in environments.  
**Watch out:** `staging` and `production` can drift if people commit extra fixes only there.

**Rule:** never develop on `production`. Always merge forward.

---

# Type 5: Trunk-Based Development

## What it is

Almost all work goes to one branch: **`main` (the trunk)**.

Branches are tiny and die the same day (or in 1–2 days).

Unfinished features are hidden with a **feature flag**.

```
main
 ├── coupon-flag-off   (merged Monday)
 └── coupon-flag-on    (later, still small PR)
```

---

# Type 5: Clear Example (same coupon)

Sara cannot finish coupon in one day, but she still merges.

```bash
git checkout main
git checkout -b feat-coupon-behind-flag

# code exists, but hidden
# if NEW_COUPON == false  → old checkout
# if NEW_COUPON == true   → new coupon UI

git commit -m "Add coupon UI behind NEW_COUPON flag"
# PR to main the same day
```

Production config:

```text
NEW_COUPON=false
```

Users do not see coupon yet, but code is already on `main`.

---

# Type 5: Enable the feature later

When QA is happy:

```text
NEW_COUPON=true
```

No giant merge. Just turn the flag on (maybe for 10% of users first).

If something breaks: set `NEW_COUPON=false` again.

**Best for:** teams with strong CI and frequent deploys.  
**Hard if:** no tests, or people keep 3-week branches.

---

# Type 6: Forking Workflow

## What it is

You do **not** push branches to the company/original repo.

You copy the repo (a **fork**), work there, then open a PR back.

```
company/shop          ← original (upstream)
 └── sara/shop        ← Sara's fork
      └── feature/discount-coupon
            └── Pull Request to company/shop
```

Used a lot in **open source**.

---

# Type 6: Clear Example (same coupon)

Sara is an outside contributor. She cannot push to `company/shop`.

```bash
# 1. Fork on GitHub website, then clone HER fork
git clone https://github.com/sara/shop.git
cd shop

# 2. Remember the original repo
git remote add upstream https://github.com/company/shop.git

# 3. Work on a branch in HER fork
git checkout -b feature/discount-coupon
git commit -m "Add discount coupon"
git push -u origin feature/discount-coupon
```

Then she opens PR:

**`sara/shop:feature/discount-coupon` → `company/shop:main`**

---

# Type 6: When to use

**Use when:**
- Open source project
- Students/vendors should not have write access
- You want every outsider change reviewed

**Not needed when:**
- Whole team already has access to one company repo
  (then Feature Branch / GitHub Flow is enough)

---

# Same Feature: Side-by-Side

**Task:** Add Discount Coupon

| Strategy | Where Sara branches | Where she merges | When users see it |
|----------|---------------------|------------------|-------------------|
| Feature Branch | from `main` | PR to `main` | after deploy from `main` |
| GitHub Flow | from `main` | PR to `main` | right after merge/deploy |
| Git Flow | from `develop` | `develop` then `release` then `main` | on release day (`v1.3.0`) |
| GitLab Flow | from `main` | `main` → `staging` → `production` | after production merge |
| Trunk-Based | from `main` | tiny PR to `main` + flag | when flag is turned on |
| Forking | from her fork | PR to original repo | after maintainers merge |

---

# Which One Should You Choose?

| If your situation is... | Choose |
|-------------------------|--------|
| Small team, learning Git | **Feature Branch** |
| Website, deploy many times a week | **GitHub Flow** |
| App Store / versioned product | **Git Flow** |
| Staging then production promotion | **GitLab Flow** |
| Strong CI, deploy many times a day | **Trunk-Based** |
| Outside contributors | **Forking** |

---

# Super Simple Decision

Ask only 2 questions:

1. **Do we deploy every day?**  
   Yes → GitHub Flow or Trunk-Based  
   No → Git Flow or GitLab Flow

2. **Do outsiders contribute?**  
   Yes → add Forking  
   No → stay on one repo

Start with the **simplest** option that works.

---

# Shared Good Habits (all strategies)

1. Do not commit directly to `main`
2. Use Pull Requests
3. Run tests in CI before merge
4. Name branches clearly: `feature/discount-coupon`
5. Delete branch after merge
6. Write clear commit messages: `feat: add discount coupon`

---

# Common Mistakes

| Mistake | Why it is bad | Do this instead |
|---------|----------------|-----------------|
| Work for 3 weeks on one branch | huge conflicts | smaller PRs |
| Commit to `production` directly | bypasses review | merge from `staging` |
| Use Git Flow “because it is famous” | extra complexity | pick by release style |
| Forget to merge hotfix back to `develop` | bug returns later | always merge hotfix to both |
| No tests on PRs | broken `main` | block merge until CI is green |

---

# Mini Practice

Match the situation:

1. “We ship iOS app every month as `v2.4.0`.”
2. “We deploy the website 8 times today.”
3. “QA must approve on staging before live.”
4. “A stranger on GitHub wants to add a coupon.”
5. “Our team is 4 people, first time using PRs.”

---

# Practice Answers

1. **Git Flow**
2. **GitHub Flow** or **Trunk-Based**
3. **GitLab Flow**
4. **Forking**
5. **Feature Branch**

---

# Cheat Sheet

```
1. Feature Branch : one branch + PR
2. GitHub Flow    : short branch + deploy main now
3. Git Flow       : develop + release + hotfix + tags
4. GitLab Flow    : main → staging → production
5. Trunk-Based    : tiny merges + feature flags
6. Forking        : work in your copy, PR back
```

Remember: all of them are just different ways to do **the same coupon feature safely**.

---

# Final Summary

- A Git strategy is a **team map** for branches
- Learn the 6 types with **one repeated example**
- GitHub Flow = simple + fast
- Git Flow = versions + planned releases
- GitLab Flow = environment promotion
- Trunk-Based = smallest changes, all day
- Forking = outside contributors

Choose simple first. Add more branches only when you truly need them.

---

# Thank You

References:
- GitHub Flow: https://docs.github.com/en/get-started/using-github/github-flow
- Git Flow: https://nvie.com/posts/a-successful-git-branching-model/
- Trunk-Based Development: https://trunkbaseddevelopment.com/
