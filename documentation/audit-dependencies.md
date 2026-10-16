# Dependencies Security Audit Workflow

This workflow runs **security audits** on project dependencies using npm or bun.  
It reports vulnerabilities, comments on PRs, and can block merges on critical issues.

---

## 🔧 Inputs

### Required Inputs
None - all inputs have sensible defaults.

### Optional Inputs

- **node_version** (string, default `"22"`)  
  Node.js version to install (e.g., `"20"`, `"20.18.0"`, `"22"`).

- **package_manager** (string, default `"npm"`)  
  Package manager to use. Options: `npm`, `bun`.

- **npm_version** (string, default `""`)
  npm version to use (e.g. "11", "11.2.0"). If empty, uses the version bundled with Node.js.

- **audit_level** (string, default `"moderate"`)  
  Minimum severity level to report. Options: `low`, `moderate`, `high`, `critical`.

- **block_on_critical** (boolean, default `true`)  
  Fail the workflow if critical vulnerabilities are found.

- **block_on_high** (boolean, default `false`)  
  Fail the workflow if high severity vulnerabilities are found.

- **create_pr_comment** (boolean, default `true`)  
  Post detailed audit results as a PR comment.

- **skip_dependabot** (boolean, default `true`)  
  Skip this workflow for PRs created by Dependabot to avoid noise.

- **runs_on** (string, default `""`)  
  Runner label for the job. When empty, uses the `RUNNER` Actions variable (set at org level; a repository variable with the same name overrides it), falling back to `ubuntu-latest`. Set it to pin this workflow to a specific runner, e.g. `ubuntu-latest` or `untile-large`.

---

## 🔑 Secrets

This workflow **does not require any secrets**.

---

## 📌 Example Usage

### Basic Usage (NPM with defaults)
```yaml
jobs:
  audit:
    uses: untile/github-workflows/.github/workflows/audit-dependencies.yml@v1
```

### Custom Configuration (Bun project)
```yaml
jobs:
  audit:
    uses: untile/github-workflows/.github/workflows/audit-dependencies.yml@v1
    with:
      node_version: "20"
      package_manager: "bun"
      audit_level: "high"
      block_on_critical: true
      block_on_high: false
```

### Strict Mode (Block on High + Critical)
```yaml
jobs:
  audit:
    uses: untile/github-workflows/.github/workflows/audit-dependencies.yml@v1
    with:
      audit_level: "high"
      block_on_critical: true
      block_on_high: true
```

### Scheduled Security Scan
```yaml
# In your repository's .github/workflows/security-scan.yml
name: Weekly Security Scan

on:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday at 9 AM
  workflow_dispatch:

jobs:
  audit:
    uses: untile/github-workflows/.github/workflows/audit-dependencies.yml@v1
```

### Pull Request Trigger
```yaml
# In your repository's .github/workflows/pull-request.yml
name: Pull Request Checks

on:
  pull_request:
    paths:
      - 'package.json'
      - 'package-lock.json'
      - 'bun.lockb'

jobs:
  audit:
    uses: untile/github-workflows/.github/workflows/audit-dependencies.yml@v1
    with:
      package_manager: "npm"
      block_on_critical: true
```

---

## 📊 What This Workflow Does

1. ✅ Checks out code
2. ✅ Sets up Node.js and package manager (npm/bun/yarn)
3. ✅ Installs dependencies with frozen lockfile
4. ✅ Runs security audit with specified audit level
5. ✅ Parses vulnerability counts by severity (critical/high/moderate/low)
6. ✅ Uploads audit results as GitHub artifacts
7. ✅ (PR only) Posts formatted comment with:
   - Vulnerability summary table
   - Detailed findings (collapsible, truncated to 5000 chars)
   - Remediation steps (package manager specific)
   - Fix commands and tips
8. ✅ (Optional) Fails workflow if blocking vulnerabilities found
9. ✅ (Schedule only) Creates/updates GitHub issue with:
   - Vulnerability summary
   - Action required steps
   - Links to workflow run
   - Appropriate labels (security, dependencies, security-critical)

---

## 🎨 PR Comment Example

The workflow generates rich PR comments:

**With vulnerabilities**:
```
## 📦 Dependency Security Audit

⚠️ **Found 5 vulnerabilities**

| Severity | Count |
|----------|-------|
| 🔴 Critical | 1 |
| 🟠 High | 2 |
| 🟡 Moderate | 2 |
| ⚪ Low | 0 |

<details>
<summary>📋 View Details</summary>

[truncated audit output]
</details>

### 🔧 Remediation

**Critical vulnerabilities require immediate attention:**
1. Review findings above
2. Run `npm audit fix` to auto-fix if possible
3. Manually update packages if auto-fix unavailable

⚠️ **This PR may be blocked until Critical vulnerabilities are resolved.**

---
💡 **Tip**: Run `npm audit` locally to see detailed vulnerability information.
```

