# TLS Registry Refresh Automation

This automation simplifies the process of updating the TLS registry from nightly builds.

## Quick Start

### Standard Mode (Known Artifacts)
```bash
make refresh-tls-registry NIGHTLY_URL=https://amd64.ocp.releases.ci.openshift.org/releasestream/4.21.0-0.nightly/release/4.21.0-0.nightly-2025-10-03-111550
```

### Discovery Mode (Detect New Artifacts) ⭐ Recommended
```bash
DISCOVER_MODE=true make refresh-tls-registry NIGHTLY_URL=https://amd64.ocp.releases.ci.openshift.org/releasestream/4.21.0-0.nightly/release/4.21.0-0.nightly-2025-10-03-111550
```

**💡 Tip**: Use discovery mode when you suspect new certificates have been added or when investigating what's in a nightly build.

## Prerequisites

1. **gh CLI** - GitHub CLI tool for creating PRs
   ```bash
   # Install if needed
   brew install gh
   gh auth login
   ```

2. **Job IDs** - You need to provide job IDs for each platform/featureset combination via environment variables

## How It Works

The script automates these steps:
1. Parses the nightly URL to extract release information
2. Downloads rawTLSInfo artifacts from test jobs for each platform/featureset
3. Validates annotation consistency across all artifacts
4. Regenerates TLS ownership/violation metadata
5. Creates a pull request with the changes

## Providing Job IDs

Since automatic job discovery is complex, you need to provide job IDs manually via environment variables.

### Finding Job IDs

