---
marp: true
theme: default
paginate: true
backgroundColor: #0d1117
color: #e6edf3
style: |
  section {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial, sans-serif;
    padding: 40px 60px;
  }
  h1 {
    color: #58a6ff;
    border-bottom: 2px solid #21262d;
    padding-bottom: 12px;
  }
  h2 {
    color: #79c0ff;
  }
  h3 {
    color: #a5f3fc;
  }
  code {
    background-color: #161b22;
    color: #e6edf3;
    border: 1px solid #30363d;
    border-radius: 6px;
    padding: 2px 6px;
  }
  pre {
    background-color: #161b22 !important;
    border: 1px solid #30363d;
    border-radius: 8px;
    padding: 20px;
  }
  pre code {
    border: none;
    padding: 0;
    background: transparent;
    font-size: 0.85em;
  }
  .highlight {
    background-color: #1f2937;
    border-left: 4px solid #58a6ff;
    padding: 12px 20px;
    border-radius: 0 8px 8px 0;
    margin: 12px 0;
  }
  .new-badge {
    background-color: #238636;
    color: #ffffff;
    padding: 3px 10px;
    border-radius: 20px;
    font-size: 0.75em;
    font-weight: bold;
    margin-left: 10px;
  }
  table {
    width: 100%;
    border-collapse: collapse;
  }
  th {
    background-color: #21262d;
    color: #58a6ff;
    padding: 10px 16px;
    text-align: left;
    border: 1px solid #30363d;
  }
  td {
    padding: 10px 16px;
    border: 1px solid #30363d;
    background-color: #161b22;
  }
  .emoji-bullet li {
    list-style: none;
    padding-left: 0;
  }
---

<!-- Title Slide -->
# 🔄 GitHub Actions: Reusable Workflows

## New Limits, New Possibilities

**Office Hours Session** | April 2026

---

> *"Write once, run everywhere — at any scale."*

---

## 📋 Agenda

1. **What are Reusable Workflows?** *(3 min)*
2. **How They Work** *(4 min)*
3. **What's New: Increased Limits** *(4 min)*
4. **Live Demo** *(6 min)*
5. **Best Practices** *(2 min)*
6. **Q&A** *(remaining time)*

---

## ❓ The Problem We're Solving

**Without reusable workflows, teams face:**

- 🔁 Copy-pasting the same CI/CD steps across dozens of repositories
- 😰 Inconsistent pipelines — one team uses node 18, another uses node 20
- 🐛 A bug fix in the build step requires updating **every** repository
- 🔐 Security best practices not enforced consistently

<div class="highlight">
💡 <strong>Sound familiar?</strong> Reusable Workflows solve exactly this.
</div>

---

## ✅ What are Reusable Workflows?

Reusable workflows let you **define a workflow once** and **call it from multiple workflows** — across the same or different repositories.

Think of them like **functions** in programming:
- Accept **inputs** (parameters)
- Return **outputs** (results)
- Handle **secrets** securely
- Can be **versioned** (by tag, branch, or SHA)

---

## 🏗️ Architecture Overview

```
Your Repositories                     Shared Workflows Repo
┌──────────────────┐                 ┌────────────────────────────┐
│  repo-frontend   │────────────────▶│  .github/workflows/        │
│  main.yml        │  calls          │    reusable-ci.yml         │
└──────────────────┘                 │    reusable-deploy.yml     │
                                     │    reusable-security.yml   │
┌──────────────────┐                 └────────────────────────────┘
│  repo-backend    │────────────────▶      ▲   ▲   ▲
│  main.yml        │  calls                │   │   │
└──────────────────┘                       │   │   │
                                     Same workflow,
┌──────────────────┐                 called by everyone!
│  repo-mobile     │────────────────▶
│  main.yml        │  calls
└──────────────────┘
```

---

## ⚙️ The `workflow_call` Trigger

A reusable workflow is just a regular workflow file that uses `workflow_call` as its trigger:

```yaml
# .github/workflows/reusable-ci.yml
on:
  workflow_call:          # 👈 This is the magic trigger
    inputs:
      node-version:
        required: true
        type: string
      run-tests:
        required: false
        type: boolean
        default: true
    secrets:
      NPM_TOKEN:
        required: true
    outputs:
      build-version:
        description: "The version that was built"
        value: ${{ jobs.build.outputs.version }}
```

---

