---
marp: true
theme: default
paginate: true
header: 'Git Strategy for GitOps'
footer: 'Practical guide with examples'
style: |
  section { font-size: 26px; }
  h1 { color: #2563eb; }
  h2 { color: #1e40af; }
  code { background: #f1f5f9; font-size: 21px; }
  table { font-size: 21px; }
---

# Git Strategy for GitOps
## Detailed + Simple Guide with Practical Examples

**How to organize Git when using Kargo and Argo CD**

---

# Who Is This For?

You should read this if you ask:

- "Which branch should Test/Prod use?"
- "Should app code and deployment config live in one repo?"
- "How do I avoid breaking GitOps with wrong Git layout?"
- "What is the safest Git strategy for promotions?"

**No Git expert level required.**

---

# Big Idea in One Slide

In GitOps:

> **Git is the contract** between humans and Kubernetes.

Your Git strategy answers 3 questions:

1. **Where** do we store deployment config?
2. **How** do we separate test/staging/prod?
3. **Who** can change what, and when?

---

# Two Different Repos (Very Important)

Argo CD best practice: split repositories.

| Repo | Contains | Changed by |
|------|----------|------------|
| **App source repo** | Java/Go/Node code, Dockerfile, unit tests | Developers |
| **GitOps config repo** | Kubernetes YAML, Helm/Kustomize files | CI + Kargo + platform team |

**Why split?**
- Cleaner audit history
- Avoid CI infinite loops
- Different access permissions (dev vs prod)

---

# Example: Bad vs Good Repo Design

### ❌ Bad (everything mixed)
```
my-app/
├── src/main.go
├── Dockerfile
└── k8s/prod-deployment.yaml
```

### ✅ Good (separated)
```
my-app/                  # source repo
└── src/main.go

my-app-config/           # gitops repo
├── base/
└── stages/
```

Argo CD reads **only** `my-app-config`.

---

# What Kargo Writes to Git

During promotion, Kargo usually:

1. Reads config from Git
2. Updates image tag / values
3. Renders final manifests
4. Commits result back to Git
5. Argo CD syncs that commit

So your Git strategy must support **read path** and **write path**.

---

# 3 Main Git Storage Strategies

| Strategy | Simple description | Best for |
|----------|-------------------|----------|
| **A. Stage branches** | `stage/test`, `stage/prod` | Most Kargo users (recommended) |
| **B. Single branch + folders** | `src/` input, `builds/` output on `main` | Teams that hate many branches |
| **C. Separate output repo** | Input repo + deploy repo | Strong compliance separation |

We will show practical examples for all 3.

---

# Strategy A: Stage-Specific Branches (Recommended)

**Idea:** each environment has its own branch storage.

```
main              -> shared base config (input)
stage/test        -> rendered config for test
stage/staging     -> rendered config for staging
stage/prod        -> rendered config for prod
```

Argo CD app for test watches `stage/test`.
Argo CD app for prod watches `stage/prod`.

---

# Strategy A: Why Kargo Recommends It

From Kargo docs:

- Stage branches are **storage**, not GitFlow merges
- You do **not** need to merge `stage/prod` back to `main`
- Think of each branch like a separate folder/bucket

**Benefit:** clear per-environment history and easy rollback.

---

# Strategy A: Practical Repo Layout (Kustomize)

```
shop-config/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── stages/
    ├── test/
    │   └── kustomization.yaml
    ├── staging/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

- `main` branch keeps this structure
- Kargo writes rendered output to `stage/<env>` branches

---

# Strategy A: Example File (`base/deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shop
spec:
  replicas: 2
  template:
    spec:
      containers:
      - name: shop
        image: ghcr.io/acme/shop:1.0.0
```

This is shared base config used by all environments.

---

# Strategy A: Example Overlay (`stages/test/kustomization.yaml`)

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../base
patches:
- patch: |-
    - op: replace
      path: /spec/replicas
      value: 1
```

Test uses 1 replica; prod can use more.

---

# Strategy A: Promotion Flow Example

**Freight contains image `shop:2.3.0`**

1. Promote to `test`
2. Kargo updates image in base
3. Kargo builds `stages/test`
4. Kargo commits to branch `stage/test`
5. Argo CD app `shop-test` syncs `stage/test`

Later, same Freight can be promoted to `stage/prod`.

---

# Strategy A: Argo CD Mapping Example

```yaml
# test app
source:
  repoURL: https://github.com/acme/shop-config.git
  targetRevision: stage/test
  path: .

# prod app
source:
  repoURL: https://github.com/acme/shop-config.git
  targetRevision: stage/prod
  path: .
```

Same repo, different branches per environment.

---

# Strategy B: Single Branch (`main`) + Folders

For teams that want one branch only:

```
shop-config/
├── src/                 # INPUT (edited by humans/tools)
│   ├── base/
│   └── stages/
└── builds/              # OUTPUT (written by Kargo)
    ├── test/
    ├── staging/
    └── prod/
```

Argo CD watches `builds/test`, `builds/prod`, etc. on `main`.

---

# Strategy B: Why `src` and `builds` Must Be Separate

If Kargo writes back into `src`, you risk a **feedback loop**:

1. Kargo writes output
2. Warehouse sees "new commit"
3. New Freight created automatically
4. Unexpected promotions

**Rule:** output path must be different from monitored input path.

---

# Strategy B: Warehouse Path Filter Example

```yaml
spec:
  subscriptions:
  - git:
      repoURL: https://github.com/acme/shop-config.git
      branch: main
      includePaths:
      - src/**
```

This means:
- changes in `src/**` can create Freight
- changes in `builds/**` are ignored

---

# Strategy B: Argo CD Example

```yaml
source:
  repoURL: https://github.com/acme/shop-config.git
  targetRevision: main
  path: builds/prod
```

Prod app always reads rendered manifests from `builds/prod`.

---

# Strategy C: Separate Output Repository

Some teams use:

- **Config source repo** (`shop-config-src`)
- **Deployment repo** (`shop-config-deploy`)

Kargo reads from source repo, writes rendered manifests to deploy repo.

**Use when:** strict separation of "design-time config" vs "runtime deploy artifacts".

---

# Strategy C: Practical Example

```
shop-config-src/                 shop-config-deploy/
├── base/                        ├── test/
└── stages/                      ├── staging/
                                 └── prod/
```

Promotion step:
- clone from `shop-config-src`
- push rendered files to `shop-config-deploy` branch/folder

Argo CD watches only `shop-config-deploy`.

---

# Git Strategy vs GitFlow (Don't Confuse Them)

| Topic | GitFlow | Kargo stage branches |
|-------|---------|----------------------|
| Purpose | Feature/release workflow for app code | Environment storage for deploy config |
| Merge direction | feature → develop → main | Not required between stage branches |
| Who uses it | App developers | Platform/CD pipeline |
| Main branch role | Production code line | Shared base config input |

**Stage branches are not GitFlow.**

---

# Branching Models for App Code (Separate Topic)

For application source code, common options:

| Model | Summary | Good when |
|-------|---------|-----------|
| **Trunk-based** | short-lived branches, frequent merge to `main` | fast CI/CD teams |
| **GitFlow** | `develop`, `release`, `hotfix` branches | scheduled releases |
| **GitHub Flow** | feature branch + PR to `main` | simple web apps |

**Important:** this is for app code repo, not necessarily GitOps config repo.

---

# Practical Example: End-to-End Git Flow

1. Developer merges feature to app repo `main`
2. CI builds image `shop:2.4.0` and pushes to registry
3. Kargo Warehouse detects new image
4. Freight `#108` created
5. Promote `#108` to test → commit on `stage/test`
6. Argo CD deploys test
7. Promote to prod → commit on `stage/prod`
8. Argo CD deploys prod

Git history shows exact promoted versions per environment.

---

# Commit Message Strategy (Practical)

Use clear, machine-friendly messages:

```text
promote(shop): ghcr.io/acme/shop:2.4.0 -> test

- freight: shop-108
- promoted-by: jane.doe
- source-freight-from: warehouse
```

Benefits:
- easy search in Git
- easier incident investigation
- better audit for compliance

---

# Pull Request Strategy (Production Safety)

For production, many teams require PR approval:

```yaml
steps:
- uses: git-push
  as: push
  config:
    generateTargetBranch: true
- uses: git-open-pr
  config:
    sourceBranch: ${{ outputs.push.branch }}
    targetBranch: stage/prod
- uses: git-wait-for-pr
- uses: argocd-update
```

**Meaning:** prod Git change is reviewed before deploy.

---

# Example: PR Promotion Timeline

1. Kargo opens PR: `promo/prod-2026-06-30-001` → `stage/prod`
2. SRE reviews rendered YAML diff
3. Approver merges PR
4. Kargo continues and triggers Argo CD sync
5. Production deployment starts

This gives human gate without breaking GitOps.

---

# Monorepo Git Strategy (Multiple Apps)

When one GitOps repo has many apps:

```
platform-config/
├── shop/
│   ├── base/
│   └── stages/
├── payments/
│   ├── base/
│   └── stages/
└── auth/
    ├── base/
    └── stages/
```

Use path filters per Warehouse:

- `shop` warehouse watches `shop/**`
- `payments` warehouse watches `payments/**`

---

# Monorepo Warehouse Filter Example

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: Warehouse
metadata:
  name: shop
spec:
  subscriptions:
  - git:
      repoURL: https://github.com/acme/platform-config.git
      branch: main
      includePaths:
      - shop/**
  - image:
      repoURL: ghcr.io/acme/shop
```

Prevents unrelated app changes from triggering shop Freight.

---

# Helm Git Layout Example

```
shop-config/
├── Chart.yaml
├── values.yaml              # base defaults
├── templates/
│   ├── deployment.yaml
│   └── service.yaml
└── stages/
    ├── test/values.yaml
    ├── staging/values.yaml
    └── prod/values.yaml
```

Promotion can:
- update image tag in base or stage values
- render chart with `helm-template`
- commit rendered YAML to stage branch or `builds/prod`

---

# Rendered Manifests vs Raw Helm/Kustomize

Kargo recommends **rendered plain YAML** for promotion output.

Why?
- PR diffs are obvious (`image: 2.3.0` → `2.4.0`)
- no mental rendering needed during review
- Argo CD does less work at sync time

**Example output file:** `builds/prod/deployment.yaml` (fully rendered)

---

# Tag vs Branch vs Commit (Argo CD Tracking)

Argo CD `targetRevision` can be:

| Value | Example | When to use |
|-------|---------|-------------|
| Branch | `stage/prod` | moving environment head (common with Kargo) |
| Tag | `v2.4.0` | immutable release marker |
| Commit SHA | `a1b2c3d` | exact pinned deployment |

In Kargo pipelines, stage branches are most common.

---

# Practical Example: Rollback with Git

**Incident:** prod `shop:2.4.0` has bug.

### Option 1: Promote older Freight again
- find previous healthy Freight
- promote to prod

### Option 2: Git revert on `stage/prod`
```bash
git checkout stage/prod
git revert <bad-commit>
git push
```
Argo CD syncs previous good state.

Git makes rollback auditable.

---

# Access Control Git Strategy

Typical permissions:

| Repo / Branch | Developers | QA | SRE/Platform |
|---------------|-----------|----|--------------|
| app source `main` | write | read | read |
| config `main` (`src`) | read/write (limited) | read | write |
| `stage/test` | indirect via Kargo | read | write |
| `stage/prod` | no direct write | read | write/approve PR |

Goal: developers promote via Kargo, not direct prod Git edits.

---

# Avoid These Common Git Mistakes

| Mistake | Result | Fix |
|---------|--------|-----|
| App code + prod YAML in same repo | noisy history, CI loops | split repos |
| Kargo writes to monitored input path | feedback loop | separate `src` and `builds` |
| No path filters in monorepo | unrelated promotions | `includePaths` per Warehouse |
| Floating remote base (`ref=HEAD`) | surprise manifest changes | pin tag/commit SHA |
| Manual prod YAML edits | drift from process | promote via Kargo only |

---

# Feedback Loop Example (What Not To Do)

### ❌ Wrong
- Warehouse watches `main`
- Kargo writes rendered output to `main` (same paths)

### ✅ Correct
- Warehouse watches `main:src/**`
- Kargo writes to `main:builds/prod/**`
- OR use `stage/prod` branch

Always define **input scope** and **output location**.

---

# Decision Guide: Which Strategy Should I Choose?

Choose **Stage branches (A)** if:
- you use Kargo promotions across environments
- you want clearest per-env history

Choose **Single branch folders (B)** if:
- your team strongly prefers one branch
- you can enforce path filters correctly

Choose **Separate output repo (C)** if:
- compliance needs strict artifact separation

---

# Starter Template (Recommended for Beginners)

```
my-app-config/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── stages/
    ├── test/kustomization.yaml
    ├── staging/kustomization.yaml
    └── prod/kustomization.yaml
```

Branches:
- `main` (source layout)
- `stage/test`, `stage/staging`, `stage/prod` (rendered output)

This works directly with Kargo quickstart patterns.

---

# 1-Week Adoption Plan

**Day 1:** create config repo + base/stages layout
**Day 2:** connect Argo CD apps per environment branch
**Day 3:** configure Kargo Warehouse + Stages
**Day 4:** test promotion to `stage/test`
**Day 5:** add PR gate for prod
**Day 6:** document commit/rollback process
**Day 7:** train team with examples and dry run

---

# Cheat Sheet

```
Source repo      = app code
Config repo      = deployment desired state
main             = shared base config input
stage/<env>      = environment-specific rendered output
Kargo            = writes promotion commits
Argo CD          = reads Git and deploys
Path filters     = prevent feedback loops
PR gate          = human approval for prod
```

---

# Practice Questions

1. Why separate app repo and config repo?
2. What is a feedback loop in GitOps?
3. Why are stage branches not GitFlow?
4. Where should Kargo write rendered manifests?
5. How do you rollback production safely?

---

# Answer Key

1. Cleaner history, safer permissions, no CI loops
2. Kargo output triggers Warehouse again unintentionally
3. Stage branches are environment storage, not feature flow
4. `stage/<env>`, or `builds/<env>`, or separate deploy repo
5. Promote old Freight or revert commit on prod branch

---

# Final Summary

A good Git strategy for Kargo + Argo CD is:

- **Simple** enough for the whole team
- **Explicit** about input vs output paths
- **Safe** with promotion gates and auditable commits
- **Practical** with real per-environment branches or folders

Git is not just storage — it is your deployment timeline.

---

# Thank You

Related files in this repo:
- `presentation/practical-example.md`
- `presentation/kargo-and-argocd.md`

Official references:
- https://docs.kargo.io/user-guide/patterns
- https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/
