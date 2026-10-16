# Semgrep SAST Workflow

This workflow runs **automated security scanning** using [Semgrep](https://semgrep.dev), a fast and customizable static analysis tool.  
It detects vulnerabilities, comments on PRs, creates issues, and can block merges on critical findings.

---

## 🔧 Inputs

### Required Inputs
None - all inputs have sensible defaults.

### Optional Inputs

- **path** (string, default `"."`)  
  Path where Semgrep checks will run (relative to repository root).

- **skip-dependabot** (boolean, default `false`)  
  Skip this workflow for PRs created by Dependabot.

- **semgrep-rules** (string, default `"p/owasp-top-ten p/secrets"`)  
  Space-separated list of Semgrep rule sets to run. Examples:
  - `p/owasp-top-ten` - OWASP Top 10 vulnerabilities
  - `p/secrets` - Hardcoded secrets detection
  - `p/nodejs` - Node.js security rules
  - `p/typescript` - TypeScript best practices
  - `p/react` - React security patterns
  - `p/sql-injection` - SQL injection detection

- **create-pr-comment** (boolean, default `true`)  
  Post detailed scan results as a PR comment.

- **create-issues** (boolean, default `true`)  
  Create GitHub issues for ERROR-level findings on master branch pushes.

- **fail-on-blocking-issues** (boolean, default `true`)  
  Fail the workflow if ERROR-level security issues are found.

- **runs-on** (string, default `""`)  
  Runner label for the job. When empty, uses the `RUNNER` Actions variable (set at org level; a repository variable with the same name overrides it), falling back to `ubuntu-latest`. Set it to pin this workflow to a specific runner, e.g. `ubuntu-latest` or `untile-large`.

---

## 🔑 Secrets

This workflow **does not require any secrets**.

---

## 📌 Example Usage

### Basic Usage (Default Configuration)
```yaml
jobs:
  security-scan:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
```

### Custom Rule Sets (Node.js + TypeScript Project)
```yaml
jobs:
  security-scan:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
    with:
      semgrep-rules: "p/owasp-top-ten p/nodejs p/typescript p/secrets p/react"
      fail-on-blocking-issues: true
```

### Scan Specific Directory
```yaml
jobs:
  api-security:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
    with:
      path: "packages/api"
      semgrep-rules: "p/owasp-top-ten p/nodejs p/sql-injection"
```

### Non-Blocking Scan (Warnings Only)
```yaml
jobs:
  security-scan:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
    with:
      fail-on-blocking-issues: false
      create-pr-comment: true
```

### Custom Semgrep Config File
```yaml
# If you have a .semgrep.yml file in your repository root,
# it will be automatically included alongside specified rules
jobs:
  security-scan:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
    with:
      semgrep-rules: "p/owasp-top-ten p/secrets"
```

### Pull Request Trigger
```yaml
# In your repository's .github/workflows/pull-request.yml
name: Pull Request Security Scan

on:
  pull_request:
    branches:
      - master
      - main

jobs:
  semgrep:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
    with:
      semgrep-rules: "p/owasp-top-ten p/nodejs p/secrets"
      fail-on-blocking-issues: true
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
  semgrep:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
    with:
      create-issues: true
      fail-on-blocking-issues: false
```

---

## 📊 What This Workflow Does

1. ✅ Checks out code with full history
2. ✅ Installs Semgrep via pip
3. ✅ Runs security scan with configured rule sets
4. ✅ Checks for custom `.semgrep.yml` config file
5. ✅ Generates JSON results with findings
6. ✅ (PR only) Posts formatted comment with:
   - Summary of errors, warnings, and info findings
   - Top 5 critical issues with file locations
   - Links to learn more about vulnerabilities
7. ✅ (Optional) Fails workflow if ERROR-level issues found
8. ✅ (Master only) Creates GitHub issues for ERROR-level findings:
   - One issue per security finding
   - Includes CWE and OWASP classifications
   - Deduplicates existing open issues
   - Labels: `security`, `semgrep`

---

## 🎨 PR Comment Example

**With security findings**:
```markdown
## 🔒 Semgrep Security Scan Results

- ❌ **Errors**: 2
- ⚠️ **Warnings**: 5
- ℹ️ **Info**: 3

### Critical Issues (Must Fix)

**javascript.express.security.audit.express-check-csurf-middleware-usage.express-check-csurf-middleware-usage**
- File: `src/routes/auth.ts:45`
- Detected use of CSRF vulnerable endpoint without protection
- [Learn more](https://semgrep.dev/r/javascript.express.security.audit.express-check-csurf-middleware-usage)

**typescript.react.security.audit.react-dangerouslysetinnerhtml.react-dangerouslysetinnerhtml**
- File: `src/components/UserProfile.tsx:120`
- Dangerous use of dangerouslySetInnerHTML can lead to XSS
- [Learn more](https://semgrep.dev/r/typescript.react.security.audit.react-dangerouslysetinnerhtml)

_...and 0 more errors. Check the workflow logs for details._
```

**No security issues**:
```markdown
✅ **Semgrep Security Scan**: No new security issues found!
```

---

## 📋 GitHub Issue Example (Master Branch)

When ERROR-level findings are detected on master, the workflow creates issues:

```markdown
[Security] javascript.lang.security.audit.crypto.hardcoded-key in src/config/encryption.ts

## Security Finding

**Rule**: `javascript.lang.security.audit.crypto.hardcoded-key`
**Severity**: ERROR ❌
**File**: `src/config/encryption.ts:12`

### Description
Hardcoded encryption key detected. Keys should be stored in environment variables or secure vaults.

**CWE**: CWE-798
**OWASP**: A02:2021 - Cryptographic Failures

**References**:
- https://owasp.org/Top10/A02_2021-Cryptographic_Failures/
- https://cwe.mitre.org/data/definitions/798.html

### Code
```javascript
const encryptionKey = "hardcoded-secret-key-123";
```

---
_Automatically created by Semgrep security scan_
```

**Labels**: `security`, `semgrep`

---

## 🚨 Blocking Behavior

- If `fail-on-blocking-issues: true` → workflow **fails** when ERROR-level findings detected
- Failed workflows **prevent PR merges** (if configured as required check in branch protection)

**Exit behavior**:
- Prints clear error message
- References findings in PR comment
- Directs to workflow logs for details

---

## 🔍 Available Semgrep Rule Sets

| Rule Set | Description | Use Case |
|----------|-------------|----------|
| `p/owasp-top-ten` | OWASP Top 10 vulnerabilities | All web applications |
| `p/secrets` | Hardcoded secrets, API keys, tokens | All projects |
| `p/nodejs` | Node.js security patterns | Node.js backends |
| `p/typescript` | TypeScript best practices | TypeScript projects |
| `p/javascript` | JavaScript security rules | JS/TS projects |
| `p/react` | React security patterns | React applications |
| `p/sql-injection` | SQL injection detection | Apps with databases |
| `p/xss` | Cross-site scripting (XSS) | Web applications |
| `p/ci` | CI/CD security misconfigurations | Workflow files |
| `p/docker` | Docker security best practices | Dockerized apps |
| `p/expressjs` | Express.js security | Express backends |
| `p/jwt` | JWT security issues | Auth systems |

**Custom Rules**: Place a `.semgrep.yml` file in your repository root to add custom rules.

---

## 💡 Best Practices

1. **Start with essential rule sets**:
   ```yaml
   semgrep-rules: "p/owasp-top-ten p/secrets"
   ```

2. **Add language-specific rules**:
   ```yaml
   # For a Node.js + React + TypeScript project
   semgrep-rules: "p/owasp-top-ten p/secrets p/nodejs p/typescript p/react"
   ```

3. **Scan on every PR**:
   ```yaml
   on:
     pull_request:
       branches: [master, main]
   ```

4. **Run scheduled full scans**:
   ```yaml
   on:
     schedule:
       - cron: '0 9 * * 1'  # Weekly
   ```

5. **Use blocking on critical projects**:
   ```yaml
   fail-on-blocking-issues: true  # Recommended for production
   ```

6. **Scan specific paths for monorepos**:
   ```yaml
   jobs:
     api-scan:
       uses: ./.github/workflows/semgrep-sast.yml
       with:
         path: "packages/api"
     
     web-scan:
       uses: ./.github/workflows/semgrep-sast.yml
       with:
         path: "packages/web"
   ```

7. **Review findings regularly**: Check workflow artifacts and PR comments

8. **Fix ERROR-level issues immediately**: These indicate critical security vulnerabilities

---

## 🔗 Complete Integration Example

```yaml
# .github/workflows/pull-request.yml
name: Pull Request Checks

on:
  pull_request:
    branches:
      - master
      - main

jobs:
  # Semgrep security scan
  security-scan:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
    with:
      semgrep-rules: "p/owasp-top-ten p/nodejs p/typescript p/secrets p/react"
      fail-on-blocking-issues: true

  # Dependency security audit
  dependency-audit:
    uses: untile/github-workflows/.github/workflows/dependencies-audit.yml@v1
    with:
      package_manager: "npm"
      block_on_critical: true

  # Code quality and tests
  test:
    needs: [security-scan, dependency-audit]
    uses: untile/github-workflows/.github/workflows/mobile-pull-request.yml@v1
    with:
      node_version: "20"
```

```yaml
# .github/workflows/security-scan.yml
name: Weekly Security Scan

on:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday at 9 AM UTC
  workflow_dispatch:

jobs:
  semgrep-scan:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
    with:
      create-issues: true
      fail-on-blocking-issues: false
  
  dependency-audit:
    uses: untile/github-workflows/.github/workflows/dependencies-audit.yml@v1
```

---

## 📚 Resources

- [Semgrep Documentation](https://semgrep.dev/docs/)
- [Semgrep Rule Explorer](https://semgrep.dev/explore)
- [Writing Custom Semgrep Rules](https://semgrep.dev/docs/writing-rules/overview/)
- [Semgrep Registry](https://semgrep.dev/r)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)

---

## 🔄 Migration Guide

**Before** (manual Semgrep setup):
```yaml
# ~150 lines of workflow code
name: Security Scan
on:
  pull_request:
jobs:
  semgrep:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Semgrep
        run: pip install semgrep
      # ... many more steps
```

**After** (reusable workflow):
```yaml
# ~8 lines
name: Security Scan
on:
  pull_request:
jobs:
  semgrep:
    uses: untile/github-workflows/.github/workflows/semgrep-sast.yml@v1
    with:
      semgrep-rules: "p/owasp-top-ten p/nodejs p/secrets"
```

**Benefits**:
✅ Reduced from ~150 lines to ~8 lines  
✅ Centralized maintenance  
✅ Automatic updates with version tags  
✅ Built-in PR comments and issue creation  
✅ Consistent security checks across all projects  

---

## 🆚 Comparison with Other SAST Tools

| Feature | Semgrep | SonarQube | CodeQL |
|---------|---------|-----------|---------|
| Speed | ⚡ Very Fast | Slow | Moderate |
| Customization | ✅ Easy (YAML) | Limited | Complex |
| Language Support | 30+ languages | 25+ languages | 15+ languages |
| False Positives | Low | Moderate | Low |
| Cost | Free (OSS) | Paid (enterprise) | Free (public repos) |
| Learning Curve | Easy | Steep | Moderate |

---

## ❓ FAQ

**Q: What's the difference between ERROR, WARNING, and INFO?**  
A: 
- **ERROR**: Critical security vulnerabilities that should block deployment
- **WARNING**: Potential issues that should be reviewed
- **INFO**: Best practice suggestions and code quality tips

**Q: Can I use custom Semgrep rules?**  
A: Yes! Create a `.semgrep.yml` file in your repository root. The workflow will automatically include it.

**Q: How do I ignore false positives?**  
A: Add inline comments in your code:
```javascript
// nosemgrep: javascript.lang.security.audit.hardcoded-secret
const apiKey = "test-key-for-development";
```

**Q: Can I scan multiple directories?**  
A: Yes! Create separate jobs for each directory:
```yaml
jobs:
  scan-api:
    uses: ./.github/workflows/semgrep-sast.yml
    with:
      path: "api"
  scan-web:
    uses: ./.github/workflows/semgrep-sast.yml
    with:
      path: "web"
```

**Q: Does this work with monorepos?**  
A: Yes! Use the `path` input to target specific packages or directories.

**Q: How often should I run security scans?**  
A: 
- **On every PR**: Catch issues before they're merged
- **Weekly scheduled**: Detect new vulnerabilities in existing code
- **On master pushes**: Create tracking issues for production code

---

## 🛡️ Security Best Practices

1. **Enable branch protection**: Require security scans to pass before merging
2. **Review findings promptly**: Don't let security issues accumulate
3. **Use fail-on-blocking-issues: true**: Enforce security standards
4. **Combine with dependency audits**: Check both code and dependencies
5. **Keep Semgrep rules updated**: The workflow always uses the latest rules
6. **Monitor created issues**: Track and fix ERROR-level findings
7. **Educate the team**: Share security findings and best practices

---
