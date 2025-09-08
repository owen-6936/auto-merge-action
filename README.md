# Auto Merge

<div align="center">
  <img src="https://img.shields.io/github/v/release/owen-6936/auto-merge-action" alt="Latest Release">
  <img src="https://img.shields.io/github/actions/workflow/status/owen-6936/auto-merge-action/test-action.yml" alt="Build Status">
  <img src="https://img.shields.io/github/license/owen-6936/auto-merge-action" alt="License">
</div>

Automatically merges a pull request using the squash merge method when all required status checks have passed. This action is designed to be part of a robust CI/CD workflow, running only after your code has passed security scans and quality checks.

## 🚀 How to Use

To use this action, add a new job to your workflow file (e.g., `.github/workflows/your-workflow.yml`). The most important step is to ensure this job runs **after** your other jobs (like CodeQL and SonarQube) have completed successfully.

### Example Workflow

This example shows a full workflow that runs CodeQL and SonarQube first, and then uses the `auto-merge` action to merge the pull request only if those jobs pass.

```yaml
name: Automatic Pull Request

on:
  push:
    branches:
      - '*'
      - '!main'
      - '!master'

permissions:
  contents: write
  pull-requests: write

jobs:
  create-pr:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Create Pull Request
        id: create-pr-step
        uses: peter-evans/create-pull-request@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          title: ${{ github.event.head_commit.message }}
          body: ${{ github.event.head_commit.message }}
          branch: ${{ github.ref_name }}
          base: 'main'
          add-paths: '*'
          
  run-checks:
    runs-on: ubuntu-latest
    needs: create-pr
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run Linter
        run: npx eslint .

      - name: Run Tests
        run: npm run test
        
  auto-merge:
    runs-on: ubuntu-latest
    needs: [create-pr, run-checks]
    steps:
      - name: Auto-Merge the Pull Request
        uses: owen-6936/auto-merge-action@v1.0.0
```

> **Note:** For this to work, you must enable branch protection rules on your `main` branch and require the `codeql-scan` and `sonarqube-scan` status checks to pass before merging.

## 🛠️ Inputs

This action does not require any inputs. It uses the `GITHUB_TOKEN` from the workflow context to interact with the GitHub API.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](https://www.google.com/search?q=LICENSE) file for details.

## 🤝 Contributing

Contributions are welcome! Please open an issue or submit a pull request for any improvements or bug fixes.

## ❓ Questions?

If you have any questions or need help, feel free to open an issue in the repository.
