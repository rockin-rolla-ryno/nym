# CodeSignTool Update Guide

## Security Context

The Windows wallet release workflow (`publish-nym-wallet-win11.yml`) uses SSL.com's CodeSignTool for code signing. To prevent supply chain attacks, the tool is:
- Pinned to a specific version
- Downloaded from a versioned URL (not a redirect endpoint)
- Verified with a SHA256 checksum before execution

## How to Update CodeSignTool

When a new version of CodeSignTool is released and you need to update it:

### Step 1: Identify the New Version
Visit the official SSL.com CodeSignTool GitHub repository:
https://github.com/SSLcom/codesigntool/releases

Identify the version you want to upgrade to (e.g., `v1.3.2`).

### Step 2: Download and Verify the Checksum
Download the release archive manually:
```bash
VERSION="v1.3.2"  # Replace with the actual version
curl -L -o CodeSignTool-${VERSION}.zip \
  "https://github.com/SSLcom/codesigntool/releases/download/${VERSION}/CodeSignTool-${VERSION}.zip"
```

Calculate the SHA256 checksum:
```bash
sha256sum CodeSignTool-${VERSION}.zip
```

### Step 3: Verify Against Official Sources
Cross-reference the checksum with:
1. The checksum file in the GitHub release (if provided)
2. SSL.com's official documentation
3. The release notes or security advisories

**IMPORTANT**: Do not proceed if you cannot verify the checksum from an official source.

### Step 4: Update the Workflow
Edit `.github/workflows/publish-nym-wallet-win11.yml`:

1. Update the `CODESIGNTOOL_VERSION` environment variable:
   ```yaml
   CODESIGNTOOL_VERSION: "v1.3.2"  # New version
   ```

2. Update the `EXPECTED_CHECKSUM` environment variable with the verified checksum:
   ```yaml
   EXPECTED_CHECKSUM: "abc123..."  # The SHA256 checksum you calculated and verified
   ```

### Step 5: Test the Update
Before merging to the main branch:
1. Create a test branch
2. Trigger the workflow manually using `workflow_dispatch`
3. Set `sign: false` to avoid consuming signing credits
4. Verify that the download and checksum verification succeed
5. Review the workflow logs to ensure CodeSignTool was downloaded and verified correctly

### Step 6: Document the Change
In your pull request or commit message, include:
- The new version number
- The source of the checksum verification
- Any relevant security advisories or release notes
- Testing performed

## Security Considerations

### Why Version Pinning?
Without version pinning, the workflow would download whatever version SSL.com serves at the redirect endpoint. An attacker who compromises:
- The SSL.com website
- The redirect chain
- The hosting infrastructure
- The served archive

...could inject malicious code that would execute with access to:
- Code signing credentials
- GitHub secrets
- Release artifacts

### Why Checksum Verification?
Even with version pinning, an attacker who compromises the download source could serve a malicious archive. Checksum verification ensures that:
- The downloaded file matches the expected content
- Any tampering is detected before execution
- The workflow fails safely if verification fails

### Defense in Depth
This approach provides multiple layers of security:
1. **Version pinning**: Prevents automatic updates to potentially compromised versions
2. **Checksum verification**: Detects tampering or corruption
3. **Fail-safe behavior**: The workflow exits with an error if verification fails
4. **Audit trail**: All downloads and verifications are logged

## Troubleshooting

### Checksum Mismatch
If you encounter a checksum mismatch error:
1. **DO NOT** simply update the checksum without investigation
2. Verify that you're using the correct version number
3. Re-download the archive and recalculate the checksum
4. Check SSL.com's official sources for the expected checksum
5. If the mismatch persists, investigate potential security issues before proceeding

### Download Failures
If the download fails:
1. Check that the version exists in the GitHub releases
2. Verify the URL format matches the expected pattern
3. Check for network issues or rate limiting
4. Review SSL.com's release notes for any changes to the distribution method

## Contact

For security concerns related to CodeSignTool or this workflow, contact the security team.
