---
marp: true
theme: default
paginate: true
header: 'Helm — Practical Examples'
footer: 'Simple guide for everyone'
style: |
  section { font-size: 26px; }
  h1 { color: #2563eb; }
  h2 { color: #1e40af; }
  code { background: #f1f5f9; font-size: 22px; }
  table { font-size: 22px; }
---

# Helm
## Practical Examples (Beginner Friendly)

**From zero to deploying apps on Kubernetes — with real commands you can run**

---

# Who Is This For?

This guide is for you if:

- You heard about **Helm** but don't know where to start
- You deploy apps to **Kubernetes** and YAML files feel repetitive
- You want **hands-on examples**, not abstract theory

**You do NOT need to be a Kubernetes expert to follow this guide.**

---

# Before Helm: What Is Kubernetes?

**Kubernetes (K8s)** runs your apps in containers across many servers.

You tell Kubernetes what you want with **YAML files**:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
```

To run this app you also need: Service, Ingress, ConfigMap, Secret…
**One app = many YAML files.** That gets messy fast.

---

# The Problem (Without Helm)

Imagine deploying a **web shop** to 3 environments:

| Environment | Replicas | Domain | Database |
|-------------|----------|--------|----------|
| **Dev** | 1 | dev.shop.com | dev-db |
| **Staging** | 2 | staging.shop.com | staging-db |
| **Production** | 10 | shop.com | prod-db |

**Without Helm you must:**
1. Copy 15+ YAML files per environment
2. Manually change values in each file
3. Keep all copies in sync when something changes
4. Remember exact `kubectl apply` order

**Result:** Copy-paste errors, drift, slow releases.

---

# What Is Helm? (1 minute)

**Helm is a package manager for Kubernetes.**

| Real world | Helm equivalent |
|------------|-----------------|
| App Store | **Helm Repository** |
| `.deb` / `.rpm` package | **Chart** |
| Installed app on your phone | **Release** |
| App settings screen | **values.yaml** |

**One command installs a full app** (Deployment + Service + Ingress + more).

---

# Helm in One Picture

```
  ┌─────────────┐     helm install      ┌──────────────────┐
  │   Chart     │  ──────────────────►  │  Kubernetes      │
  │  (template  │     fills in values     │  Cluster         │
  │   + values) │                         │  (running app)   │
  └─────────────┘                         └──────────────────┘
        ▲
        │ you customize
  ┌─────────────┐
  │ values.yaml │
  └─────────────┘
```

**Chart** = blueprint. **Values** = your choices. **Release** = live deployment.

---

# 4 Words You Must Know

| Word | Simple meaning | Example |
|------|----------------|---------|
| **Chart** | Folder of templates + default config | `nginx-chart/` |
| **Release** | One installed instance of a chart | `my-nginx-prod` |
| **Repository** | Website that hosts charts | `https://charts.bitnami.com/bitnami` |
| **Values** | Settings you pass to a chart | `replicaCount: 3` |

---

# Install Helm (Step 1)

**Linux / macOS:**
```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**Verify:**
```bash
helm version
# version.BuildInfo{Version:"v3.x.x", ...}
```

**You need:**
- `kubectl` configured to a cluster
- Helm 3 (current version — no Tiller needed)

---

# Your First Helm Command (Step 2)

Search for available charts:
```bash
helm search hub nginx
```

Add a popular chart repository:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

List charts in that repo:
```bash
helm search repo nginx
```

---

# Example 1: Install Nginx in 30 Seconds

**Plain `kubectl` way:** write 3–5 YAML files, then `kubectl apply -f ...`

**Helm way:**
```bash
helm install my-nginx bitnami/nginx \
  --namespace web \
  --create-namespace
```

**What happened?**
- Helm created Deployment, Service, and more
- Named the release `my-nginx`
- Stored release history in the cluster

---

# Example 1: Check What Was Created

```bash
# List all Helm releases
helm list -n web

# See Kubernetes resources Helm created
kubectl get all -n web

# See the values Helm used
helm get values my-nginx -n web
```

**Output of `helm list`:**
```
NAME       NAMESPACE  REVISION  STATUS   CHART
my-nginx   web        1         deployed nginx-18.x.x
```

---

# Example 2: Customize with Values

Charts accept settings without editing templates.

**Inline values:**
```bash
helm install my-nginx bitnami/nginx \
  --namespace web \
  --set replicaCount=3 \
  --set service.type=LoadBalancer
```

**Values file (recommended for teams):**
```yaml
# nginx-values.yaml
replicaCount: 3
service:
  type: LoadBalancer
  port: 80
resources:
  limits:
    cpu: 200m
    memory: 256Mi
```

```bash
helm install my-nginx bitnami/nginx \
  -n web -f nginx-values.yaml
```

---

# Example 3: Upgrade a Release

App version `1.27` → `1.28`? Change values and upgrade:

```bash
helm upgrade my-nginx bitnami/nginx \
  -n web \
  -f nginx-values.yaml \
  --set image.tag=1.28
```

**Helm creates revision 2** — previous config is saved.

```bash
helm history my-nginx -n web
```

```
REVISION  STATUS     CHART         DESCRIPTION
1         superseded nginx-18.x.x  Install complete
2         deployed   nginx-18.x.x  Upgrade complete
```

---

# Example 4: Rollback When Something Breaks

Upgrade caused a problem? Roll back in one command:

```bash
helm rollback my-nginx 1 -n web
```

**Revision 1 is restored.** No manual YAML hunting.

```bash
helm history my-nginx -n web
# REVISION 3 → rolled back to 1
```

**This is one of Helm's biggest wins: safe, fast rollbacks.**

---

# Example 5: Uninstall Cleanly

```bash
helm uninstall my-nginx -n web
```

Helm removes all resources it created (with labels tracking ownership).

**Compare to kubectl:** you'd need to remember every file you applied.

---

# Anatomy of a Chart

```
my-shop/
├── Chart.yaml          # chart metadata (name, version)
├── values.yaml         # default settings
├── charts/             # dependent subcharts (optional)
└── templates/          # Kubernetes YAML with Go templates
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    └── _helpers.tpl    # reusable template snippets
```

**You edit `values.yaml` and templates — Helm renders final YAML.**

---

# Example 6: Create Your Own Chart

```bash
helm create my-shop
```

Helm generates a starter chart with Deployment, Service, Ingress, HPA, and tests.

**Look at the structure:**
```bash
tree my-shop/
```

**Dry-run — see rendered YAML without deploying:**
```bash
helm install shop my-shop/ --dry-run --debug
```

---

# Example 7: A Simple Custom Chart

**`Chart.yaml`:**
```yaml
apiVersion: v2
name: my-shop
description: A simple web shop
type: application
version: 0.1.0
appVersion: "1.0.0"
```

**`values.yaml`:**
```yaml
replicaCount: 2
image:
  repository: ghcr.io/my-team/my-shop
  tag: "1.0.0"
service:
  port: 80
```

---

# Example 7: Template with Values

**`templates/deployment.yaml`:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-shop.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

**`{{ .Values.replicaCount }}`** is replaced at install time.
Change `values.yaml` → same chart, different environments.

---

# Example 8: One Chart, Three Environments

**Base `values.yaml` (defaults):**
```yaml
replicaCount: 1
image:
  tag: "latest"
ingress:
  enabled: false
```

**`values-dev.yaml`:**
```yaml
replicaCount: 1
ingress:
  enabled: true
  host: dev.shop.com
```

**`values-prod.yaml`:**
```yaml
replicaCount: 10
image:
  tag: "2.0.0"
ingress:
  enabled: true
  host: shop.com
```

---

# Example 8: Deploy Each Environment

```bash
# Dev
helm upgrade --install shop-dev ./my-shop \
  -n shop-dev --create-namespace \
  -f values.yaml -f values-dev.yaml

# Production
helm upgrade --install shop-prod ./my-shop \
  -n shop-prod --create-namespace \
  -f values.yaml -f values-prod.yaml
```

**`--install`** = install if missing, upgrade if exists (idempotent — safe for CI).

---

# Example 9: `helm template` (No Cluster Needed)

Render YAML locally — great for review and CI:

```bash
helm template shop-prod ./my-shop \
  -f values.yaml -f values-prod.yaml \
  > rendered-prod.yaml
```

**Use cases:**
- Code review before deploy
- Validate chart in CI pipeline
- Debug template errors without touching cluster

---

# Example 10: Chart Dependencies (Subcharts)

Your shop needs a database. Don't reinvent it — depend on an existing chart.

**`Chart.yaml`:**
```yaml
apiVersion: v2
name: my-shop
version: 0.2.0
dependencies:
  - name: postgresql
    version: "15.x.x"
    repository: https://charts.bitnami.com/bitnami
```

```bash
helm dependency update ./my-shop   # downloads postgresql chart
helm install shop ./my-shop -f values.yaml
```

**One release deploys app + database together.**

---

# Example 11: Conditions and Logic in Templates

Enable Ingress only when needed:

**`values.yaml`:**
```yaml
ingress:
  enabled: true
  host: shop.example.com
```

**`templates/ingress.yaml`:**
```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "my-shop.fullname" . }}
spec:
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: {{ include "my-shop.fullname" . }}
            port:
              number: {{ .Values.service.port }}
{{- end }}
```

If `ingress.enabled: false` → Ingress resource is **not** created.

---

# Example 12: Helm Hooks (Advanced but Useful)

Run jobs at specific lifecycle points:

| Hook | When it runs |
|------|--------------|
| `pre-install` | Before resources are created |
| `post-install` | After all resources are ready |
| `pre-upgrade` | Before upgrade |
| `pre-delete` | Before uninstall |

**Use case:** database migration before new app version starts.

```yaml
# templates/db-migrate-job.yaml
metadata:
  annotations:
    "helm.sh/hook": pre-upgrade,pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
```

---

# Example 13: Secrets in Helm

**Never put real passwords in `values.yaml` committed to Git.**

**Option A — pass at install time:**
```bash
helm install shop ./my-shop \
  --set database.password="$(cat /run/secrets/db-pass)"
```

**Option B — external secrets operator (production pattern):**
- Helm deploys the app skeleton
- External Secrets Operator injects credentials from Vault/AWS SM

**Option C — Sealed Secrets / SOPS** (encrypted in Git)

---

# Example 14: Lint and Test Your Chart

Before every release:

```bash
# Check chart structure and templates
helm lint ./my-shop

# Render + validate against Kubernetes API
helm template shop ./my-shop -f values-prod.yaml | kubectl apply --dry-run=client -f -

# Run chart tests (if defined)
helm test shop-prod -n shop-prod
```

**Add `helm lint` to your CI pipeline — catches errors early.**

---

# Example 15: Package and Share Your Chart

```bash
# Package chart into a .tgz file
helm package ./my-shop
# my-shop-0.2.0.tgz

# Push to OCI registry (modern approach)
helm push my-shop-0.2.0.tgz oci://ghcr.io/my-team/charts
```

**Others install with:**
```bash
helm install shop oci://ghcr.io/my-team/charts/my-shop --version 0.2.0
```

---

# Helm vs Plain kubectl

| Task | kubectl | Helm |
|------|---------|------|
| Install app with 10 resources | `kubectl apply -f` × 10 | `helm install` × 1 |
| Change replica count | Edit YAML, re-apply | `helm upgrade --set replicaCount=5` |
| Rollback bad deploy | Find old YAML, re-apply | `helm rollback release 1` |
| Share app with team | Send folder of YAMLs | Share one chart |
| Track what's deployed | Manual labels/notes | `helm list` + revision history |

---

# Helm vs Kustomize (Quick Comparison)

| | **Helm** | **Kustomize** |
|---|----------|---------------|
| Approach | Templates + values | Overlays + patches |
| Package format | Chart (.tgz) | Kustomization.yaml |
| Rollback built-in | Yes | No (use Git) |
| Best for | Packaged apps, marketplaces | Git-native config repos |

**Many teams use both:** Helm chart as base, Kustomize overlay on top.

---

# Helm in a GitOps Pipeline

**Helm fits naturally with Argo CD and Kargo:**

```
Developer → CI builds image
         → updates values.yaml (image tag)
         → Git commit
         → Argo CD syncs Helm chart from Git
         → Cluster updated
```

**Argo CD Application with Helm:**
```yaml
spec:
  source:
    repoURL: https://github.com/my-team/shop-config.git
    path: charts/my-shop
    helm:
      valueFiles:
      - values-prod.yaml
```

Argo CD runs `helm template` and applies the result.

---

# Real Scenario: Day in the Life

**09:00** — Developer pushes `my-shop:2.1.0` to registry
**09:05** — CI updates `values-prod.yaml` image tag, commits to Git
**09:10** — Argo CD detects change, runs Helm sync
**09:12** — Production running `2.1.0` (revision 14)
**11:00** — Bug found in `2.1.0`
**11:02** — `helm rollback shop-prod 13` (or Git revert + Argo CD sync)
**11:05** — Production back on `2.0.9` ✅

---

# Common Mistakes (And Fixes)

| Mistake | Fix |
|---------|-----|
| Editing live resources with `kubectl edit` | Always change values + `helm upgrade` |
| Secrets in Git values files | Use `--set`, External Secrets, or SOPS |
| Skipping `helm lint` | Add lint step to CI |
| Same release name twice | Use unique names or `helm upgrade --install` |
| Orphan resources after uninstall | Ensure resources have Helm labels/annotations |
| `--set` for complex config | Use `-f values.yaml` files instead |

---

# Helm Command Cheat Sheet

```bash
helm repo add <name> <url>       # add chart repository
helm search repo <keyword>       # find charts
helm install <release> <chart>   # deploy
helm upgrade <release> <chart>     # update
helm rollback <release> <rev>    # undo
helm uninstall <release>         # remove
helm list                        # show releases
helm get values <release>        # show current values
helm history <release>           # show revisions
helm template <release> <chart>  # render locally
helm lint <chart>                # validate chart
helm create <name>               # scaffold new chart
```

---

# Try It Yourself (Mini Lab)

**Prerequisites:** kubectl + a cluster (minikube, kind, or cloud)

```bash
# 1. Install Helm
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# 2. Start local cluster (kind example)
kind create cluster

# 3. Install nginx
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install demo-nginx bitnami/nginx -n demo --create-namespace

# 4. Verify
kubectl get pods -n demo
helm list -n demo

# 5. Upgrade replicas
helm upgrade demo-nginx bitnami/nginx -n demo --set replicaCount=3

# 6. Rollback
helm rollback demo-nginx 1 -n demo

# 7. Create your own chart
helm create hello-world
helm install hello ./hello-world --dry-run --debug

# 8. Cleanup
helm uninstall demo-nginx -n demo
kind delete cluster
```

**Time needed: ~20 minutes.**

---

# Practice Questions

Test your understanding:

1. What is the difference between a **Chart** and a **Release**?
2. How do you deploy to Production with 10 replicas without editing templates?
3. How do you undo a bad upgrade?
4. What command shows rendered YAML without deploying?

---

# Practice Answers

1. **Chart** = package template. **Release** = one live installed instance of that chart.
2. `helm upgrade --install shop-prod ./my-shop -f values-prod.yaml` where `values-prod.yaml` sets `replicaCount: 10`.
3. `helm rollback <release> <revision-number>`
4. `helm template <release> <chart> -f values.yaml`

---

# Final Summary

If you remember only 4 things:

1. **Helm packages Kubernetes apps** into reusable Charts
2. **Values** let you customize without copy-pasting YAML
3. **`helm upgrade` + `helm rollback`** make releases safe and reversible
4. **Helm works with GitOps** (Argo CD, Flux) — Git stays the source of truth

---

# How Helm Fits With This Repo's Other Guides

| Guide | What it covers |
|-------|----------------|
| `helm-practical-example.md` | **This file** — package & deploy with Helm |
| `practical-example.md` | Promote releases with Kargo + Argo CD |
| `git-strategy-practical-example.md` | Where to store Helm charts in Git |
| `kargo-and-argocd.md` | Deep technical integration |

**Typical stack:** Helm charts in Git → Argo CD deploys → Kargo promotes between environments.

---

# Next Steps

- **Beginner:** run the Mini Lab above on a local cluster
- **Intermediate:** create a chart for your own app, add `helm lint` to CI
- **Advanced:** publish chart to OCI registry, integrate with Argo CD

**You are ready to explain Helm to your team.**

---

# Thank You

**Official docs:**
- [Helm Documentation](https://helm.sh/docs/)
- [Chart Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Argo CD + Helm](https://argo-cd.readthedocs.io/en/stable/user-guide/helm/)

Questions? Try the Mini Lab — hands-on practice beats reading alone.
