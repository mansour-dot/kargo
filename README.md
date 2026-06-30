# Kargo & Argo CD Presentation

A detailed technical presentation on **Argo CD** and **Kargo** — how they work together in a GitOps-based continuous delivery pipeline for Kubernetes.

## Contents

| File | Description |
|------|-------------|
| [presentation/kargo-and-argocd.md](presentation/kargo-and-argocd.md) | Full slide deck (Marp-compatible Markdown) |

## Viewing the Presentation

### Option 1: Read as Markdown
Open `presentation/kargo-and-argocd.md` in any Markdown viewer or GitHub.

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
