<!-- markdownlint-disable -->

# Hardening Report: lucacome--docker-image-update-checker/v3.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lucacome--docker-image-update-checker/v3.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in test-workflow.yml directly interpolate ${{ steps.test.outputs.* }} expressions (steps.*.outputs.* context) into shell commands without routing through env: variables. Examples include: `echo "Images: ${{ steps.test.outputs.diff-images }}"` and `if [[ "${{ steps.test.outputs.needs-updating }}" != "true" ]]`. These expressions flow through YAML template substitution before the shell processes them, allowing any value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) to be interpreted as shell code. This pattern is repeated across all test jobs throughout the file (rule a: direct expression interpolation in run: blocks).

Locations:

- `.github/workflows/test-workflow.yml:38`
- `.github/workflows/test-workflow.yml:43`
- `.github/workflows/test-workflow.yml:48`

### broad-permissions (severity: medium)

scorecards.yml sets top-level `permissions: read-all`, which grants overly broad read access to all scopes. This should be replaced with specific minimal permissions (e.g., contents: read, security-events: write, id-token: write, actions: read) matching only what the jobs actually require.

Locations:

- `.github/workflows/scorecards.yml:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, broad-permissions

**Notes:**

1. script-injection (test-workflow.yml): Rewrote the entire test-workflow.yml to move all ${{ steps.test.outputs.* }} expressions from run: shell blocks into env: blocks. Each step that previously interpolated outputs directly into shell commands (echo, if [[ ... ]]) now declares env vars (DIFF_IMAGES, NEEDS_UPDATING, DIFF_JSON, NEEDS_BUILDING) and references them as plain shell variables. Step name: fields retain the expressions (safe, not executed as shell). 2. broad-permissions (scorecards.yml): Replaced top-level `permissions: read-all` with `permissions:\n  contents: read` — the minimal permission needed at the workflow level. The job-level permissions block already grants the specific elevated permissions (security-events: write, id-token: write, contents: read, actions: read) required by the scorecard analysis job.

