# Kargo & Argo CD Presentation

A detailed technical presentation on **Argo CD** and **Kargo** — how they work together in a GitOps-based continuous delivery pipeline for Kubernetes.

## Contents

| File | Description | Audience |
|------|-------------|----------|
| [presentation/practical-example.md](presentation/practical-example.md) | **Start here** — simple guide with practical examples | Beginners |
| [presentation/git-strategy-practical-example.md](presentation/git-strategy-practical-example.md) | Git strategy for GitOps (branches, repos, layouts) | Beginners / teams designing repos |
| [presentation/kargo-and-argocd.md](presentation/kargo-and-argocd.md) | Full technical slide deck | Intermediate / advanced |

## Which Presentation Should I Read?

- **New to Kargo and Argo CD?** → Open `presentation/practical-example.md`
- **Designing your Git repo layout and branch strategy?** → Open `presentation/git-strategy-practical-example.md`
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

Information in this presentation is based on official documentation:

- [Argo CD Documentation](https://argo-cd.readthedocs.io/)
- [Kargo Documentation](https://docs.kargo.io/)
- [Kargo Quickstart](https://docs.kargo.io/quickstart)
- [Argo CD Integration Guide](https://docs.kargo.io/user-guide/how-to-guides/argo-cd-integration)
- [Kargo Git Patterns](https://docs.kargo.io/user-guide/patterns)
- [Argo CD Best Practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)
