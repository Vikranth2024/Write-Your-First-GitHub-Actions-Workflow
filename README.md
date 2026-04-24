# 🚀 TaskPulse — Solution (Write Your First GitHub Actions Workflow)

> ⚠️ **This is the SOLUTION repo.** For the skeleton, see `lu-8.8-GitHub-Actions-Workflow`.

---

## Complete Workflow

The completed `.github/workflows/deploy.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Deploy to Render
        run: curl --fail -X POST ${{ secrets.RENDER_DEPLOY_HOOK_URL }}
```

---

## Step-by-Step Breakdown

### TODO 1 — Trigger

```yaml
on:
  push:
    branches:
      - main
```

**What it does**: Runs the workflow every time code is pushed to the `main` branch. Pushes to other branches (feature branches, PRs) won't trigger it.

### TODO 2 — Checkout Code

```yaml
- name: Checkout code
  uses: actions/checkout@v4
```

**What it does**: Clones the repository into the GitHub Actions runner (a fresh Ubuntu VM). Without this, the runner has no access to your code files.

### TODO 3 — Setup Node.js

```yaml
- name: Set up Node.js
  uses: actions/setup-node@v4
  with:
    node-version: "20"
```

**What it does**: Installs Node.js v20 on the runner. The `with:` block passes inputs to the action — in this case, which Node version to install.

### TODO 4 — Install Dependencies

```yaml
- name: Install dependencies
  run: npm ci
```

**Why `npm ci` and not `npm install`?**

| | `npm install` | `npm ci` |
|---|---|---|
| Speed | Slower (resolves versions) | Faster (uses lockfile) |
| Reproducible | No (may get different versions) | Yes (exact lockfile match) |
| node_modules | Updates in place | Deletes and recreates |
| Best for | Local development | CI/CD pipelines |

### TODO 5 — Run Tests

```yaml
- name: Run tests
  run: npm test
```

**The safety gate**: If any test fails, `npm test` returns a non-zero exit code, the step fails, and **all subsequent steps are skipped**. The deploy never happens. This ensures only code that passes tests reaches production.

### TODO 6 — Deploy to Render

```yaml
- name: Deploy to Render
  run: curl --fail -X POST ${{ secrets.RENDER_DEPLOY_HOOK_URL }}
```

**Key details:**
- `--fail`: Makes curl return exit code 22 on HTTP errors (4xx/5xx). Without it, curl returns 0 even on errors.
- `-X POST`: Sends a POST request to trigger the deploy hook.
- `${{ secrets.RENDER_DEPLOY_HOOK_URL }}`: Reads the URL from GitHub Secrets — **never hardcode** deploy hooks in YAML.

---

## Setting Up the Secret

1. Get the deploy hook URL from Render: **Dashboard → Service → Settings → Deploy Hook**
2. Go to GitHub: **Repo → Settings → Secrets and variables → Actions → New repository secret**
3. Name: `RENDER_DEPLOY_HOOK_URL`
4. Value: The URL you copied from Render

---

## What Happens When Tests Fail?

```
✓ Checkout code        (passed)
✓ Set up Node.js       (passed)
✓ Install dependencies (passed)
✗ Run tests            (FAILED — exit code 1)
⊘ Deploy to Render     (SKIPPED — previous step failed)
```

The deploy step **never runs**. This is the entire point of CI/CD — broken code never reaches production.

---

## Rubric Mapping

| Points | Criteria | How Satisfied |
|--------|----------|---------------|
| 3 | All 5 steps in correct order | ✅ checkout → setup-node → npm ci → npm test → curl |
| 3 | Green workflow run | ✅ Passes on push to main, triggers Render deploy |
| 2 | Secrets used correctly | ✅ `secrets.RENDER_DEPLOY_HOOK_URL`, not hardcoded |
| 2 | Video explanation | ✅ Each step explained + test failure demo |
| **10** | **Total** | |
