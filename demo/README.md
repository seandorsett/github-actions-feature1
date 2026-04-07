# GitHub Actions: Reusable Workflows Demo

This directory contains a working demo showcasing **GitHub Actions Reusable Workflows** — the recently enhanced feature (November 2025) with increased limits:

- **10 nesting levels** (up from 4)
- **50 workflow calls per run** (up from 20)

---

## 📁 Demo Structure

```
.github/
└── workflows/
    ├── reusable-ci.yml            # Reusable: Build & Test workflow
    ├── reusable-security-scan.yml # Reusable: Security scanning workflow
    ├── reusable-deploy.yml        # Reusable: Deployment workflow
    └── main-pipeline.yml          # Orchestrator: calls all 3 reusable workflows
```

---

## 🔄 Workflow Architecture

```
main-pipeline.yml  (Orchestrator)
    │
    ├── [Job: ci]
    │   └── uses: reusable-ci.yml
    │         inputs:  node-version, run-tests
    │         outputs: build-version, artifact-name
    │
    ├── [Job: security] ← needs: [ci]
    │   └── uses: reusable-security-scan.yml
    │         inputs:  artifact-name, severity-threshold, fail-on-finding
    │
    └── [Job: deploy-dev] ← needs: [ci, security]
        └── uses: reusable-deploy.yml
              inputs:  environment, artifact-name, build-version
              secrets: DEPLOY_TOKEN (via secrets: inherit)
```

---

## 🚀 How to Run the Demo

### Trigger the pipeline manually

1. Navigate to the **Actions** tab in this repository
2. Select **"Demo: Main Pipeline (Orchestrator)"** from the left sidebar
3. Click **"Run workflow"**
4. Fill in the parameters:
   - **Node.js version**: `20` (or `18`, `21`)
   - **Run tests**: `true`
   - **Deploy environment**: `dev` (or `staging`, `prod`)
5. Click the green **"Run workflow"** button

### What you'll see

- The orchestrator workflow starts
- It fans out to call all three reusable workflows
- Each reusable workflow runs as its own job
- The deploy job waits for CI and security to pass first (`needs`)
- Outputs flow from CI → deploy (the artifact name is passed through)

---

## 📋 Key Concepts Demonstrated

### 1. `workflow_call` Trigger

Each reusable workflow uses `workflow_call` instead of (or in addition to) `push`/`pull_request`:

```yaml
on:
  workflow_call:
    inputs:
      node-version:
        type: string
        required: true
    outputs:
      build-version:
        description: "Semantic version of the build"
        value: ${{ jobs.build.outputs.version }}
    secrets:
      DEPLOY_TOKEN:
        required: false
```

### 2. Calling with `uses` + `with` + `secrets`

```yaml
jobs:
  run-ci:
    uses: ./.github/workflows/reusable-ci.yml
    with:
      node-version: "20"
      run-tests: true
    secrets: inherit   # passes all secrets from caller
```

### 3. Consuming Outputs

```yaml
jobs:
  deploy:
    needs: [ci]
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      artifact-name: ${{ needs.ci.outputs.artifact-name }}
      build-version: ${{ needs.ci.outputs.build-version }}
```

### 4. `secrets: inherit`

When reusable workflows are called within the same organization/owner, you can use `secrets: inherit` to automatically forward all secrets from the calling workflow to the called workflow — no need to enumerate them individually.

---

## 🧪 Try These Scenarios

| Scenario | What to observe |
|----------|-----------------|
| Run with `run-tests: false` | The test step is skipped in `reusable-ci.yml` |
| Run with `fail-on-finding: false` | Security scan reports issues but doesn't fail the pipeline |
| Run with `environment: prod` | Deploy job requires manual approval (environment protection rule) |
| Run with invalid `node-version` | CI fails early with a clear error message |

---

## 📚 Further Reading

- [GitHub Docs: Reusing Workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows)
- [Nov 2025 Changelog: Increased Reusable Workflow Limits](https://github.blog/changelog/2025-11-06-new-releases-for-github-actions-november-2025/)
- [GitHub Blog: Using Reusable Workflows](https://github.blog/developer-skills/github/using-reusable-workflows-github-actions/)
- [Security Hardening for GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
