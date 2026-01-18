# Renovate Configuration for Containers Repository

## Overview

This repository uses the **GitHub Renovate App** for automated dependency updates. Since this is a **public repository**, we use GitHub's official Renovate app instead of self-hosting.

**Advantages:**
- ✅ No GitHub Actions minutes consumed
- ✅ Runs automatically on GitHub's infrastructure  
- ✅ Hourly update checks
- ✅ Zero maintenance required
- ✅ Better performance and reliability

## Installation

### GitHub Renovate App

1. Visit https://github.com/apps/renovate
2. Click **"Configure"**
3. Select **first-it-consulting** organization
4. Choose **"Only select repositories"**
5. Select **containers** repository
6. Click **"Save"**

Renovate will:
- Automatically detect `.renovaterc.json5` configuration
- Create PRs for outdated dependencies  
- Run hourly to check for updates
- Auto-merge PRs via the `auto-merge-renovate.yaml` workflow

## Configuration

Renovate monitors version updates in `docker-bake.hcl` files using custom annotations:

```hcl
variable "VERSION" {
  // renovate: datasource=docker depName=ghcr.io/actions/actions-runner
  default = "2.330.0"
}
```

**Supported datasources:**
- `docker` - Docker images
- `github-releases` - GitHub release tags
- `repology` - Alpine/Repology packages

**Optional parameters:**
- `versioning=<type>` - Override version detection (e.g., `semver`, `loose`)

### Workflows

#### 1. Nightly Build (3:45 AM)
**File**: `.github/workflows/nightly-build.yaml`

Rebuilds all containers every night at 3:45 AM to ensure:
- Latest security patches
- Fresh base images
- Updated dependencies

#### 2. Auto-merge Renovate PRs
**File**: `.github/workflows/auto-merge-renovate.yaml`

Automatically merges Renovate PRs using:
- Squash merge strategy
- Rebase updates
- 6 retry attempts with 10s delay

**Note**: For full functionality, enable branch protection rules:
1. Go to Settings → Branches → Branch protection rules
2. Add rule for `main` branch
3. Enable "Require status checks to pass before merging"
4. Select required checks (e.g., build workflows)

#### 3. Vulnerability Scan (2:30 AM)
**File**: `.github/workflows/vulnerability-scan.yaml`

Daily security scanning with:
- Grype scanner
- SARIF upload to GitHub Security tab
- Database caching for faster scans

#### 4. Release Workflow
**File**: `.github/workflows/release.yaml`

Triggers on:
- Pull requests to main
- Pushes to main (changes in `apps/**`)
- Manual dispatch

## Enabling Renovate

### Option 1: GitHub Renovate App (Recommended)

Since this is a public repository, use GitHub's official Renovate app:

1. Visit https://github.com/apps/renovate
2. Click "Configure"
3. Select `first-it-consulting` organization
4. Choose "Only select repositories"
5. Select `containers` repository
6. Click "Save"

Renovate will:
- Automatically detect `.renovaterc.json5`
- Create PRs for outdated dependencies
- Run on its own schedule (typically hourly)
- Auto-merge PRs (if workflows pass)

### Option 2: Mend Renovate (Alternative)

1. Visit https://www.mend.io/renovate/
2. Sign up with GitHub account
3. Install on `first-it-consulting/containers`
4. Configure via `.renovaterc.json5`

## Current Apps

The repository contains the following apps:

- **actions-runner**: GitHub Actions runner (from `ghcr.io/actions/actions-runner`)
- **cni-plugins**: CNI network plugins (from `containernetworking/plugins`)
- **mcpo**: Custom app with GitHub releases tracking
- **postgres-init**: PostgreSQL initialization container (Alpine-based)

## Testing Renovate

To test if Renovate is working:

1. **Check for PRs**: Renovate should create PRs within 1-2 hours
2. **Manual trigger**: Comment `@renovatebot rebase` on a Renovate PR
3. **Force run**: In Renovate dashboard, click "Run now" for this repository

## Troubleshooting

### No PRs created

1. Check Renovate is installed: https://github.com/first-it-consulting/containers/settings/installations
2. Review Renovate logs in the app dashboard
3. Validate `.renovaterc.json5` syntax

### Auto-merge not working

1. Verify branch protection rules are configured
2. Check workflow runs in Actions tab
3. Ensure all required checks are passing

### Custom regex not matching

Test the regex pattern:
```bash
grep -A1 "renovate:" apps/*/docker-bake.hcl
```

Expected output:
```
  // renovate: datasource=docker depName=ghcr.io/actions/actions-runner
  default = "2.330.0"
```

## Monitoring

- **Dependency Dashboard**: Check for Renovate PRs in Issues tab
- **Security Alerts**: Monitor https://github.com/first-it-consulting/containers/security
- **Workflow Runs**: https://github.com/first-it-consulting/containers/actions
- **Nightly Builds**: Check Actions tab at ~3:45 AM UTC

## Comparison with containers-images

| Feature | containers (public) | containers-images (private) |
|---------|-------------------|---------------------------|
| Renovate | GitHub App | Self-hosted workflow |
| SARIF Upload | ✅ Yes | ❌ No (requires Enterprise) |
| Cost | Free | Free (uses GitHub Actions minutes) |
| Setup | Simple (install app) | Complex (PAT, workflow, secrets) |
| Updates | Hourly (automatic) | Every 6 hours (cron) |
| Advanced Security | ✅ Available | ❌ Not available |

## Next Steps

1. ✅ Install GitHub Renovate app
2. ✅ Enable branch protection for auto-merge
3. ✅ Monitor first Renovate run
4. ✅ Review and merge initial PRs
5. ✅ Verify nightly builds run successfully
