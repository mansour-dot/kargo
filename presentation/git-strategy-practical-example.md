---
marp: true
theme: default
paginate: true
header: 'Git / Branching Strategy'
footer: 'How to tell the 6 types apart'
style: |
  section { font-size: 25px; }
  h1 { color: #2563eb; }
  h2 { color: #1e40af; }
  code { background: #f1f5f9; font-size: 19px; }
  table { font-size: 19px; }
---

# Git Strategy = Branching Strategy
## How to tell each type apart

**If they feel the same, look at ONE unique rule per type.**

---

# Why it feels confusing

All 6 types do this:

```
create a branch  →  write code  →  open a PR  →  merge
```

So they look similar.

**They are different in the next question:**

> After the coupon is finished, **where does it go**, and **when do users see it?**

That one answer is how you tell them apart.

---

# The 6 types — one unique rule each

| Type | Unique rule (remember only this) |
|------|-------------------------------------|
| **1. Feature Branch** | One branch per change + PR. **No extra rule** about deploy or versions. |
| **2. GitHub Flow** | Merge to `main` **and deploy production immediately**. |
| **3. Git Flow** | Two long branches: **`develop` = work**, **`main` = released versions only**. |
| **4. GitLab Flow** | Extra branches named after **environments**: `staging`, `production`. |
| **5. Trunk-Based** | Merge to `main` **even if unfinished**. Hide it with a **feature flag**. |
| **6. Forking** | You do **not** push to the company repo. You work in **your copy**. |

If you remember the **unique rule**, you can differentiate them.

---

# Same story for every type

**App:** online shop  
**Task:** add **Discount Coupon**

Ask for each type:

1. Which branch do I create?
2. Which branch do I merge into?
3. When do customers see the coupon?

---

# Picture: the only thing that changes

```
          Feature Branch     →  merge to main           (deploy later, maybe)
          GitHub Flow        →  merge to main           (deploy NOW)
          Git Flow           →  merge to develop         (users wait for a release)
          GitLab Flow       →  main → staging → prod    (users after last merge)
          Trunk-Based       →  merge to main hidden     (users when flag = ON)
          Forking           →  PR from YOUR fork         (not from company clone)
```

---

# Type 1 — Feature Branch

## Unique rule
**One change = one branch + one PR.** That is all.

No `develop`. No “must deploy now”. No feature flag.

```
main
 └── feature/discount-coupon   →  PR  →  main
```

---

# Type 1 — Coupon example

```bash
git checkout main
git checkout -b feature/discount-coupon
# write coupon code
git commit -m "Add discount coupon"
git push -u origin feature/discount-coupon
```

PR: `feature/discount-coupon` → `main`

**Customers see it:** after someone deploys `main` (today or later).  
The strategy **does not force** a deploy time.

---

# Type 1 — Do not confuse with GitHub Flow

They look the same (branch + PR to `main`).

| | Feature Branch | GitHub Flow |
|--|----------------|-------------|
| Branch + PR to `main` | Yes | Yes |
| Must deploy after merge? | **No** | **Yes** |

**Feature Branch** = how you **work**.  
**GitHub Flow** = Feature Branch **+ deploy `main` every time**.

---

# Type 2 — GitHub Flow

## Unique rule
**`main` is always production-ready, and you deploy it right after merge.**

```
main  (live / always deployable)
 └── feature/discount-coupon   →  PR  →  main  →  deploy NOW
```

No `develop`. No `staging` branch. No waiting for “release day”.

---

# Type 2 — Coupon example

09:00 create `feature/discount-coupon`  
11:00 open PR  
12:00 merge to `main`  
12:05 **production website updates**

```bash
git checkout main
git checkout -b feature/discount-coupon
git commit -m "Add discount coupon"
# PR → main, then deploy
```

**Customers see it:** minutes after merge.

---

# Type 2 vs Type 1 (the only difference)

```
Feature Branch:   PR → main           → deploy whenever
GitHub Flow:      PR → main           → deploy immediately
```

Same Git commands.  
Different **team rule after merge**.

Choose **GitHub Flow** for websites that ship many times a week.

---

# Type 3 — Git Flow

## Unique rule
There are **two permanent branches**:

| Branch | Meaning |
|--------|---------|
| `develop` | next version being built |
| `main` | versions already given to customers (`v1.3.0`) |

Features **never** merge straight to `main`.

```
feature/discount-coupon → develop → release/1.3.0 → main (tag v1.3.0)
```

---

# Type 3 — Coupon example (the unique part)

Sara branches from **`develop`**, not from `main`:

```bash
git checkout develop
git checkout -b feature/discount-coupon
git commit -m "Add discount coupon"
# PR → develop   ← NOT to main
```

Coupon is now in `develop` only.  
**Customers still have old shop** until a release.

Later:

```bash
git checkout develop
git checkout -b release/1.3.0
git checkout main
git merge release/1.3.0
git tag v1.3.0
```

**Customers see it:** on release day, as version `v1.3.0`.

---

# Type 3 — Hotfix (only Git Flow needs this)

Production is broken. `develop` already has unfinished features.  
Fix from **`main`**, not from `develop`.

```bash
git checkout main
git checkout -b hotfix/1.3.1
# fix, then merge to main AND develop
```

**Unique to Git Flow:** `hotfix/*` because `main` and `develop` are different.

---

# Type 3 — Do not confuse with GitHub Flow

| Question | GitHub Flow | Git Flow |
|---------|-------------|----------|
| Where does the feature PR go? | `main` | **`develop`** |
| Do we have `develop`? | No | **Yes** |
| When do users see it? | Right after merge | **On release day** |
| Version tags like `v1.3.0`? | Optional | **Normal** |

If you have a `develop` branch, you are **not** using GitHub Flow.

---

# Type 4 — GitLab Flow

## Unique rule
Branches are named after **environments**. You **promote** by merging.

```
feature/discount-coupon
        ↓ PR
      main          ← developers
        ↓ merge
     staging        ← QA tests here
        ↓ merge
    production      ← customers
```

The unique idea is **promotion**, not “feature branches” (everyone uses those).

---

# Type 4 — Coupon example

```bash
# 1) build
git checkout main
git checkout -b feature/discount-coupon
# PR into main

# 2) send to QA
git checkout staging
git merge main

# 3) send to customers
git checkout production
git merge staging
```

**Customers see it:** only after the last merge into `production`.

---

# Type 4 vs Type 3 (this is the confusing pair)

| | Git Flow | GitLab Flow |
|--|----------|-------------|
| Extra long branches | `develop` + `release/*` | **`staging` + `production`** |
| Branch names mean | work vs released **version** | **which environment** |
| How users get the feature | tag a release (`v1.3.0`) | merge into `production` |
| Typical product | mobile app / versioned software | website with staging URL |

Memory trick:

- Git Flow = **version** (`v1.3.0`)
- GitLab Flow = **environment** (`staging` / `production`)

---

# Type 5 — Trunk-Based Development

## Unique rule
You merge to `main` **before the coupon is finished**.  
Users do not see it until a **feature flag** is ON.

```
main  (trunk)
 └── small PR 1: coupon code, NEW_COUPON=false
 └── small PR 2: more coupon code, still false
 later: NEW_COUPON=true   ← users see it (no big merge)
```

---

# Type 5 — Coupon example (the flag)

```javascript
if (process.env.NEW_COUPON === "true") {
  showCoupon()     // new
} else {
  showOldCheckout()
}
```

Day 1: merge to `main`, production has `NEW_COUPON=false` → **hidden**  
Day 4: change config to `true` → **customers see coupon**

No `develop`. No waiting for a full finished feature branch.

---

# Type 5 vs Type 2 (the only difference)

Both merge into `main` often.

| | GitHub Flow | Trunk-Based |
|--|-------------|-------------|
| Merge unfinished coupon to `main`? | **No** (finish first) | **Yes** |
| How is unfinished work hidden? | Keep it on the feature branch | **Feature flag** |
| Branch lifetime | days | **hours / 1–2 days** |

**GitHub Flow:** `main` has only finished features.  
**Trunk-Based:** `main` can have unfinished features, **switched off**.

---

# Type 6 — Forking

## Unique rule
This is about **who owns the copy**, not about `develop` vs `main`.

```
company/shop     ← original (you cannot push)
sara/shop        ← Sara's fork (she pushes here)
     └── PR back to company/shop
```

You **can** combine forking with GitHub Flow or Git Flow.  
Forking answers: “Do I clone the company repo, or **my** copy?”

---

# Type 6 — Coupon example

```bash
git clone https://github.com/sara/shop.git
git remote add upstream https://github.com/company/shop.git
git checkout -b feature/discount-coupon
git push origin feature/discount-coupon
# PR: sara/shop → company/shop
```

`git remote add upstream` = save the original repo name.  
Later: `git fetch upstream` to update your fork.

**Customers see it:** after maintainers merge the PR (using whatever flow they use).

---

# Type 6 — Do not confuse with Feature Branch

| | Feature Branch | Forking |
|--|----------------|---------|
| Where do you push? | `company/shop` | **`your-name/shop`** |
| Who uses it? | company team | **outsiders / open source** |

Same “one branch + PR” idea.  
Different **remote** (`origin` = your fork).

---

# One table to differentiate all 6

**Question: the coupon is done. What happens?**

| Type | Merge into | Customers see coupon when |
|------|------------|---------------------------|
| Feature Branch | `main` | whenever you deploy |
| GitHub Flow | `main` | **immediately after merge** |
| Git Flow | **`develop` first** | **release / tag** (`v1.3.0`) |
| GitLab Flow | `main`, then **`staging`**, then **`production`** | merge to `production` |
| Trunk-Based | `main` (maybe unfinished) | **flag = true** |
| Forking | PR from **your fork** | after original repo accepts PR |

---

# “Which type is this?” quiz

1. We have `develop` and we tag `v2.1.0` on `main`.
2. We merge to `main` and the website updates in 5 minutes.
3. QA tests `staging`, then we merge `staging` → `production`.
4. Code is on `main` but users don’t see it until a switch is ON.
5. I cannot push to the company repo; I opened a PR from my copy.
6. We use feature PRs to `main`, and deploy when the manager says so.

---

# Quiz answers

1. **Git Flow** (`develop` + version tag)
2. **GitHub Flow** (merge = deploy now)
3. **GitLab Flow** (environment branches)
4. **Trunk-Based** (feature flag)
5. **Forking**
6. **Feature Branch** (PR to `main`, deploy not forced)

---

# Decision: pick only with this question

**How do customers get new code?**

```
Immediately after merge to main?     → GitHub Flow
On a planned version day (v1.3.0)?  → Git Flow
After QA on a staging branch?       → GitLab Flow
After turning a flag ON?            → Trunk-Based
Contributor has no write access?    → Forking
Just PRs, no extra rules yet?       → Feature Branch
```

Start with **Feature Branch**. Add extra rules only when you need them.

---

# Cheat sheet (pin this)

```
1 Feature Branch : branch + PR                    (base habit)
2 GitHub Flow    : that + deploy main NOW
3 Git Flow       : develop ≠ main (versions)
4 GitLab Flow    : main → staging → production
5 Trunk-Based    : merge early + FLAG
6 Forking        : your copy, not company clone
```

**Git strategy** and **branching strategy** are the same topic.

---

# Thank You

Remember: all types use branches and PRs.  
Differentiate them by **where you merge** and **when users see the change**.
