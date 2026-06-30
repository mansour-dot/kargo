---
marp: true
theme: default
paginate: true
header: 'Kargo & Argo CD'
footer: 'GitOps Continuous Delivery for Kubernetes'
style: |
  section { font-size: 28px; }
  h1 { color: #2563eb; }
  h2 { color: #1e40af; }
  code { background: #f1f5f9; }
---

# Kargo & Argo CD
## GitOps Continuous Delivery for Kubernetes

**A complete guide to deployment and multi-stage promotion**

---

# Agenda

1. GitOps fundamentals
2. Argo CD — what it is and how it works
3. Kargo — what it is and why it exists
4. How Kargo and Argo CD work together
5. Core Kargo concepts
6. Promotion pipelines in practice
7. Integration details
8. Common patterns and best practices
9. When to use which tool
10. Getting started

---

# What Is GitOps?

**GitOps** is an operational model where:

- **Git is the single source of truth** for desired application and infrastructure state
- Changes are made by **committing to Git**, not by running imperative commands
- A controller **continuously reconciles** live cluster state with Git
- Every change is **versioned, auditable, and reversible**

> "Application definitions, configurations, and environments should be declarative and version controlled. Application deployment and lifecycle management should be automated, auditable, and easy to understand."
> — Argo CD project

---

# The GitOps Gap

GitOps tools like Argo CD excel at **deployment**:

```
Git (desired state)  →  Controller  →  Kubernetes (live state)
```

But they do **not** natively answer:

- How do I move a validated change from **test → staging → production**?
- How do I ensure **image + config** promote together as a unit?
- How do I enforce **verification gates** between environments?
- How do I provide an **audit trail** of promotions?

**This is where Kargo fits.**

---

# Part 1: Argo CD

---

# What Is Argo CD?

**Argo CD** is a declarative, GitOps continuous delivery tool for Kubernetes.

| Attribute | Detail |
|-----------|--------|
| **Type** | Kubernetes-native controller |
| **License** | Apache 2.0 |
| **Project** | Part of the [Argo Project](https://argoproj.github.io/) (CNCF incubating) |
| **Purpose** | Sync Kubernetes clusters to desired state stored in Git |

Argo CD automates deployment of application states to target environments and detects configuration drift.

---

# Argo CD Core Concepts

| Concept | Definition |
|---------|------------|
| **Application** | A group of Kubernetes resources defined by a manifest (CRD) |
| **Target state** | Desired state expressed in a Git repository |
| **Live state** | What is actually running in the cluster |
| **Sync status** | Whether live state matches target state (`Synced` / `OutOfSync`) |
| **Sync** | Process of applying Git state to the cluster |
| **Refresh** | Compare latest Git code with live state |
| **Health** | Whether the application is running correctly |

---

# Argo CD Architecture

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Web UI /   │────▶│   API Server     │────▶│ Application         │
│  CLI / CI   │     │  (gRPC/REST)     │     │ Controller          │
└─────────────┘     └──────────────────┘     └──────────┬──────────┘
                              │                           │
                              ▼                           ▼
                    ┌──────────────────┐         ┌─────────────────────┐
                    │ Repository       │         │ Kubernetes Cluster(s)│
                    │ Server           │         │ (live state)         │
                    │ (Git cache,      │         └─────────────────────┘
                    │  manifest gen)   │
                    └──────────────────┘
```

---

# Argo CD Components

### API Server
- Exposes REST/gRPC API for UI, CLI, and CI/CD
- Manages applications, credentials, RBAC
- Handles Git webhooks and authentication

### Repository Server
- Maintains local cache of Git repositories
- Generates Kubernetes manifests from Helm, Kustomize, Jsonnet, or plain YAML

### Application Controller
- Continuously monitors applications
- Detects `OutOfSync` state and triggers sync
- Invokes lifecycle hooks (PreSync, Sync, PostSync)

---

# What Argo CD Supports

**Configuration management tools:**
- Kustomize
- Helm charts
- Jsonnet
- Plain YAML/JSON directories
- Custom config management plugins (CMPs)

**Deployment features:**
- Multi-cluster management
- Automated or manual sync
- Rollback to any Git commit
- Sync waves and hooks (blue/green, canary)
- Sync windows (time-based deployment controls)
- Drift detection and visualization
- SSO (OIDC, OAuth2, LDAP, SAML, GitHub, GitLab, etc.)

---

# Argo CD Application Model

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-test
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/gitops-repo.git
    targetRevision: stage/test    # branch, tag, or commit
    path: stages/test
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app-test
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
```

Argo CD watches `source` and reconciles `destination` to match.

---

# Argo CD ApplicationSet

**ApplicationSet** generates multiple `Application` resources from templates.

Useful for:
- One application per environment (test, uat, prod)
- One application per cluster
- One application per tenant

```yaml
generators:
- list:
    elements:
    - stage: test
    - stage: uat
    - stage: prod
template:
  metadata:
    name: my-app-{{stage}}
```

---

# Argo CD: What It Does Well

✅ Declarative deployment from Git
✅ Multi-cluster, multi-environment sync
✅ Rich UI with real-time application status
✅ Health analysis and drift detection
✅ Rollback to any previous Git state
✅ Integration with CI via webhooks and API
✅ Strong RBAC and multi-tenancy (Projects)

---

# Argo CD: What It Does NOT Do

❌ **Promotion** — no concept of moving validated state between environments
❌ **Artifact discovery** — does not watch container registries for new images
❌ **Bundling** — no native way to tie image + config + chart as one promotable unit
❌ **Pipeline orchestration** — no built-in test → uat → prod workflow
❌ **Verification gates** — no native "wait for tests to pass before next stage"

> Argo CD syncs clusters to Git. It does not orchestrate *how* Git changes flow between environments.

---

# Part 2: Kargo

---

# What Is Kargo?

**Kargo** is an open-source, GitOps-native **continuous promotion** platform for Kubernetes.

| Attribute | Detail |
|-----------|--------|
| **Created by** | Akuity (founders of the Argo Project, including Argo CD) |
| **Announced** | September 18, 2023 |
| **GA (v1.0)** | October 22, 2024 |
| **License** | Apache 2.0 |
| **Repository** | https://github.com/akuity/kargo |

> "Kargo is a continuous promotion orchestration layer that complements Argo CD for Kubernetes."

---

# Why Kargo Was Created

The creators of Argo CD recognized a gap:

| Argo CD handles | Kargo handles |
|---------------|---------------|
| **Deployment** — making cluster match Git | **Promotion** — moving changes between stages |
| Syncing one environment to one Git revision | Orchestrating test → staging → production |
| Detecting drift | Packaging artifacts as promotable units |
| Application health in one cluster | Pipeline health across the lifecycle |

**Promotions ≠ Deployments**

- **Promotion** = update the *desired state* of the next stage
- **Deployment** = make the cluster match that desired state (Argo CD's job)

---

# Kargo Design Principles

1. **GitOps-native** — promotions write to Git; Argo CD deploys from Git
2. **Unopinionated** — flexible promotion processes via composable steps
3. **Kubernetes-native** — all resources are Kubernetes CRDs
4. **Multi-tenant** — Projects map to namespaces with RBAC
5. **Complements existing tools** — works with Argo CD, Helm, Kustomize, OpenTofu

---

# Part 3: How They Work Together

---

# The Combined Workflow

```
┌──────────────┐    discovers     ┌──────────────┐    promotes     ┌──────────────┐
│  Warehouse   │ ──────────────▶  │   Freight    │ ──────────────▶ │    Stage     │
│ (image repo, │   new artifacts  │ (bundled     │   via promotion │  (test/uat/  │
│  git, helm)  │                  │  artifacts)  │   steps         │   prod)      │
└──────────────┘                  └──────────────┘                 └──────┬───────┘
                                                                          │
                              ┌───────────────────────────────────────────┘
                              │ git-commit, git-push, kustomize-set-image
                              ▼
                    ┌──────────────────┐    argocd-update    ┌──────────────────┐
                    │   Git Repo       │ ◀────────────────── │     Kargo        │
                    │ (desired state)  │                     │  (orchestrator)  │
                    └────────┬─────────┘                     └──────────────────┘
                             │
                             │ Argo CD sync
                             ▼
                    ┌──────────────────┐
                    │   Kubernetes     │
                    │   Cluster        │
                    └──────────────────┘
```

---

# Responsibility Split

| Layer | Tool | Responsibility |
|-------|------|----------------|
| **Promotion** | Kargo | Discover artifacts, bundle as Freight, orchestrate stage transitions, write to Git |
| **Deployment** | Argo CD | Sync cluster to Git, report health, detect drift |
| **Source of truth** | Git | All desired state is versioned and auditable |

**Key insight:** Kargo never deploys directly to Kubernetes. It updates Git (and triggers Argo CD), preserving GitOps principles.

---

# Part 4: Kargo Core Concepts

---

# Projects

A **Project** is Kargo's unit of tenancy.

- Maps to a **Kubernetes namespace**
- Groups related Warehouses, Stages, and promotion pipelines
- Policies and RBAC scoped per project

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: Project
metadata:
  name: kargo-demo
```

---

# Warehouses

A **Warehouse** monitors repositories for new artifact revisions.

When new revisions are found, Kargo packages them into **Freight**.

**Subscriptions can include:**
- Container images (ECR, GCR, Docker Hub, etc.)
- Git repositories
- Helm charts (OCI or HTTP)
- Other artifact sources

```yaml
spec:
  subscriptions:
  - image:
      repoURL: public.ecr.aws/nginx/nginx
      constraint: ^1.29.0
      discoveryLimit: 5
```

> Think of a Warehouse as the **fulfillment center** that packages items into boxes.

---

# Freight

**Freight** is a "meta-artifact" — a bundle of specific artifact revisions that travel together through the pipeline.

**Example:** A single piece of Freight might contain:
- `nginx:1.29.1` (container image)
- `abc123def` (Git commit with manifests)
- `my-chart-2.4.0` (Helm chart version)

> Think of Freight as a **shipping box** — all items inside stay together from warehouse to production.

Freight is **color-coded** in the Kargo UI to show which Stages are using it.

---

# Stages

A **Stage** is a promotion target representing desired state for part of your application's lifecycle.

- Often maps to an **environment** (test, uat, prod)
- Can represent a subset of microservices or an entire cluster
- Stages link together to form a **promotion pipeline**

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: Stage
metadata:
  name: uat
  namespace: kargo-demo
spec:
  requestedFreight:
  - origin:
      kind: Warehouse
      name: kargo-demo
    sources:
      stages:
      - test          # uat only accepts Freight verified in test
```

---

# Promotions

A **Promotion** is the process of transitioning a Stage to a new state by applying Freight.

Triggered when:
- A user drags Freight to a Stage in the UI
- A user clicks "Promote" and selects Freight
- Auto-promotion policies are configured

Kargo creates a `Promotion` object that executes a sequence of **promotion steps**.

| Status | Meaning |
|--------|---------|
| `Running` | Steps are executing |
| `Succeeded` | All steps completed |
| `Failed` | A step failed (e.g., `fail` step or error) |

---

# Promotion Templates & Tasks

**PromotionTemplate** — defined on a Stage, specifies the steps to run when Freight is promoted.

**PromotionTask** — reusable step sequences shared across Stages.

```yaml
promotionTemplate:
  spec:
    steps:
    - task:
        name: demo-promo-process    # references a PromotionTask
```

Steps support:
- Variables (`vars`)
- Expressions (`${{ }}` syntax)
- Step aliases (`as:`) for referencing outputs
- Composition of built-in steps and custom tasks

---

# Part 5: Promotion Steps

---

# Built-in Promotion Steps (Overview)

Kargo ships with **40+ built-in promotion steps**:

| Category | Steps |
|----------|-------|
| **Git** | `git-clone`, `git-commit`, `git-push`, `git-clear`, `git-open-pr`, `git-wait-for-pr`, `git-merge-pr`, `git-tag` |
| **Argo CD** | `argocd-update`, `argocd-wait` |
| **Kustomize** | `kustomize-set-image`, `kustomize-build` |
| **Helm** | `helm-template`, `helm-update-chart` |
| **OCI** | `oci-download`, `oci-push` |
| **OpenTofu** | `tf-plan`, `tf-apply`, `tf-output`, `hcl-update` |
| **Integrations** | `http`, `send-message`, `jira`, `snow-*`, `gha-*`, `jfrog-evidence` |
| **Utility** | `copy`, `delete`, `yaml-update`, `json-update`, `compose-output`, `fail` |
| **Custom** | User-provided container images |

---

# Typical Promotion Flow

A common promotion process for a Kustomize-based app:

```yaml
steps:
- uses: git-clone          # Clone main + stage branch
- uses: git-clear           # Clear output directory
- uses: kustomize-set-image # Update image tag in base
- uses: kustomize-build     # Render manifests for this stage
- uses: git-commit          # Commit rendered manifests
- uses: git-push            # Push to stage/<stage> branch
- uses: argocd-update       # Trigger Argo CD sync
```

Each step is discrete, composable, and auditable.

---

# argocd-update Step

The bridge between Kargo and Argo CD.

**Capabilities:**
- Update `Application` source revisions
- Force sync via the `operation` field
- Select apps by name or label selector
- Register ongoing health checks for the Stage

```yaml
- uses: argocd-update
  config:
    apps:
    - name: kargo-demo-${{ ctx.stage }}
      sources:
      - repoURL: https://github.com/example/repo.git
        desiredRevision: ${{ task.outputs.commit.commit }}
```

> Usually the **last step** in a promotion process.

---

# Authorizing Kargo to Update Applications

Kargo is multi-tenant. Even with cluster RBAC, each `Application` must explicitly authorize a Stage:

```yaml
metadata:
  annotations:
    kargo.akuity.io/authorized-stage: "kargo-demo:test"
```

- Format: `<project-name>:<stage-name>`
- Presence of annotation = delegation of authority
- Only users who can edit the Application can add this annotation

**ApplicationSet example:**

```yaml
template:
  metadata:
    annotations:
      kargo.akuity.io/authorized-stage: kargo-demo:{{stage}}
```

---

# Health Checks

When `argocd-update` completes, Kargo registers **health checks** on the target Stage.

Stage health considers:
- Health of managed Argo CD `Application` resources
- Sync status of those Applications
- Whether the latest Promotion succeeded
- Custom verification steps

A Stage shows ❤️ when healthy, enabling promotion to downstream Stages.

> **Note:** Stage health is not determined solely by Application health — failed Promotions also affect Stage health.

---

# Part 6: Promotion Pipeline Example

---

# Pipeline: test → uat → prod

```
Warehouse                    test Stage              uat Stage              prod Stage
(polls ECR)                  (direct subscribe)      (subscribe: test)      (subscribe: uat)
     │                              │                       │                      │
     ▼                              ▼                       ▼                      ▼
  Freight ──────────────────▶  Promote ──────────▶  Promote ──────────▶  Promote
  (new image)                  (git + sync)          (git + sync)           (git + sync)
```

**Rules:**
- `test` subscribes directly to the Warehouse
- `uat` only accepts Freight **verified in test**
- `prod` only accepts Freight **verified in uat**
- Freight cannot skip stages

---

# What Happens During a Promotion to Test

1. User promotes Freight to the `test` Stage
2. Kargo clones `main` and creates/checks out `stage/test` branch
3. `kustomize-set-image` updates the image tag in `base/`
4. `kustomize-build` renders manifests for `stages/test/`
5. `git-commit` + `git-push` write to `stage/test` branch
6. `argocd-update` tells Argo CD Application `kargo-demo-test` to sync
7. Argo CD deploys the new manifests to the `kargo-demo-test` namespace
8. Kargo monitors Application health until Stage is healthy
9. Freight becomes **verified** in test — available for uat promotion

---

# Git as Audit Trail

Every promotion produces Git commits:

```
main
 └── stage/test   ← updated by Kargo promotion to test
 └── stage/uat    ← updated by Kargo promotion to uat
 └── stage/prod   ← updated by Kargo promotion to prod
```

Benefits:
- Full history of what was promoted, when, and by whom
- Easy rollback via Git revert
- Argo CD can diff any revision
- Compliance and audit requirements met

---

# Part 7: Common Patterns

---

# Pattern: Kustomize Base + Overlays

```
gitops-repo/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── stages/
    ├── test/
    │   └── kustomization.yaml    # references ../../base
    ├── uat/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

Kargo updates the image in `base/`, builds each stage overlay, and writes rendered output to stage-specific branches.

---

# Pattern: Pull Request Gates

For environments requiring human review:

```yaml
steps:
- uses: git-push
  as: push
  config:
    generateTargetBranch: true
- uses: git-open-pr
  as: open-pr
  config:
    sourceBranch: ${{ outputs.push.branch }}
    targetBranch: stage/${{ ctx.stage }}
- uses: git-wait-for-pr
  config:
    prNumber: ${{ outputs['open-pr'].pr.id }}
- uses: argocd-update
  config:
    apps:
    - name: my-app-${{ ctx.stage }}
      sources:
      - desiredRevision: ${{ outputs['wait-for-pr'].commit }}
```

---

# Pattern: Helm Charts

```yaml
steps:
- uses: git-clone
- uses: helm-update-chart      # Update chart dependencies
- uses: helm-template          # Render chart to manifests
- uses: git-commit
- uses: git-push
- uses: argocd-update
```

Kargo works equally well with Helm and Kustomize.

---

# Pattern: Infrastructure (OpenTofu)

Kargo supports infrastructure promotion via OpenTofu steps:

- `hcl-update` — modify OpenTofu configuration
- `tf-plan` — preview infrastructure changes
- `tf-apply` — apply changes
- `tf-output` — capture outputs for subsequent steps

Enables promoting both application config and infrastructure together.

---

# Pattern: External Integrations

| Integration | Use Case |
|-------------|----------|
| `send-message` | Slack/email notifications on promotion |
| `jira` | Create/update tickets for promotions |
| `snow-*` | ServiceNow change management gates |
| `gha-dispatch-workflow` | Trigger GitHub Actions for tests |
| `gha-wait-for-workflow` | Wait for CI to pass before continuing |
| `jfrog-evidence` | Supply chain evidence in Artifactory |

---

# Part 8: Comparison

---

# Argo CD vs Kargo

| Dimension | Argo CD | Kargo |
|-----------|---------|-------|
| **Primary role** | Deploy (sync cluster to Git) | Promote (move changes between stages) |
| **Watches** | Git repositories | Image registries, Git, Helm repos |
| **Core unit** | Application | Freight (bundled artifacts) |
| **Pipeline** | No native pipeline | Stages linked in promotion pipeline |
| **Writes to Git** | No (reads Git) | Yes (commits promoted config) |
| **Multi-stage flow** | Manual or custom CI scripts | Built-in, declarative |
| **Verification** | App health only | Stage health + verifications + integrations |
| **UI focus** | Application sync status | Promotion timeline and Freight flow |

**They are complementary, not competing.**

---

# CI vs Kargo vs Argo CD

| Concern | Traditional CI | Kargo | Argo CD |
|---------|---------------|-------|---------|
| Build container images | ✅ | ❌ | ❌ |
| Run unit tests | ✅ | ❌ (can trigger via GHA) | ❌ |
| Promote between envs | ⚠️ Custom scripts | ✅ | ❌ |
| Deploy to Kubernetes | ⚠️ kubectl/helm | ❌ | ✅ |
| Git as source of truth | ⚠️ Often bypassed | ✅ | ✅ |
| Audit trail | ⚠️ CI logs only | ✅ Git history | ✅ Git history |

**Ideal setup:** CI builds and tests → Kargo promotes → Argo CD deploys.

---

# Part 9: Best Practices

---

# Best Practices

### Git Repository Layout
- Use **stage-specific branches** (`stage/test`, `stage/uat`, `stage/prod`)
- Separate base config from stage overlays (Kustomize or Helm values)
- Keep the GitOps repo focused on deployment config, not application source

### Security
- Always use `kargo.akuity.io/authorized-stage` annotations
- Scope Kargo credentials per project (Git PATs as Secrets)
- Use Kargo Projects for multi-tenancy and RBAC

### Promotion Design
- Define reusable `PromotionTask` resources
- Add verification steps before downstream stages
- Use PR-based gates for production promotions

---

# Best Practices (continued)

### Argo CD Configuration
- Use **ApplicationSet** for multi-environment Applications
- Use Argo CD **v2.11.0+** for proper sync window support with `argocd-update`
- Configure sync policies appropriate per environment (auto-sync for test, manual for prod)

### Operations
- Monitor Stage health in the Kargo dashboard
- Use Freight aliases for human-readable version names
- Leverage the Kargo CLI for automation: `kargo promote`, `kargo get freight`

### Installation
- Kargo requires **Helm v3.13.1+**
- cert-manager is a prerequisite
- Can be installed alongside Argo CD via the official quickstart script

---

# Part 10: Getting Started

---

# Quickstart (Local)

```bash
# 1. Install Kargo + Argo CD + cert-manager locally
curl -L https://raw.githubusercontent.com/akuity/kargo/main/hack/quickstart/install.sh | sh

# 2. Access dashboards
# Argo CD: http://localhost:31080  (admin/admin)
# Kargo:   http://localhost:31081  (admin)

# 3. Fork https://github.com/akuity/kargo-demo
# 4. Create ApplicationSet, Warehouse, Stages
# 5. Promote Freight through test → uat → prod
```

Full guide: https://docs.kargo.io/quickstart

---

# Key CRDs Summary

| CRD | API Group | Purpose |
|-----|-----------|---------|
| `Project` | `kargo.akuity.io/v1alpha1` | Tenancy and organization |
| `Warehouse` | `kargo.akuity.io/v1alpha1` | Artifact discovery |
| `Stage` | `kargo.akuity.io/v1alpha1` | Promotion target / environment |
| `Promotion` | `kargo.akuity.io/v1alpha1` | Running promotion instance |
| `PromotionTask` | `kargo.akuity.io/v1alpha1` | Reusable step sequence |
| `Freight` | `kargo.akuity.io/v1alpha1` | Bundled artifacts (auto-created) |
| `Application` | `argoproj.io/v1alpha1` | Argo CD deployment unit |
| `ApplicationSet` | `argoproj.io/v1alpha1` | Generate multiple Applications |

---

# Ecosystem & Adoption

### Argo CD
- CNCF incubating project
- De facto GitOps deployment tool for Kubernetes
- 2025 CNCF survey: **97%** of Argo CD respondents run it in production

### Kargo
- Created by Akuity (Argo Project founders)
- GA since October 2024
- Adopted by Deutsche Telekom, JumpCloud, Cisco ThousandEyes, and others
- Active development with frequent releases

### Akuity Platform
- Enterprise managed offering for Argo CD and Kargo
- Custom promotion steps (container-based extensibility)

---

# Summary

| | Argo CD | Kargo |
|---|---------|-------|
| **One-liner** | "Make the cluster match Git" | "Move validated changes through stages" |
| **Analogy** | The **delivery truck** | The **logistics coordinator** |
| **When to adopt** | You need GitOps deployment | You need multi-stage promotion pipelines |

**Together:** CI builds artifacts → Kargo promotes through stages → Argo CD deploys each stage → Git records everything.

---

# References

- **Argo CD Docs:** https://argo-cd.readthedocs.io/
- **Argo CD GitHub:** https://github.com/argoproj/argo-cd
- **Kargo Docs:** https://docs.kargo.io/
- **Kargo GitHub:** https://github.com/akuity/kargo
- **Kargo Quickstart:** https://docs.kargo.io/quickstart
- **Argo CD Integration:** https://docs.kargo.io/user-guide/how-to-guides/argo-cd-integration
- **Promotion Steps Reference:** https://docs.kargo.io/user-guide/reference-docs/promotion-steps
- **Kargo Patterns:** https://docs.kargo.io/user-guide/patterns
- **Akuity Blog:** https://akuity.io/blog

---

# Thank You

**Questions?**

Explore the demo:
1. Install locally with the Kargo quickstart script
2. Fork `akuity/kargo-demo`
3. Promote Freight through test → uat → prod
4. Watch Argo CD sync each stage in real time
