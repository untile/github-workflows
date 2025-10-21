# GitHub Workflows

Repository for **reusable GitHub Actions and workflows**.
Centralizes CI/CD pipelines, composite actions, and shared automation logic to keep projects consistent and maintainable.

---

## 📦 Structure

- `.github/workflows/` → Reusable workflows (CI, build, deploy, etc.)
- `actions/` → Composite actions (custom step groups)
- `documentation/` → Detailed documentation for workflows and their inputs/secrets

---

## 🚀 Available Workflows

### 1. [Mobile Pull Request](documentation/mobile-pull-request.md)
Reusable workflow to run the mobile project **test suite**, including:
- Node.js and Bun setup
- Dependencies installation
- Commit linting
- Lint checks

Usage example:

```yaml
jobs:
  test:
    uses: untile/github-workflows/.github/workflows/mobile-pull-request.yml@v1
    with:
      node_version: "20"
      bun_version: "latest"
      lint_commits: true
      commitlint_config: ".commitlintrc.cjs"
      cache_modules: true
```

---

### 2. [Mobile Publish App](documentation/mobile-publish-app.md)
Reusable workflow to **build and distribute** the iOS app with EAS:
- Builds with Expo EAS
- Optionally distributes to **Firebase App Distribution** and/or **TestFlight**

Usage example:

```yaml
jobs:
  release:
    uses: untile/github-workflows/.github/workflows/mobile-publish-app.yml@v1
    with:
      environment: preview
      publish_to_firebase: true
      publish_to_testflight: false
      node_version: "23"
      bun_version: "latest"
      firebase_groups: "internal-testers"
      release_notes: "CI build ${{ github.sha }}"
    secrets:
      EXPO_TOKEN: ${{ secrets.EXPO_TOKEN }}
      PREVIEW_FIREBASE_APP_ID_IOS: ${{ secrets.PREVIEW_FIREBASE_APP_ID_IOS }}
      PRODUCTION_FIREBASE_APP_ID_IOS: ${{ secrets.PRODUCTION_FIREBASE_APP_ID_IOS }}
      PREVIEW_FIREBASE_SERVICE_ACCOUNT_JSON: ${{ secrets.PREVIEW_FIREBASE_SERVICE_ACCOUNT_JSON }}
      PRODUCTION_FIREBASE_SERVICE_ACCOUNT_JSON: ${{ secrets.PRODUCTION_FIREBASE_SERVICE_ACCOUNT_JSON }}
      APP_STORE_CONNECT_API_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_API_ISSUER_ID }}
      APP_STORE_CONNECT_API_KEY_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_ID }}
      APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
```

---

### 3. Semgrep SAST

Reusable workflow for automated security scanning using [Semgrep](https://semgrep.dev):

- Runs configurable [Semgrep rule sets](https://semgrep.dev/explore)
- Generates PR comments with security findings
- Creates GitHub issues for critical vulnerabilities on master
- Supports custom scan paths and rule configurations

Usage example:

```yaml
jobs:
  semgrep:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@semgrep-sast
    with:
      path: packages/api
      semgrep-rules: p/owasp-top-ten p/nodejs p/typescript p/secrets
      skip-dependabot: false
      create-pr-comment: true
      create-issues: true
      fail-on-blocking-issues: true
```

---

## 🔖 Versioning

- Workflows and actions are referenced by **Git tags**.
- Use stable tags like `@v1` for compatibility, or `@v1.2.0` for a specific release.

---

## 🛠️ Development

1. Create or update workflows under `.github/workflows/`
2. Create or update composite actions under `actions/`
3. Create or update documentation under `documentation/`
4. Test locally in another repository by referencing a branch (e.g., `@main`)
5. When stable, create a new tag:

```bash
git tag v1.0.0
git push origin v1.0.0
```

---

## 📚 Resources

- [Reusable Workflows Docs](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Creating Composite Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

---
