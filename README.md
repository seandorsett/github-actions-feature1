# GitHub Actions: Reusable Workflows — Office Hours Showcase

> **Office Hours Session** — 20-minute feature showcase  
> **Feature**: GitHub Actions Reusable Workflows (enhanced November 2025)  
> **What's New**: Up to **10 nesting levels** (was 4) and **50 workflow calls per run** (was 20)

---

## 📂 Repository Contents

| Path | Description |
|------|-------------|
| [`slides/presentation.md`](./slides/presentation.md) | Marp-compatible slide deck for the 20-minute showcase |
| [`demo/README.md`](./demo/README.md) | Step-by-step guide to running the demo |
| [`.github/workflows/reusable-ci.yml`](./.github/workflows/reusable-ci.yml) | Reusable workflow: Build & Test |
| [`.github/workflows/reusable-security-scan.yml`](./.github/workflows/reusable-security-scan.yml) | Reusable workflow: Security Scan |
| [`.github/workflows/reusable-deploy.yml`](./.github/workflows/reusable-deploy.yml) | Reusable workflow: Deploy |
| [`.github/workflows/main-pipeline.yml`](./.github/workflows/main-pipeline.yml) | Orchestrator: calls all 3 reusable workflows |

---

## 🎯 Feature Highlighted

**GitHub Actions Reusable Workflows** — announced **November 2025** — received a major limit increase:

| Limit | Before | After |
|-------|--------|-------|
| Nesting levels | 4 | **10** |
| Workflow calls per run | 20 | **50** |

Reusable workflows let you define CI/CD logic once and call it from any workflow across your organization — like functions for your pipelines.

---

## 🚀 Running the Demo

1. Go to the **Actions** tab
2. Select **"Demo: Main Pipeline (Orchestrator)"**
3. Click **"Run workflow"** and fill in the parameters
4. Watch three reusable workflows execute in sequence, passing outputs between them

---

## 📑 Viewing the Slides

The slide deck (`slides/presentation.md`) is written in [Marp](https://marp.app/) format.

**To view as a presentation:**

```bash
# Install Marp CLI
npm install -g @marp-team/marp-cli

# Export to HTML
marp slides/presentation.md --output slides/presentation.html

# Export to PDF
marp slides/presentation.md --pdf --output slides/presentation.pdf
```

Or use the [Marp for VS Code extension](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode) to preview directly in your editor.

---

## 📚 Resources

- [GitHub Docs: Reusing Workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows)
- [Nov 2025 Changelog: New Limits](https://github.blog/changelog/2025-11-06-new-releases-for-github-actions-november-2025/)
- [GitHub Blog: Using Reusable Workflows](https://github.blog/developer-skills/github/using-reusable-workflows-github-actions/)