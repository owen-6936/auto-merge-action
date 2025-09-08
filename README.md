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
    branches-ignore:
      - main
      - master

permissions:
  contents: write
  pull-requests: write

jobs:
  create-pr:
    runs-on: ubuntu-latest
    outputs:
      pr-branch: ${{ steps.extract-branch.outputs.branch }}
      pr-number: ${{ steps.get-pr.outputs.pr-number }}
    steps:
      - name: Extract Branch Name
        id: extract-branch
        run: echo "branch=${GITHUB_REF#refs/heads/}" >> $GITHUB_OUTPUT

      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Prettier
        run: npm install prettier

      - name: Format code with Prettier
        run: npx prettier --write .

      - name: Install GitHub CLI
        run: sudo apt-get install gh

      - name: Authenticate GitHub CLI
        run: echo "${{ secrets.GITHUB_TOKEN }}" | gh auth login --with-token

      - name: Check if PR already exists
        id: pr-check
        run: |
          existing_pr=$(gh pr list --head ${{ steps.extract-branch.outputs.branch }} --json number --jq '.[0].number')
          if [ -n "$existing_pr" ]; then
            echo "PR already exists: #$existing_pr"
            echo "skip=true" >> $GITHUB_OUTPUT
          else
            echo "No existing PR found"
            echo "skip=false" >> $GITHUB_OUTPUT
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Create Pull Request
        if: steps.pr-check.outputs.skip == 'false'
        run: |
          gh pr create \
            --title "chore: automated formatting" \
            --body "Automated PR from branch ${{ steps.extract-branch.outputs.branch }}" \
            --base main \
            --head ${{ steps.extract-branch.outputs.branch }} \
            --repo ${{ github.repository }}
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Get Pull Request Number
        id: get-pr
        run: |
          pr_number=$(gh pr list --head ${{ steps.extract-branch.outputs.branch }} --json number --jq '.[0].number')
          echo "pr-number=$pr_number" >> $GITHUB_OUTPUT
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

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
        run: echo "No linter configured" # Replace with: npm run lint

      - name: Run Tests
        run: echo "No tests configured" # Replace with: npm test

  auto-merge:
    runs-on: ubuntu-latest
    needs: [create-pr, run-checks]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Auto-Merge the Pull Request
        uses: ./.github/actions/auto-merge
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          pull-request-number: ${{ needs.create-pr.outputs.pr-number }}
````

> **Note:** For this to work, you must enable branch protection rules on your `main` branch and require the `codeql-scan` and `sonarqube-scan` status checks to pass before merging.

## 🛠️ Inputs

This action does not require any inputs. It uses the `GITHUB_TOKEN` from the workflow context to interact with the GitHub API.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](https://www.google.com/search?q=LICENSE) file for details.
