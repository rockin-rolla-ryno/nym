# Security Requirements for Nym Node Ansible Deployment

## Script Integrity Verification

The Ansible playbooks download and execute shell scripts as root from remote sources. To prevent execution of tampered or malicious content, **SHA256 checksum verification is mandatory** for all deployment scripts.

### Required Configuration

Before running the `tunnel` or `quic` roles, you **MUST** set the following checksums in `playbooks/group_vars/all.yml`:

```yaml
tunnel_manager_sha256: "<sha256_checksum>"
quic_bridge_deployment_sha256: "<sha256_checksum>"
```

### Generating Checksums

To generate the required SHA256 checksums, run:

```bash
# For network-tunnel-manager.sh
curl -fsSL https://github.com/nymtech/nym/raw/refs/heads/develop/scripts/nym-node-setup/network-tunnel-manager.sh | sha256sum

# For quic_bridge_deployment.sh
curl -fsSL https://raw.githubusercontent.com/nymtech/nym/refs/heads/develop/scripts/nym-node-setup/quic_bridge_deployment.sh | sha256sum
```

Copy the resulting checksums (first field of the output) into the respective variables in `group_vars/all.yml`.

### Security Rationale

The deployment scripts are downloaded from mutable GitHub branch URLs (`refs/heads/develop`). Without integrity verification:

- A compromised upstream repository could inject malicious code
- A compromised maintainer account could modify the scripts
- Network-level attacks (DNS hijacking, MITM) could serve malicious content
- The scripts execute with root privileges via Ansible's `become: true`

SHA256 checksum verification ensures that:

1. Only scripts with known, trusted content are executed
2. Any modification to the upstream scripts will cause deployment to fail
3. Operators must explicitly update checksums when upgrading to new script versions
4. The security decision is explicit and auditable

### Updating Scripts

When you intentionally want to use a newer version of the deployment scripts:

1. Review the changes in the upstream repository
2. Generate new checksums using the commands above
3. Update the checksum variables in `group_vars/all.yml`
4. Document the reason for the update in your change management process

### Failure Behavior

If checksums are not configured or do not match:

- **Missing checksum**: Deployment will fail immediately with a clear error message before downloading the script
- **Checksum mismatch**: Deployment will fail after download but before execution, displaying both expected and actual checksums

This fail-safe behavior prevents accidental execution of unverified scripts.

## Alternative: Pinning to Immutable Commits

For enhanced security, consider modifying the URLs to reference specific Git commit hashes instead of the mutable `develop` branch:

```yaml
tunnel_manager_url: "https://github.com/nymtech/nym/raw/<commit_hash>/scripts/nym-node-setup/network-tunnel-manager.sh"
quic_bridge_deployment_url: "https://raw.githubusercontent.com/nymtech/nym/<commit_hash>/scripts/nym-node-setup/quic_bridge_deployment.sh"
```

This approach provides immutability at the source level, though checksum verification should still be used as defense-in-depth.
