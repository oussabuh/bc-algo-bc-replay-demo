# 🧠 Teaching AL-Go to Test Itself: How the New Page Scripting Tool (bc-replay) Makes Business Central CI/CD Revolutionary

This guide walks you step-by-step through integrating **bc-replay** into your **AL-Go for GitHub** pipeline — so your Business Central CI/CD setup can actually *test itself* by replaying real UI interactions.

---

## 🚀 Goal

Automate Business Central UI tests directly in your AL-Go CI/CD pipeline using the new **Page Scripting Tool** (`bc-replay`).

---

## 📋 Prerequisites

Before you start, ensure the following:

| Requirement | Description |
|--------------|-------------|
| **Business Central Environment** | Sandbox environment with Page Scripting enabled |
| **Service User** | Dedicated user without MFA, assigned *SUPER* in sandbox |
| **Node.js LTS** | v20+ |
| **AL-Go Template** | Repo created from [AL-Go for GitHub](https://github.com/microsoft/AL-Go) |
| **bc-replay** | Installed locally for recording scripts |
| **GitHub Actions Secrets** | `BC_URL`, `BC_USER`, `BC_PASSWORD` added in Repo → Settings → Secrets → Actions |

---

## 🏗️ Step 1 — Plan & Bootstrap the Repo

1. Go to the [AL-Go for GitHub template](https://github.com/microsoft/AL-Go).
2. Click **Use this template** → **Create a new repository**.
3. Clone your new repo locally.
4. Confirm that `.github/workflows` contains the default AL-Go CI/CD YAMLs.

---

## ⚙️ Step 2 — Local Tooling Setup

In PowerShell:

```powershell
node -v
npm -v
npm install -g @microsoft/bc-replay
```

Confirm the installation:

```powershell
npx replay --help
```

---

## 🧑‍💻 Step 3 — Prepare Your Business Central Sandbox

1. Log in to your **BC Admin Center** → Create a sandbox environment.  
2. Create a **service user** (no MFA) with role *SUPER*.  
3. Log in once with this user in a browser to activate it.  
4. Enable Page Scripting under **Feature Management** (if needed).


---

## 🎥 Step 4 — Record a Page Script (UI Test)

1. Open BC as your test user.  
2. Start **Page Scripting** (🧩 icon in top-right).  
3. Record a realistic process (example: Create & Post a Sales Invoice).  
4. Stop recording and **Export YAML**.  
5. Save it to your repo under:

```
/tests/pageScripts/post-sales-invoice.yml
```

---

## 🔄 Step 5 — Wire Up CI/CD (AL-Go + bc-replay)

Create a new workflow `.github/workflows/ui-scripts.yml`:

```yaml
name: Run page scripts (post AL-Go CI)

on:
  workflow_run:
    workflows: [" CI/CD"]
    types: [completed]
  workflow_dispatch:

jobs:
  replay-ui:
    if: ${{ github.event_name == 'workflow_dispatch' || github.event.workflow_run.conclusion == 'success' }}
    runs-on: windows-latest
    defaults:
      run:
        shell: pwsh

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install bc-replay
        run: npm install @microsoft/bc-replay --save

      - name: Install Playwright browsers
        run: npx playwright install

      - name: Prepare results folder
        run: New-Item -ItemType Directory -Path ".\tests\pageScripts\results" -Force

      - name: Run page script (AAD auth)
        env:
          BC_URL: ${{ secrets.BC_URL }}
          BC_USER: ${{ secrets.BC_USER }}
          BC_PASSWORD: ${{ secrets.BC_PASSWORD }}
        run: >
          npx --yes --package @microsoft/bc-replay replay
          -Tests ".\tests\pageScripts\post-sales-invoice.yml"
          -StartAddress "$env:BC_URL"
          -Authentication AAD
          -UserNameKey BC_USER
          -PasswordKey BC_PASSWORD
          -ResultDir ".\tests\pageScripts\results"

      - name: Upload Playwright report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: tests/pageScripts/results/playwright-report
```

---

## ▶️ Step 6 — Run and Validate

### Run manually
In GitHub → Actions → “Run page scripts (post AL-Go CI)” → **Run workflow**.

### Run automatically
Push any commit to trigger your main **“ CI/CD”** workflow.  
Once it completes, your new workflow runs automatically.

✅ *Expected result:*  
- bc-replay launches Chrome  
- Runs the test  
- Closes and uploads a `playwright-report` artifact  
- GitHub marks the run as **success**


## 🧩 Appendix — Folder Layout

```
.github/
  workflows/
    CI/CD.yml
    ui-scripts.yml
tests/
  pageScripts/
    post-sales-invoice.yml
    results/
```

---

## ✅ Final Outcome

You now have:
- An AL-Go pipeline that automatically runs **UI tests** after each build.
- Replay-based Business Central verification.
- Artifacts with UI replay reports — all within GitHub Actions.

---

### 💡 Credits
Built with:
- [@microsoft/bc-replay](https://www.npmjs.com/package/@microsoft/bc-replay)
- [AL-Go for GitHub](https://github.com/microsoft/AL-Go)
- [Dynamics 365 Business Central](https://learn.microsoft.com/en-us/dynamics365/business-central/)

---

> © 2025 — Authored by [Oussama Sabbouh]  
