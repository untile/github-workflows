# GitHub Workflows

Repository for **reusable GitHub Actions and workflows**.
Centralizes CI/CD pipelines, composite actions, and shared automation logic to keep projects consistent and maintainable.

---

## 📦 Structure

- `.github/workflows/` → Reusable workflows (CI, build, deploy, etc.)
- `actions/` → Composite actions (custom step groups)

---

## 🚀 Usage

### Using a reusable workflow

In your project’s `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    uses: untile/github-workflows/.github/workflows/test.yml@v1
    with:
      node_version: "20"
      run_tests: true
    secrets: inherit
```

### Using a composite action

In your workflow:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: untile/github-workflows/actions/setup-node-npm@v1
      - run: npm test
```

---

## 🔖 Versioning

- Workflows and actions are referenced by **Git tags**.
- Use stable tags like `@v1` for compatibility, or `@v1.2.0` for a specific release.

---

## 🛠️ Development

1. Create or update workflows under `.github/workflows/`
2. Create or update composite actions under `actions/`
3. Test locally in another repository by referencing a branch (e.g., `@main`)
4. When stable, create a new tag:

```bash
git tag v1.0.0
git push origin v1.0.0
```

---

## 📚 Resources

- [Reusable Workflows Docs](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Creating Composite Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

---

## 📄 License

MIT License
