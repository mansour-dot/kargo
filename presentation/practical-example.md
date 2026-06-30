---
marp: true
theme: default
paginate: true
header: 'Kargo & Argo CD — Practical Examples'
footer: 'Simple guide for everyone'
style: |
  section { font-size: 26px; }
  h1 { color: #2563eb; }
  h2 { color: #1e40af; }
  code { background: #f1f5f9; font-size: 22px; }
  table { font-size: 22px; }
---

# Kargo & Argo CD
## Practical Examples (Beginner Friendly)

**No expert knowledge required — just follow the examples**

---

# Who Is This For?

This guide is for you if:

- You heard about **Argo CD** or **Kargo** but feel lost
- You deploy apps to Kubernetes and want a **safer, clearer process**
- You want **real examples**, not heavy theory

**You do NOT need to be a Kubernetes expert to understand the ideas here.**

---

# A Simple Story First

Imagine you built a **web shop** app.

You have 3 places to run it:

| Environment | Who uses it | Risk if broken |
|-------------|-------------|----------------|
| **Test** | Developers | Low |
| **Staging** | QA team | Medium |
| **Production** | Real customers | High |

**Question:** When version `v2.0` works in Test, how do you safely move it to Production?

That is exactly what **Kargo + Argo CD** help you do.

---

# The Problem (Without These Tools)

**Old manual way:**

1. Developer changes image tag in a YAML file for Test ✅
2. Test passes ✅
3. Developer copies the same change to Staging YAML 😰
4. Someone forgets one file 😱
5. Production gets wrong version 💥

**Result:** Human errors, no clear history, hard rollbacks.

---

# The Solution (Simple View)

```
         KARGO                          ARGO CD
    "Move the change"              "Deploy the change"
         │                                │
         ▼                                ▼
   Test → Staging → Prod            Cluster matches Git
```

- **Kargo** = traffic controller between environments
- **Argo CD** = robot that keeps Kubernetes equal to Git

---

# Real-Life Analogy

Think of shipping packages:

| Concept | Real world | Tool |
|---------|------------|------|
| Product in factory | New app version (Docker image) | Warehouse |
| Box with items | Image + config together | Freight |
| City checkpoint | Test / Staging / Prod | Stage |
| Truck delivery | Deploy to cluster | Argo CD |

**Kargo packs and routes the box. Argo CD delivers it.**

---

# What Is Argo CD? (1 minute)

**Argo CD watches Git and your cluster.**

- Git says: "Run nginx version 1.29"
- Cluster runs: nginx version 1.28
- Argo CD says: **OutOfSync** → then syncs cluster to Git

**One job only:** make cluster match Git.

---

# Argo CD Example: Before Sync

**Git file (`deployment.yaml`):**
```yaml
image: my-shop:2.0
```

**Cluster currently running:**
```yaml
image: my-shop:1.9
```

**Argo CD dashboard shows:**
- Status: `OutOfSync` ⚠️
- Health: maybe still `Healthy` (old version works)

After sync → cluster runs `my-shop:2.0`.

---

# Argo CD Example: Application Resource

This tells Argo CD **what to deploy** and **where**:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-shop-test
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/my-team/shop-config.git
    targetRevision: stage/test      # branch name
    path: .                           # folder in repo
  destination:
    server: https://kubernetes.default.svc
    namespace: shop-test              # K8s namespace
```

**Plain English:**
- Read config from Git branch `stage/test`
- Deploy it into namespace `shop-test`

---

# What Is Kargo? (1 minute)

**Kargo moves approved changes between environments.**

Example flow:
1. New Docker image appears
2. Kargo creates a **Freight** (bundle)
3. You promote Freight to **Test**
4. After Test is healthy, promote to **Staging**
5. Then promote to **Production**

**Kargo does not replace Argo CD. It works with it.**

---

# Kargo Words Made Easy

| Kargo word | Simple meaning | Example |
|------------|----------------|---------|
| **Project** | Folder for one app pipeline | `shop-project` |
| **Warehouse** | Watcher for new versions | watches `my-shop` image |
| **Freight** | One releasable package | image `2.0` + config commit |
| **Stage** | One environment step | `test`, `staging`, `prod` |
| **Promotion** | Action of moving Freight | test → staging |

---

# Example 1: One App, Three Environments

**App:** `my-shop`

**Git repo structure:**
```
shop-config/
├── base/                 # common files
└── stages/
    ├── test/
    ├── staging/
    └── prod/
```

**Argo CD apps:**
- `my-shop-test` → deploys `stages/test`
- `my-shop-staging` → deploys `stages/staging`
- `my-shop-prod` → deploys `stages/prod`

---

# Example 1: What Happens Step by Step

**Step 1:** CI builds image `my-shop:2.0`

**Step 2:** Kargo Warehouse sees new image

**Step 3:** Kargo creates Freight `#42` containing `my-shop:2.0`

**Step 4:** You promote `#42` to Test

**Step 5:** Kargo updates Git branch `stage/test`

**Step 6:** Argo CD syncs Test cluster

**Step 7:** Test becomes healthy ✅

**Step 8:** Now you can promote `#42` to Staging

---

# Example 2: Warehouse (Watch New Images)

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: Warehouse
metadata:
  name: my-shop
  namespace: shop-project
spec:
  subscriptions:
  - image:
      repoURL: ghcr.io/my-team/my-shop
      constraint: ^2.0.0
```

**What this means:**
- Watch `ghcr.io/my-team/my-shop`
- Accept versions like `2.0.1`, `2.0.2`, etc.
- When a new one appears, create new Freight

---

# Example 3: Stage for Test Environment

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: Stage
metadata:
  name: test
  namespace: shop-project
spec:
  requestedFreight:
  - origin:
      kind: Warehouse
      name: my-shop
    sources:
      direct: true
  promotionTemplate:
    spec:
      steps:
      - task:
          name: promote-shop
```

**Plain English:**
- `test` stage accepts Freight directly from Warehouse
- When promoted, run task `promote-shop`

---

# Example 4: Stage for Production (Safer)

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: Stage
metadata:
  name: prod
  namespace: shop-project
spec:
  requestedFreight:
  - origin:
      kind: Warehouse
      name: my-shop
    sources:
      stages:
      - staging
```

**Important rule:**
- `prod` does **NOT** take Freight directly from Warehouse
- It only accepts Freight already verified in `staging`

This prevents skipping safety checks.

---

# Example 5: Promotion Task (The Real Work)

This is what Kargo runs during promotion:

```yaml
steps:
- uses: git-clone
- uses: kustomize-set-image
- uses: kustomize-build
- uses: git-commit
- uses: git-push
- uses: argocd-update
```

**In simple words:**
1. Get files from Git
2. Change image version
3. Build final YAML
4. Commit + push to Git
5. Tell Argo CD: "Please sync now"

---

# Example 5: Full Mini Task

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: PromotionTask
metadata:
  name: promote-shop
  namespace: shop-project
spec:
  vars:
  - name: gitRepo
    value: https://github.com/my-team/shop-config.git
  - name: imageRepo
    value: ghcr.io/my-team/my-shop
  steps:
  - uses: git-clone
    config:
      repoURL: ${{ vars.gitRepo }}
      checkout:
      - branch: main
        path: ./src
      - branch: stage/${{ ctx.stage }}
        create: true
        path: ./out
```

`ctx.stage` automatically becomes `test`, `staging`, or `prod`.

---

# Example 5: Update Image + Deploy

```yaml
  - uses: kustomize-set-image
    config:
      path: ./src/base
      images:
      - image: ${{ vars.imageRepo }}
        tag: ${{ imageFrom(vars.imageRepo).Tag }}

  - uses: kustomize-build
    config:
      path: ./src/stages/${{ ctx.stage }}
      outPath: ./out

  - uses: git-commit
    config:
      path: ./out
      message: "Promote ${{ vars.imageRepo }} to ${{ ctx.stage }}"

  - uses: git-push
    config:
      path: ./out
```

After push, Git has the new version for that environment.

---

# Example 5: Tell Argo CD to Sync

```yaml
  - uses: argocd-update
    config:
      apps:
      - name: my-shop-${{ ctx.stage }}
        sources:
        - repoURL: ${{ vars.gitRepo }}
          desiredRevision: ${{ task.outputs.commit.commit }}
```

**Meaning:**
- Find Argo CD app `my-shop-test` (or staging/prod)
- Sync to the commit Kargo just pushed

---

# Example 6: Allow Kargo to Control Argo CD App

Add this annotation to each Argo CD Application:

```yaml
metadata:
  annotations:
    kargo.akuity.io/authorized-stage: shop-project:test
```

For all environments with ApplicationSet:

```yaml
annotations:
  kargo.akuity.io/authorized-stage: shop-project:{{stage}}
```

Without this, Kargo is blocked from updating that app.

---

# Example 7: ApplicationSet for 3 Environments

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: my-shop
  namespace: argocd
spec:
  generators:
  - list:
      elements:
      - stage: test
      - stage: staging
      - stage: prod
  template:
    metadata:
      name: my-shop-{{stage}}
      annotations:
        kargo.akuity.io/authorized-stage: shop-project:{{stage}}
    spec:
      source:
        repoURL: https://github.com/my-team/shop-config.git
        targetRevision: stage/{{stage}}
        path: .
      destination:
        namespace: shop-{{stage}}
```

One template creates 3 apps automatically.

---

# Example 8: What You See in Kargo UI

When Freight `#42` is promoted:

1. Drag Freight box to `test` (or click Promote)
2. Promotion starts (Running)
3. Steps turn green one by one
4. Test stage gets heart ❤️ (healthy)
5. Freight color matches `test`
6. `staging` unlocks for same Freight

If one step fails, promotion stops and shows error.

---

# Example 9: Failed Promotion (Real Scenario)

**Scenario:** Git push fails (bad token)

- `git-clone` ✅
- `kustomize-set-image` ✅
- `git-commit` ✅
- `git-push` ❌ fails
- `argocd-update` not run

**Result:**
- Test still on old version
- Promotion status = Failed
- You fix token and promote again

This is safer than half-deploying silently.

---

# Example 10: Rollback (Simple)

Because everything is in Git:

1. Open Git history for `stage/prod`
2. Find previous good commit
3. Revert or promote older Freight again

Argo CD syncs back to previous version.

No manual `kubectl` surgery needed.

---

# Example 11: Pull Request Gate (Production)

For production, many teams require approval:

```yaml
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

**Flow:**
1. Kargo opens PR
2. Team reviews
3. After merge, Argo CD deploys

---

# Example 12: Day in the Life

**09:00** — CI publishes `my-shop:2.0.3`
**09:02** — Kargo creates Freight
**09:05** — Developer promotes to Test
**09:10** — QA validates Test
**11:00** — Release manager promotes to Staging
**14:00** — Staging tests pass
**16:00** — Promote to Production
**16:05** — Argo CD deploys prod
**16:10** — Monitoring confirms success

Everyone can see this timeline in Kargo.

---

# Common Mistakes (And Fixes)

| Mistake | Fix |
|---------|-----|
| Promote directly to prod | Make prod subscribe to staging |
| Argo CD app not syncing | Check `authorized-stage` annotation |
| Wrong image in one env | Promote same Freight, don't edit manually |
| Git push denied | Add repo credentials secret in Kargo project |
| Stage always unhealthy | Check Argo CD app health/sync first |

---

# Quick Comparison Table

| Question | Use this |
|----------|----------|
| "Deploy this Git commit to cluster" | **Argo CD** |
| "Move version from test to prod safely" | **Kargo** |
| "Build app binary/image" | **CI (GitHub Actions, etc.)** |
| "Store desired deployment state" | **Git** |

**Best combo:** CI + Kargo + Argo CD

---

# Try It Yourself (Mini Lab)

1. Install local demo:
```bash
curl -L https://raw.githubusercontent.com/akuity/kargo/main/hack/quickstart/install.sh | sh
```

2. Open dashboards:
- Argo CD: `http://localhost:31080` (admin/admin)
- Kargo: `http://localhost:31081` (admin)

3. Fork `akuity/kargo-demo`
4. Promote Freight: test → uat → prod

You will understand the full flow in ~30 minutes.

---

# Cheat Sheet

```
Warehouse  -> finds new versions
Freight    -> package to release
Stage      -> environment checkpoint
Promotion  -> move Freight to Stage
Git        -> source of truth
Argo CD    -> deploy to Kubernetes
```

**Remember:**
- Kargo = promotion
- Argo CD = deployment

---

# Final Summary

If you remember only 3 things:

1. **Git is the truth** (not manual cluster edits)
2. **Kargo promotes between environments** (with rules)
3. **Argo CD deploys each environment** (from Git)

With these tools, releases become:
- More predictable
- Easier to audit
- Safer for non-experts on the team

---

# Next Step

- Beginner path: read this file + run quickstart demo
- Advanced path: read `presentation/kargo-and-argocd.md`

**You are ready to explain Kargo and Argo CD to your team.**

---

# Thank You

Questions to practice:
1. What is Freight?
2. Why can't prod take Freight directly from Warehouse?
3. Which tool actually deploys to Kubernetes?

**Answers:**
1. A releasable bundle (image + config)
2. To enforce test/staging checks first
3. Argo CD
