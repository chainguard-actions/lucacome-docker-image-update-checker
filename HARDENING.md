<!-- markdownlint-disable -->

# Hardening Report: lucacome--docker-image-update-checker/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **lucacome--docker-image-update-checker/v1.2.2** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates multiple GitHub Actions expressions inside the shell command string, violating rule (a). The following expressions are embedded verbatim into the shell before it executes:
- `${{ env.DEBUG == 'true' && '1' || '0' }}` (env context)
- `${{ inputs.base-image }}` (attacker-controlled input)
- `${{ inputs.image }}` (attacker-controlled input)
- `${{ inputs.platforms }}` (attacker-controlled input)
- `${{ github.action_path }}` (github context)

An attacker supplying a malicious value for `inputs.base-image`, `inputs.image`, or `inputs.platforms` (e.g. containing shell metacharacters, command substitution, or semicolons) can achieve arbitrary command execution on the runner. All `${{ ... }}` expressions must be moved to an `env:` block and the resulting env vars must be double-quoted in the shell script.

Locations:

- `action.yml:23`

### github-env-injection (severity: high)

The `run:` block writes the variable `result` — which is derived from executing `docker.sh` with unsanitized `${{ inputs.* }}` values injected directly into the shell environment — to `$GITHUB_OUTPUT` via `echo "result=${result}" >>$GITHUB_OUTPUT` without applying the required sanitization step (`printf '%s' "$result" | tr -d '\n\r'`). Because the inputs are attacker-controlled and flow unsanitized into the script execution and then into `$GITHUB_OUTPUT`, a crafted input could inject additional key=value pairs into the output file, potentially poisoning downstream steps. The sanitization pipeline must be applied before every write to `$GITHUB_OUTPUT`.

Locations:

- `action.yml:24`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed action.yml by moving all ${{ }} expressions to an env: block (TRACE, base, image, platforms, ACTION_PATH) and referencing them as double-quoted shell variables. Added newline sanitization (printf '%s' "$result" | tr -d '\n\r') before writing to $GITHUB_OUTPUT to prevent env injection. The docker.sh script already reads base, image, platforms, and TRACE as environment variables, so the behavior is preserved.