## 📞 Calling a Reusable Workflow

From any workflow, call it with `uses`:

```yaml
# .github/workflows/main.yml  (in your app repo)
jobs:
  run-ci:
    uses: my-org/shared-workflows/.github/workflows/reusable-ci.yml@v2
    #     ^─────────────────────^ ^──────────────────────────────^ ^──^
    #     Organization/Repo        Path to the workflow file       Tag/Branch/SHA
    with:
      node-version: "20"
      run-tests: true
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
      # OR use secrets: inherit  👇
```

<div class="highlight">
🔑 <code>secrets: inherit</code> — passes all caller secrets to the reusable workflow automatically (same org/owner only).
</div>

---

## 🆕 What's New: Increased Limits <span class="new-badge">NEW Nov 2025</span>

| Limit | Before | After (Nov 2025) | Improvement |
|-------|--------|-----------------|-------------|
| **Nesting levels** | 4 | **10** | 2.5× deeper |
| **Workflow calls per run** | 20 | **50** | 2.5× more |

**Why does this matter?**

Large organizations can now build truly modular, deeply composed pipelines:
- A "platform" workflow calls a "security" workflow that calls "scanning" sub-workflows
- Enterprise CI/CD with specialized reusable components at every layer
- Multi-team workflows with each team owning their own reusable piece

---

## 🗂️ Deep Nesting — Now Up to 10 Levels!

```
main-pipeline.yml               (Level 1 — caller)
  └── platform-ci.yml           (Level 2 — calls ↓)
        ├── build.yml            (Level 3)
        │     └── lint.yml       (Level 4 — was the limit before!)
        │           └── format.yml  (Level 5)
        │                 └── style-check.yml  (Level 6)
        ├── test.yml             (Level 3)
        │     └── unit-test.yml  (Level 4)
        │           └── coverage.yml  (Level 5)
        └── security.yml         (Level 3)
              └── sast.yml       (Level 4)
                    └── snyk.yml (Level 5)
```

> ✅ Previously stopped at Level 4 — now goes up to **Level 10!**

---

## 📊 50 Workflow Calls Per Run

Before: max **20** unique reusable workflow calls per run  
Now: max **50** unique reusable workflow calls per run

**Real-world scenario:**

```yaml
jobs:
  build-service-1:   uses: ./.github/workflows/reusable-build.yml    # call 1
  build-service-2:   uses: ./.github/workflows/reusable-build.yml    # call 2
  build-service-3:   uses: ./.github/workflows/reusable-build.yml    # call 3
  # ... up to 50 services/components in a monorepo! ...
  deploy-service-1:  uses: ./.github/workflows/reusable-deploy.yml   # call 26
  deploy-service-2:  uses: ./.github/workflows/reusable-deploy.yml   # call 27
  # ... deploy all 50 services! ...
  security-scan-1:   uses: ./.github/workflows/reusable-security.yml # call 49
  security-scan-2:   uses: ./.github/workflows/reusable-security.yml # call 50
```

---

## 🎯 Demo: Our Pipeline

We'll build a **realistic multi-stage pipeline** showcasing:

```
main-pipeline.yml  ◀── Orchestrator
    │
    ├── reusable-ci.yml          Build & test with Node.js
    │       └── outputs: build-version, artifact-path
    │
    ├── reusable-security-scan.yml    Security & code quality scan
    │       └── inputs: severity-threshold, fail-on-finding
    │
    └── reusable-deploy.yml      Deploy to environment
            └── inputs: environment (dev/staging/prod)
                secrets: DEPLOY_TOKEN
```

All reusable workflows live in **this repo** and can be called from any repo in the org!

---

## 💻 Demo: Reusable CI Workflow

```yaml
# .github/workflows/reusable-ci.yml
on:
  workflow_call:
    inputs:
      node-version:
        type: string
        default: "20"
      run-tests:
        type: boolean
        default: true
    outputs:
      build-version:
        value: ${{ jobs.build.outputs.version }}
      artifact-name:
        value: ${{ jobs.build.outputs.artifact }}
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get-version.outputs.version }}
      artifact: "app-build-${{ github.sha }}"
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      # ... build and test steps
```

---

## 💻 Demo: Orchestrator Calling All Workflows

