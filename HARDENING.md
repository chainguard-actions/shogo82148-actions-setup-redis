<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-redis/v1.58.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-redis/v1.58.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: `${{ steps.setup.outputs.* }}` expressions are interpolated directly inside `run:` shell command strings in the `test` and `test-tls` jobs. `steps.*.outputs.*` is a workflow-controllable context that flows through YAML template substitution before the shell sees it, enabling script injection. Offending lines:
- Line 57: `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-port }}" ping`
- Line 59: `run: redis-cli -s "${{ steps.setup.outputs.redis-unix-socket }}" ping`
- Lines 111–116: `run:` block using `${{ steps.setup.outputs.redis-tls-port }}` and `${{ steps.setup.outputs.redis-tls-dir }}`
Fix: move the values into `env:` variables and reference them as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:59`
- `.github/workflows/test.yml:111`

### missing-permissions (severity: medium)

The workflow file `check-dist.yml` has no top-level `permissions:` key and the only job (`check-dist`) also has no job-level `permissions:` key. Without explicit permissions, the job inherits the repository default (often `write-all` for older repositories), granting broader access than necessary. Add a `permissions:` block with minimal required scopes (e.g., `contents: read`).

Locations:

- `.github/workflows/check-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script-injection in .github/workflows/test.yml by moving all ${{ steps.setup.outputs.* }} expressions into step-level env: blocks (REDIS_PORT, REDIS_UNIX_SOCKET, REDIS_TLS_PORT, REDIS_TLS_DIR) and referencing them as plain shell variables. Fixed missing-permissions in .github/workflows/check-dist.yml by adding `permissions: contents: read` to the check-dist job.

