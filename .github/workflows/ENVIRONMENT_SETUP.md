# GitHub Environments Setup for cd-docs.yml

## Security Notice

The `cd-docs.yml` workflow has been configured to use GitHub Environments to protect sensitive credentials from being exposed to untrusted code execution during manual workflow dispatches.

## Required Environments

To ensure the workflow functions correctly and securely, the following GitHub Environments must be configured in the repository settings:

### 1. vercel-docs-preview

**Purpose:** Protects preview deployments to Vercel

**Configuration:**
- Navigate to: Repository Settings → Environments → New environment
- Name: `vercel-docs-preview`
- **Recommended Protection Rules:**
  - ✅ Required reviewers: Add at least one trusted reviewer
  - ✅ Wait timer: Optional (e.g., 5 minutes for automated checks)
  - ⚠️ Deployment branches: Configure based on your branching strategy

**Required Secrets:**
- `VERCEL_TOKEN` - Vercel API token with deployment permissions
- `VERCEL_PROJECT_JSON_BASE64` - Base64-encoded Vercel project configuration

### 2. vercel-docs-production

**Purpose:** Protects production deployments to Vercel

**Configuration:**
- Navigate to: Repository Settings → Environments → New environment
- Name: `vercel-docs-production`
- **Recommended Protection Rules:**
  - ✅ Required reviewers: Add at least 2 trusted reviewers
  - ✅ Wait timer: Optional (e.g., 10 minutes for final checks)
  - ✅ Deployment branches: Restrict to `master` branch only

**Required Secrets:**
- `VERCEL_TOKEN` - Vercel API token with production deployment permissions
- `VERCEL_PROJECT_JSON_BASE64` - Base64-encoded Vercel project configuration

## Security Benefits

This configuration provides the following security improvements:

1. **Credential Isolation:** The `VERCEL_TOKEN` is only exposed to deployment jobs that run in protected environments, not during the build phase where untrusted code executes.

2. **Manual Approval:** Required reviewers must approve deployments before credentials are exposed, preventing unauthorized deployments from malicious branches.

3. **Audit Trail:** All deployment approvals are logged in GitHub's audit log, providing accountability.

4. **Branch Protection:** Production deployments can be restricted to only run from the `master` branch.

## Migration Steps

1. Create both environments in GitHub repository settings
2. Configure required reviewers for each environment
3. Move the `VERCEL_TOKEN` and `VERCEL_PROJECT_JSON_BASE64` secrets from repository secrets to environment secrets
4. Test the workflow with a non-production branch to verify preview deployment approval flow
5. Test with the master branch to verify production deployment approval flow

## Additional Recommendations

- **Separate Tokens:** Consider using separate Vercel tokens for preview and production environments with appropriate permission scoping
- **Token Rotation:** Regularly rotate the `VERCEL_TOKEN` credentials
- **Audit Reviews:** Periodically review the list of required reviewers and update as team membership changes
- **Branch Protection:** Enable branch protection rules on `master` to prevent unauthorized pushes