```yaml
# .github/workflows/main-pipeline.yml
jobs:
  ci:
    uses: ./.github/workflows/reusable-ci.yml
    with:
      node-version: "20"
      run-tests: true
    secrets: inherit

  security:
    needs: [ci]
    uses: ./.github/workflows/reusable-security-scan.yml
    with:
      severity-threshold: "high"
      fail-on-finding: true
    secrets: inherit

  deploy-dev:
    needs: [ci, security]
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: "dev"
      artifact-name: ${{ needs.ci.outputs.artifact-name }}
    secrets: inherit
```

---

## ✅ Key Benefits — Recap

| Benefit | Description |
|---------|-------------|
| **DRY Pipelines** | Define once, use across unlimited repos |
| **Consistency** | All teams use the same, tested CI/CD logic |
| **Security** | Centralized secrets management, OIDC support |
| **Versioning** | Pin to tags/SHAs, test new versions safely |
| **Outputs** | Pass data between workflow layers |
| **Matrix** | Call reusable workflows with matrix strategy |
| **Scale** | Now: 10 nesting levels, 50 calls per run |

---

## 🔒 Security Highlight: OIDC + Reusable Workflows

Reusable workflows integrate with **OpenID Connect (OIDC)** for keyless authentication:

```yaml
# reusable-deploy.yml
jobs:
  deploy:
    permissions:
      id-token: write   # Required for OIDC
      contents: read
    steps:
      - name: Azure Login (OIDC)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
          # No long-lived credentials needed! 🔐
```

> Organizations can **require** specific reusable workflows for deployments via branch protection rules — ensuring every deployment goes through the approved security gates.

---

## 📝 Best Practices

1. **Use a dedicated shared-workflows repository** per organization
2. **Version your reusable workflows** with semantic tags (`v1`, `v1.2`, `v1.2.3`)
3. **Document inputs and outputs** — treat them as a public API
4. **Use `secrets: inherit` carefully** — only within the same org/owner
5. **Pin actions inside reusable workflows** to specific SHAs for security
6. **Test your reusable workflows** — use `workflow_dispatch` for manual testing
7. **Keep workflows focused** — one workflow per concern (build, test, deploy, scan)
8. **Use outputs instead of artifacts** for small data passing between workflows

---

## 🚀 Getting Started

**Three steps to your first reusable workflow:**

### Step 1: Create the reusable workflow
```yaml
# shared-workflows-repo/.github/workflows/my-reusable.yml
on:
  workflow_call:
    inputs:
      message: { type: string, required: true }
```

### Step 2: Call it from any workflow
```yaml
jobs:
  my-job:
    uses: my-org/shared-workflows/.github/workflows/my-reusable.yml@main
    with:
      message: "Hello from my app!"
```

### Step 3: Profit 🎉
Your entire organization now benefits from a single, maintained workflow.

---

## 📚 Resources

| Resource | Link |
|----------|------|
| **Official Docs** | [docs.github.com → Reusing Workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows) |
| **Nov 2025 Changelog** | [github.blog/changelog/2025-11-06-new-releases-for-github-actions-november-2025](https://github.blog/changelog/2025-11-06-new-releases-for-github-actions-november-2025/) |
| **This Demo Repo** | [github.com/seandorsett/github-actions-feature1](https://github.com/seandorsett/github-actions-feature1) |
| **Blog: Using Reusable Workflows** | [github.blog → using-reusable-workflows-github-actions](https://github.blog/developer-skills/github/using-reusable-workflows-github-actions/) |
| **Security Guide** | [docs.github.com → Security hardening](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions) |

---

## 🙋 Q&A

**Questions I commonly hear:**

- *"Can I call a reusable workflow from a different organization?"*
  → Only if the calling workflow is in a **public repo** or the reusable workflow is in a **public repo**

- *"Do environment-level secrets pass through automatically?"*
  → No — environment secrets must be explicitly mapped

- *"Can I use `matrix` with reusable workflows?"*
  → **Yes!** Use `strategy.matrix` in the job that calls the reusable workflow

- *"What's the difference between reusable workflows and composite actions?"*
  → Reusable workflows: multi-job orchestration. Composite actions: single-job step composition.

---

<!-- Final Slide -->

# Thank You! 🙌

## Let's automate together.

**Sean Dorsett** | GitHub Actions Office Hours

📁 Demo & slides: **github.com/seandorsett/github-actions-feature1**

---

*"The best CI/CD pipeline is one you only have to write once."*
