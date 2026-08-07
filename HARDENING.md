<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-redis/v1.56.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-redis/v1.56.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ ... }}` expressions from the `steps.*.outputs.*` context are directly interpolated inside `run:` shell commands in test.yml. This allows shell metacharacter injection if any step output contains attacker-influenced data. Offending lines:
- Line 63: `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-port }}" ping`
- Line 65: `run: redis-cli -s "${{ steps.setup.outputs.redis-unix-socket }}" ping`
- Lines 109-113: `run: |` block using `${{ steps.setup.outputs.redis-tls-port }}`, `${{ steps.setup.outputs.redis-tls-dir }}` multiple times.
These expressions should be moved to `env:` variables and the shell expansions double-quoted.

Locations:

- `.github/workflows/test.yml:63`
- `.github/workflows/test.yml:65`
- `.github/workflows/test.yml:109`

### missing-permissions (severity: medium)

The workflow file check-dist.yml has no top-level `permissions:` key and the single job `check-dist` also has no job-level `permissions:` key. This means the workflow runs with GitHub's default permissions (which may include `contents: write` and other broad scopes depending on repository settings). A minimal `permissions:` block (e.g. `contents: read`) should be added.

Locations:

- `.github/workflows/check-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script-injection in .github/workflows/test.yml by moving all `${{ steps.setup.outputs.* }}` expressions to `env:` blocks and referencing them as plain environment variables in the shell commands: REDIS_PORT (line 63), REDIS_UNIX_SOCKET (line 65), REDIS_TLS_PORT and REDIS_TLS_DIR (lines 109-113). Fixed missing-permissions in .github/workflows/check-dist.yml by adding `permissions: contents: read` to the `check-dist` job.

