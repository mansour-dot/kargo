# Kubernetes GitOps Presentations

Technical presentations on **Helm**, **Argo CD**, **Kargo**, and **Istio** — from beginner-friendly guides to deep integration details for Kubernetes delivery and service mesh.

## Contents

| File | Description | Audience |
|------|-------------|----------|
| [presentation/helm-practical-example.md](presentation/helm-practical-example.md) | **Start here for Helm** — package manager basics with hands-on examples | Beginners |
| [presentation/istio-practical-example.md](presentation/istio-practical-example.md) | **Start here for Istio** — service mesh basics with practical examples | Beginners |
| [presentation/practical-example.md](presentation/practical-example.md) | Kargo & Argo CD — simple guide with practical examples | Beginners |
| [presentation/git-strategy-practical-example.md](presentation/git-strategy-practical-example.md) | Common Git strategies (Git Flow, GitHub Flow, trunk-based, etc.) | Beginners / teams choosing a branching model |
| [presentation/kargo-and-argocd.md](presentation/kargo-and-argocd.md) | Full technical slide deck | Intermediate / advanced |

## Which Presentation Should I Read?

- **New to Helm?** → Open `presentation/helm-practical-example.md`
- **New to Istio / service mesh?** → Open `presentation/istio-practical-example.md`
- **New to Kargo and Argo CD?** → Open `presentation/practical-example.md`
- **Choosing a Git branching model (Git Flow, GitHub Flow, etc.)?** → Open `presentation/git-strategy-practical-example.md`
- **Need architecture, CRDs, and deep integration details?** → Open `presentation/kargo-and-argocd.md`

## Viewing the Presentation

### Option 1: Read as Markdown
Open either presentation file in any Markdown viewer or GitHub.

### Option 2: Export to PDF/HTML with Marp
```bash
npm install -g @marp-team/marp-cli
marp presentation/kargo-and-argocd.md --pdf
# or
marp presentation/kargo-and-argocd.md --html
```

### Option 3: VS Code / Cursor
Install the [Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode) extension and open the presentation file in slide preview mode.

## Sources

Information in these presentations is based on official documentation:

- [Helm Documentation](https://helm.sh/docs/)
- [Helm Chart Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Argo CD Documentation](https://argo-cd.readthedocs.io/)
- [Kargo Documentation](https://docs.kargo.io/)
- [Kargo Quickstart](https://docs.kargo.io/quickstart)
- [Argo CD Integration Guide](https://docs.kargo.io/user-guide/how-to-guides/argo-cd-integration)
- [Kargo Git Patterns](https://docs.kargo.io/user-guide/patterns)
- [Argo CD Best Practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)
- [Istio Documentation](https://istio.io/latest/docs/)
- [Istio Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)
