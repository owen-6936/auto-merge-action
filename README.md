# Auto Merge

<div align="center">
  <img src="https://img.shields.io/github/v/release/owen-6936/auto-merge-action" alt="Latest Release">
  <img src="https://img.shields.io/github/actions/workflow/status/owen-6936/auto-merge-action/test-workflow.yml" alt="Build Status">
  <img src="https://img.shields.io/github/license/owen-6936/auto-merge-action" alt="License">
</div>

Automatically merges a pull request using the squash merge method when all required status checks have passed. This action is designed to be part of a robust CI/CD workflow, running only after your code has passed security scans and quality checks.

## 🚀 How to Use

To use this action, add a new job to your workflow file (e.g., `.github/workflows/your-workflow.yml`). The most important step is to ensure this job runs **after** your other jobs (like CodeQL and SonarQube) have completed successfully.

### Example Workflow

This example shows a full workflow that runs CodeQL and SonarQube first, and then uses the `auto-merge` action to merge the pull request only if those jobs pass.

```yaml
name: CI/CD Pipeline

on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches: ["main"]

jobs:
  codeql-scan:
    # Your existing CodeQL job configuration
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with:
          languages: javascript
      - uses: github/codeql-action/analyze@v3

  sonarqube-scan:
    # Your existing SonarQube job configuration
    needs: [codeql-scan]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@7295e71c9583053f5bf40e9d4068a0c974603ec8
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

  auto-merge:
    name: Auto Merge PR
    needs: [codeql-scan, sonarqube-scan]
    runs-on: ubuntu-latest
    if: success()
    steps:
      - name: Auto Merge
        uses: owen-6936/auto-merge-action@v1.0.0
````

> **Note:** For this to work, you must enable branch protection rules on your `main` branch and require the `codeql-scan` and `sonarqube-scan` status checks to pass before merging.

## 🛠️ Inputs

This action does not require any inputs. It uses the `GITHUB_TOKEN` from the workflow context to interact with the GitHub API.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](https://www.google.com/search?q=LICENSE) file for details.

## 🤝 Contributing

Contributions are welcome! Please open an issue or submit a pull request for any improvements or bug fixes.

## ❓ Questions?

If you have any questions or need help, feel free to open an issue in the repository.
