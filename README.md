# CI/CD Setup for Tests Repository

## ⚙️ Workflow Triggers

### Local Triggers
- Each workflow runs when files inside its project folder change:
  - `push` events to `main`
  - `pull_request` events targeting `main`
- Example: `merchant-portal-ui-tests.yml` runs only if files under `merchant-portal-ui-tests/**` are modified.

### External Triggers
- Workflows also listen for `repository_dispatch` events of type `run-tests`.
- These events are sent from the **code repo** when commits are made.
- Payloads specify which project’s tests should run.

---

## 📝 Example Workflow: Merchant Portal UI Tests

```yaml
name: Merchant Portal UI Tests CI

on:
  push:
    branches: [ main ]
    paths:
      - 'merchant-portal-ui-tests/**'
  pull_request:
    branches: [ main ]
    paths:
      - 'merchant-portal-ui-tests/**'
  repository_dispatch:
    types: [run-tests]

jobs:
  merchant-portal-ui-tests:
    if: github.event_name != 'repository_dispatch' || github.event.client_payload.project == 'merchant-portal-ui-tests'
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: merchant-portal-ui-tests
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright Browsers
        run: npx playwright install --with-deps

      - name: Run tests
        run: npm test

      - name: Upload test results
        uses: actions/upload-artifact@v4
        with:
          name: merchant-portal-results
          path: merchant-portal-ui-tests/test-results
```
##🏗️ Code Repo Trigger Workflow
### In the code repo, add a workflow that dispatches events to this repo:

```yaml
name: Trigger Tests Repo

on:
  push:
    branches: [ main ]

jobs:
  trigger-tests:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Merchant Portal Tests
        uses: peter-evans/repository-dispatch@v3
        with:
          token: ${{ secrets.REPO_B_TOKEN }}
          repository: your-org/tests-repo
          event-type: run-tests
          client-payload: '{"project":"merchant-portal-ui-tests"}'
```
