<!-- markdownlint-disable -->

# Hardening Report: lucacome--docker-image-update-checker/v3.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lucacome--docker-image-update-checker/v3.1.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### broad-permissions (severity: medium)

scorecards.yml has a top-level `permissions: read-all` which grants overly broad read access to all scopes. This should be replaced with specific minimal permissions at the job level (the job already has specific permissions, so the top-level `read-all` can be removed or replaced with `permissions: {}`).

Locations:

- `.github/workflows/scorecards.yml:14`

### script-injection (severity: high)

test-workflow.yml contains numerous `run:` blocks that directly interpolate `${{ steps.test.outputs.* }}` expressions (sub-rule a). The `steps.*.outputs.*` context is listed as an untrusted input source — its values flow through YAML template substitution before the shell ever sees them, allowing an attacker to inject shell metacharacters. Examples include: `echo "Images: ${{ steps.test.outputs.diff-images }}"` and `if [[ "${{ steps.test.outputs.needs-updating }}" != "true" ]]`. These patterns repeat throughout the entire file across all test jobs. The fix is to route the values through env vars and double-quote the shell expansions, e.g. `env: NEEDS_UPDATING: ${{ steps.test.outputs.needs-updating }}` then `if [[ "$NEEDS_UPDATING" != "true" ]]`.

Locations:

- `.github/workflows/test-workflow.yml:44`
- `.github/workflows/test-workflow.yml:47`
- `.github/workflows/test-workflow.yml:52`
- `.github/workflows/test-workflow.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** broad-permissions, script-injection

**Notes:**

1. scorecards.yml: Replaced top-level `permissions: read-all` with `permissions: {}` since each job already defines its own specific minimal permissions. 2. test-workflow.yml: Moved all `${{ steps.test.outputs.* }}` expressions out of `run:` shell blocks and into `env:` blocks for each affected step. Shell scripts now reference plain environment variables ($NEEDS_UPDATING, $DIFF_IMAGES, $DIFF_JSON, $NEEDS_BUILDING) instead of inline template expressions. Step `name:` fields that display output values for logging purposes were left unchanged as they are not shell-executed.

