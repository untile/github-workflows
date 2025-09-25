# Mobile Pull Request Workflow

This workflow runs the **mobile project test suite**.  
It sets up Node.js and Bun, installs dependencies, lints commits, and runs lint checks.

---

## 🔧 Inputs

- **node_version** (string, default `"20"`)  
  Node.js version to install (e.g., `"20"`, `"20.18.0"`).

- **bun_version** (string, default `"latest"`)  
  Bun version to install.

- **lint_commits** (boolean, default `true`)  
  Whether to run commit linting with commitlint.

- **commitlint_config** (string, default `".commitlintrc.cjs"`)  
  Path to the commitlint configuration file.

- **cache_modules** (boolean, default `true`)  
  Enable caching of `node_modules` and `bun.lockb`.

---

## 🛠️ Secrets

This workflow **does not require any secrets**.

---

## 📌 Example Usage

```yaml
jobs:
  test:
    uses: untile/github-workflows/.github/workflows/mobile-pull-request.yml@v1
    with:
      node_version: "20.18.0"
      bun_version: "latest"
      lint_commits: true
      commitlint_config: ".commitlintrc.cjs"
      cache_modules: true
```
