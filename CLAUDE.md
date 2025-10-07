# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a centralized repository for **reusable GitHub Actions workflows**. It provides standardized CI/CD pipelines and composite actions that can be consumed by other repositories in the organization to maintain consistency and reduce duplication.

## Architecture

### Workflows vs Composite Actions
- **Reusable workflows** (`.github/workflows/`) are called from other repositories via `workflow_call` trigger
- **Composite actions** (hypothetical `actions/` directory - none currently exist) would bundle multiple steps into reusable actions
- Each workflow is designed to be version-tagged and referenced as `@v1`, `@v1.2.0`, etc.

### Workflow Inputs and Secrets Pattern
All workflows follow a consistent pattern:
1. Define inputs with sensible defaults in the `workflow_call` section
2. Declare required/optional secrets
3. Use conditional logic based on inputs (e.g., `if: ${{ inputs.publish_to_firebase }}`)
4. Environment-specific behavior controlled via the `environment` input (preview/production)

### Environment Variable Strategy (mobile-publish-app.yml)
The mobile publish workflow uses a multi-layered environment approach:
1. `NODE_CONFIG_ENV` set to the input `environment` (preview/production)
2. `EXPO_PUBLIC_ENV` pulled from EAS environment variables via `eas env:pull`
3. All `.env.local` variables exported to `$GITHUB_ENV` for cross-step persistence
4. Environment validation ensures `EXPO_PUBLIC_ENV` matches the expected `environment`

## Available Workflows

### 1. mobile-pull-request.yml
**Purpose**: Run test suite for mobile projects
**Key steps**: Node/Bun setup → Install deps → Commitlint → Lint
**Target consumers**: Mobile app repositories for PR validation
**File**: `.github/workflows/mobile-pull-request.yml:1`

### 2. mobile-publish-app.yml
**Purpose**: Build and distribute iOS apps via Expo EAS
**Key steps**:
- Environment setup and cache cleaning (`.github/workflows/mobile-publish-app.yml:99`)
- EAS build with local runner on macOS (`.github/workflows/mobile-publish-app.yml:187`)
- Conditional distribution to Firebase App Distribution (`.github/workflows/mobile-publish-app.yml:216`) and/or TestFlight (`.github/workflows/mobile-publish-app.yml:194`)
**Important**: Firebase credentials are dynamically selected based on `environment` input (`.github/workflows/mobile-publish-app.yml:73`)

## Development Workflow

### Testing Changes
Since this repository provides workflows consumed by others, testing must be done in a consuming repository:

1. Make changes to workflows in this repository
2. Push to a feature branch (e.g., `feature/new-workflow`)
3. In a consuming repository, reference the branch:
   ```yaml
   uses: untile/github-workflows/.github/workflows/mobile-pull-request.yml@feature/new-workflow
   ```
4. Test thoroughly
5. When stable, return to this repo and create a version tag:
   ```bash
   git tag v1.1.0
   git push origin v1.1.0
   ```

### Versioning Strategy
- **Major versions** (v1, v2): Breaking changes to inputs/secrets/behavior
- **Minor versions** (v1.1.0): New features, backward compatible
- **Patch versions** (v1.0.1): Bug fixes
- Consumers typically reference `@v1` for auto-updates within major version

### Documentation Structure
- `documentation/` contains detailed docs for each workflow
- Each workflow doc includes:
  - Description of purpose
  - Input parameters with defaults
  - Required/optional secrets
  - Example usage

## Key Implementation Details

### Cache Strategy
Both workflows cache dependencies but with different approaches:
- **mobile-pull-request.yml**: Caches `node_modules` and `bun.lockb` based on lockfile hash (`.github/workflows/mobile-pull-request.yml:56`)
- **mobile-publish-app.yml**: Caches Bun cache directory `~/.bun/install/cache` plus lockfiles (`.github/workflows/mobile-publish-app.yml:123`)

### EAS Build Configuration
The mobile publish workflow uses local EAS builds on macOS runners:
- `EAS_LOCAL_BUILD_SKIP_CLEANUP=1` prevents automatic cleanup (`.github/workflows/mobile-publish-app.yml:69`)
- Custom working directory via `EAS_LOCAL_BUILD_WORKINGDIR` (`.github/workflows/mobile-publish-app.yml:70`)
- Aggressive cache cleaning before build to avoid stale artifacts (`.github/workflows/mobile-publish-app.yml:99`)
- `EAS_NO_XCPRETTY=1` for cleaner build output (`.github/workflows/mobile-publish-app.yml:187`)

### Firebase + TestFlight Conditional Publishing
The workflow supports flexible distribution:
- Can publish to both platforms, one, or neither
- Secrets only validated when the corresponding `publish_to_*` input is true
- Firebase credentials handled via Google auth action (`.github/workflows/mobile-publish-app.yml:204`)
- Project ID extracted from service account JSON at runtime (`.github/workflows/mobile-publish-app.yml:211`)
