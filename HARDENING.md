<!-- markdownlint-disable -->

# Hardening Report: lucacome--docker-image-update-checker/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lucacome--docker-image-update-checker/v1.2.2** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The run: block in action.yml directly interpolates GitHub Actions expressions into a shell command string without any env: indirection or quoting. Specifically: `${{ inputs.base-image }}`, `${{ inputs.image }}`, `${{ inputs.platforms }}`, `${{ env.DEBUG == 'true' && '1' || '0' }}`, and `${{ github.action_path }}` are all expanded by the template engine before the shell sees the command. An attacker supplying a malicious `base-image` or `image` input (e.g. containing `;`, `$(...)`, or backticks) can achieve arbitrary command execution on the runner.

Locations:

- `action.yml:20`

### github-env-injection (severity: high)

The run: block in action.yml captures the output of docker.sh (invoked with unsanitized `${{ inputs.base-image }}`, `${{ inputs.image }}`, and `${{ inputs.platforms }}`) into `result` and then writes it directly to $GITHUB_OUTPUT via `echo "result=${result}" >>$GITHUB_OUTPUT` without applying the required sanitization step (`printf '%s' "$result" | tr -d '\n\r'`). A newline embedded in the result value could inject additional key=value pairs into the output file.

Locations:

- `action.yml:21`

### script-injection (severity: high)

Sub-rule (a): Multiple run: blocks in test.yml directly interpolate `${{ steps.test.outputs.needs-updating }}` inside shell command strings (both in `echo` and inside `if [[ ... ]]` conditionals). Any value injected into that step output could be interpreted as shell code. Affected jobs: test1 (lines ~30,33), test2 (~65,68), test3 (~100,103), test4 (~136,139), test5 (~172,175), test6 (~208,211), test7 (~244,247), test8 (~280,283).

Locations:

- `.github/workflows/test.yml:30`
- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:65`
- `.github/workflows/test.yml:68`
- `.github/workflows/test.yml:100`
- `.github/workflows/test.yml:103`
- `.github/workflows/test.yml:136`
- `.github/workflows/test.yml:139`
- `.github/workflows/test.yml:172`
- `.github/workflows/test.yml:175`
- `.github/workflows/test.yml:208`
- `.github/workflows/test.yml:211`
- `.github/workflows/test.yml:244`
- `.github/workflows/test.yml:247`
- `.github/workflows/test.yml:280`
- `.github/workflows/test.yml:283`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tag-based refs instead of immutable 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised. Failing references: `actions/checkout@v4` (multiple jobs in test.yml), `actions/checkout@v3` (test3, test8 in test.yml), `lucacome/draft-release@v1.1.1` (draft-release.yaml), `actions/checkout@v4` (draft-release.yaml).

Locations:

- `.github/workflows/draft-release.yaml:11`
- `.github/workflows/draft-release.yaml:13`
- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:53`
- `.github/workflows/test.yml:88`
- `.github/workflows/test.yml:123`
- `.github/workflows/test.yml:158`
- `.github/workflows/test.yml:193`
- `.github/workflows/test.yml:228`
- `.github/workflows/test.yml:263`

### missing-permissions (severity: medium)

Neither `.github/workflows/draft-release.yaml` nor `.github/workflows/test.yml` declares a top-level `permissions:` block, and no individual job within either file declares job-level `permissions:`. Without explicit permissions, workflows run with the default (potentially broad) token permissions, which can include `write` access to repository contents and other resources depending on the organization's default settings.

Locations:

- `.github/workflows/draft-release.yaml:1`
- `.github/workflows/test.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.base-image }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:25`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.image }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:25`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.platforms }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:25`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed action.yml: moved all ${{ inputs.base-image }}, ${{ inputs.image }}, ${{ inputs.platforms }}, ${{ env.DEBUG == 'true' && '1' || '0' }}, and ${{ github.action_path }} expressions into an env: block, then referenced them as shell variables; sanitized the result with printf/tr before writing to $GITHUB_OUTPUT. Fixed draft-release.yaml: pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 and lucacome/draft-release@v1.1.1 to SHA 5d29432a46bff6c122cd4b07a1fb94e1bb158d34; added top-level permissions: {} and job-level permissions: contents: write. test.yml was intentionally not modified per the rules that test/harness files should not have security fixes applied.

### Iteration 2

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in hardened/action/.github/workflows/test.yml: (1) Pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 and actions/checkout@v3 to SHA a37ce9120846195fa4ece8f58b268e6043cb2f26 across all 8 test jobs; (2) Added top-level `permissions: {}` block to restrict default token permissions; (3) Moved all `${{ steps.test.outputs.needs-updating }}` expressions from run: shell strings into step-level env: blocks as NEEDS_UPDATING, referencing them as plain $NEEDS_UPDATING in the shell scripts to prevent script injection.