**No vulnerabilities**:
```
## 📦 Dependency Security Audit

✅ **No vulnerabilities found!** All dependencies are secure.

---
💡 **Tip**: Run `npm audit` locally to see detailed vulnerability information.
```

---

## 📋 GitHub Issue Example (Schedule Trigger)

When run on schedule (e.g., weekly), the workflow creates/updates issues:

```
[Security] 1 Critical and 3 High dependency vulnerabilities found

## 📦 Dependency Security Scan

**Scan Date**: 2025-10-22
**Package Manager**: npm

### Summary

- 🔴 Critical: 1
- 🟠 High: 3
- 🟡 Moderate: 2
- ⚪ Low: 0

### Action Required

1. Review [audit results](https://github.com/org/repo/actions/runs/123456)
2. Run `npm audit` locally for details
3. Fix vulnerabilities:
   ```bash
   npm audit fix
   npm audit fix --force  # if needed
   ```
4. Update dependencies that require manual intervention
```

---

## 🚨 Blocking Behavior

- If `block_on_critical: true` → workflow **fails** when critical vulnerabilities found
- If `block_on_high: true` → workflow **fails** when high vulnerabilities found
- Failed workflows **prevent PR merges** (if configured as required check in branch protection)

**Exit behavior**:
- Prints clear error message
- Shows vulnerability counts
- Provides fix instructions

---

## 📦 Package Manager Support

| Package Manager | Audit Command | Lockfile | Notes |
|----------------|---------------|----------|-------|
| npm | `npm audit` | package-lock.json | Full support |
| bun | `bun audit` | bun.lockb | Experimental; falls back to npm if needed |

---

## 💡 Best Practices

1. **Trigger on dependency changes**:
   ```yaml
   on:
     pull_request:
       paths:
         - 'package.json'
         - 'package-lock.json'  # or bun.lockb
   ```

2. **Set appropriate blocking levels**:
   - Always use `block_on_critical: true`
   - Consider `block_on_high: true` for production apps

3. **Run scheduled scans**:
   ```yaml
   on:
     schedule:
       - cron: '0 9 * * 1'  # Weekly on Monday
   ```

4. **Test fixes**: Always test after running `audit fix` to ensure no breaking changes

5. **Review artifacts**: Detailed audit results are always available as workflow artifacts for debugging

---

## 🔗 Complete Integration Example

```yaml
# .github/workflows/pull-request.yml
name: Pull Request Checks

on:
  pull_request:
    paths:
      - '**.js'
      - '**.ts'
      - 'package.json'
      - 'package-lock.json'

jobs:
  # Security audit on dependency changes
  security-audit:
    uses: untile/github-workflows/.github/workflows/audit-dependencies.yml@v1
    with:
      package_manager: "npm"
      block_on_critical: true

  # Linting and tests
  test:
    needs: security-audit
    uses: untile/github-workflows/.github/workflows/mobile-pull-request.yml@v1
    with:
      node_version: "20"
      bun_version: "latest"

  # SAST security scan
  semgrep:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
    with:
      path: "src"
      semgrep-rules: "p/owasp-top-ten p/nodejs p/secrets"
```

```yaml
# .github/workflows/security-scan.yml
name: Weekly Security Scan

on:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday at 9 AM UTC
  workflow_dispatch:

jobs:
  dependency-audit:
    uses: untile/github-workflows/.github/workflows/audit-dependencies.yml@v1
  
  sast-scan:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
    with:
      create-issues: true
```

---

## 📚 Resources

- [npm audit documentation](https://docs.npmjs.com/cli/v9/commands/npm-audit)
- [Bun audit](https://bun.sh/docs/cli/audit)
- [pnpm audit](https://pnpm.io/cli/audit)
- [Yarn audit](https://classic.yarnpkg.com/en/docs/cli/audit/)
- [GitHub Security Best Practices](https://docs.github.com/en/code-security)

---

## 🔄 Migration Guide

**Before** (standalone workflow in your repo):
```yaml
# ~200 lines of workflow code
name: Dependencies Audit
on:
  pull_request:
jobs:
  npm-audit:
    runs-on: ubuntu-latest
    steps:
      # ... many steps
```

**After** (reusable workflow):
```yaml
# ~10 lines
name: Dependencies Audit
on:
  pull_request:
    paths:
      - 'package.json'
      - 'package-lock.json'
jobs:
  audit:
    uses: untile/github-workflows/.github/workflows/audit-dependencies.yml@v1
    with:
      node_version: "22"
      block_on_critical: true
```

**Benefits**:
✅ Reduced from ~200 lines to ~10 lines  
✅ Centralized maintenance  
✅ Automatic updates with version tags  
✅ Consistent behavior across all projects  