1. Visit the nightly release page (the URL you're passing to the script)
2. Click on "Tests" to see all test jobs for that release
3. For each successful job, find the job ID (the long number in the URL)
4. Export environment variables in the format: `PERIODIC_CI_..._JOB_ID=<job-id>`

### Example: Finding AWS Default Job ID

1. Look for a job named like: `periodic-ci-openshift-release-master-nightly-4.21-e2e-aws-ovn`
2. Click on the successful run
3. The URL will be: `https://prow.ci.openshift.org/view/gs/test-platform-results/logs/periodic-ci-openshift-release-master-nightly-4.21-e2e-aws-ovn/1234567890`
4. Extract the job ID: `1234567890`
5. Export: `export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_4_21_E2E_AWS_OVN_JOB_ID=1234567890`

### Required Environment Variables

For a typical nightly (replace X.Y with the version and <job-id> with actual IDs):

```bash
# AWS
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_AWS_OVN_JOB_ID=<job-id>
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_AWS_OVN_TECHPREVIEW_JOB_ID=<job-id>
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_AWS_OVN_SINGLE_NODE_JOB_ID=<job-id>
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_AWS_OVN_TECHPREVIEW_SINGLE_NODE_JOB_ID=<job-id>

# Azure
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_AZURE_OVN_JOB_ID=<job-id>
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_AZURE_OVN_TECHPREVIEW_JOB_ID=<job-id>

# GCP
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_GCP_OVN_JOB_ID=<job-id>
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_GCP_OVN_TECHPREVIEW_JOB_ID=<job-id>

# Metal
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_METAL_OVN_JOB_ID=<job-id>
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_METAL_OVN_TECHPREVIEW_JOB_ID=<job-id>

# vSphere
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_VSPHERE_OVN_JOB_ID=<job-id>
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_VSPHERE_OVN_TECHPREVIEW_JOB_ID=<job-id>

# OpenStack
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_X_Y_E2E_OPENSTACK_OVN_JOB_ID=<job-id>
```

## Complete Example

```bash
# Step 1: Set job IDs for all platforms
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_4_21_E2E_AWS_OVN_JOB_ID=1234567890
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_4_21_E2E_AWS_OVN_TECHPREVIEW_JOB_ID=1234567891
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_4_21_E2E_AWS_OVN_SINGLE_NODE_JOB_ID=1234567892
export PERIODIC_CI_OPENSHIFT_RELEASE_MASTER_NIGHTLY_4_21_E2E_AWS_OVN_TECHPREVIEW_SINGLE_NODE_JOB_ID=1234567893
# ... (set all others)

# Step 2: Run the automation
make refresh-tls-registry NIGHTLY_URL=https://amd64.ocp.releases.ci.openshift.org/releasestream/4.21.0-0.nightly/release/4.21.0-0.nightly-2025-10-03-111550
```

## Options

### Discovery Mode (Recommended for New Cert Detection)

**Use this mode when you suspect new certificates have been added or when platforms/topologies have changed.**

Discovery mode automatically scans all known test jobs and downloads ALL rawTLSInfo artifacts found, not just the hardcoded list. This helps detect:
- New certificates added by components
- New platforms or topologies
- New featuresets
- Architecture-specific variations

```bash
DISCOVER_MODE=true make refresh-tls-registry NIGHTLY_URL=<url>
```

When new files are discovered, the script will:
- Download them automatically
- Mark them clearly in the output with 🆕
- List all new files in the summary
- Warn you to review what changed

**Example output when new files are found:**
```
🆕 NEW FILE DISCOVERED: raw-tls-artifacts-ha-amd64-rosa-ovn-default.json

🆕 Newly discovered artifacts:
  • raw-tls-artifacts-ha-amd64-rosa-ovn-default.json

⚠️  IMPORTANT: New TLS artifacts were discovered!
   This could indicate:
   - A new platform or topology was added
   - A new featureset was introduced  
   - A component added new certificates
```

### Dry Run Mode

Test the download process without creating a PR:

```bash
DRY_RUN=true make refresh-tls-registry NIGHTLY_URL=<url>
```

### Force Re-download

Re-download all artifacts even if they already exist:

```bash
FORCE_DOWNLOAD=true make refresh-tls-registry NIGHTLY_URL=<url>
```

### Combine Options

You can combine multiple options:

```bash
# Discover new files in dry-run mode
DISCOVER_MODE=true DRY_RUN=true make refresh-tls-registry NIGHTLY_URL=<url>

# Force re-download with discovery
DISCOVER_MODE=true FORCE_DOWNLOAD=true make refresh-tls-registry NIGHTLY_URL=<url>
```

### Additional Jobs

If you need to scan additional job patterns beyond the standard set:

```bash
ADDITIONAL_JOBS="job-name-1,job-name-2" DISCOVER_MODE=true make refresh-tls-registry NIGHTLY_URL=<url>
```

## Troubleshooting

### "Job ID not found" Errors

If you see errors like:
```
✗ raw-tls-artifacts-ha-amd64-aws-ovn-default.json (job ID not found for periodic-ci-...)
```

This means you need to set the corresponding environment variable. The error message will show you the exact variable name to use.

### Annotation Consistency Errors

If you see:
```
ANNOTATION CONSISTENCY ERROR DETECTED
mismatch of certificate info for --namespace=X secret/Y
```

This means the same certificate appears in multiple raw-data files with different annotations. To fix:

1. Identify the conflicting certificate from the error message
2. Manually edit the affected `tls/raw-data/*.json` files
3. Ensure all instances have identical annotations
4. Re-run the script

### Download Failures

If artifact downloads fail:
- Verify the job ID is correct
- Check that the job completed successfully
- Verify the artifact exists in the job's output
- Check network connectivity to gcsweb

## Advanced Usage

### Incremental Updates

If some downloads fail, you can:
1. Fix the job IDs for failed downloads
2. Re-run the script
3. It will skip already-downloaded files automatically

### Manual Job URL Specification

For special platforms (OpenStack, ROSA, two-node arbiter), you may need to use different job names. Modify the script's job name patterns or download artifacts manually to `tls/raw-data/` before running.

## Files Modified

The script modifies these directories:
- `tls/raw-data/*.json` - Raw TLS artifact data
- `tls/ownership/` - Ownership metadata
- `tls/violations/` - Violation tracking
- `tls/descriptions/` - Description metadata
- `tls/refresh-period/` - Refresh period metadata
- `tls/autoregenerate-after-expiry/` - Auto-regeneration metadata
- `tls/testcase/` - Test case metadata

## See Also

- `hack/update-tls-ownership.sh` - Regenerates ownership without downloading new data
- `tls/README.md` - TLS registry documentation
- PR #30116 - Example of a manual TLS registry update
