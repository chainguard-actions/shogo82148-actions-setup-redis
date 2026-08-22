<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-redis/v1.57.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-redis/v1.57.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: `${{ steps.setup.outputs.* }}` expressions are interpolated directly inside `run:` shell command strings. These values flow through YAML template substitution before the shell sees them, allowing an attacker who can influence action outputs to inject shell metacharacters. Offending lines:
- Line 63: `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-port }}" ping`
- Line 65: `run: redis-cli -s "${{ steps.setup.outputs.redis-unix-socket }}" ping`
- Lines ~106-110 (test-tls job): `${{ steps.setup.outputs.redis-tls-port }}` and `${{ steps.setup.outputs.redis-tls-dir }}` used in a multi-line `run:` block.
Fix: pass these values via `env:` variables and reference them as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/test.yml:63`
- `.github/workflows/test.yml:65`
- `.github/workflows/test.yml:106`

### missing-permissions (severity: medium)

`check-dist.yml` has no top-level `permissions:` key and its only job (`check-dist`) also has no job-level `permissions:` key. This means the workflow runs with GitHub's default permissions, which include `contents: write` and other broad scopes. A minimal `permissions:` block (e.g. `contents: read`) should be added.

Locations:

- `.github/workflows/check-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script-injection in .github/workflows/test.yml: moved all ${{ steps.setup.outputs.* }} expressions (redis-port, redis-unix-socket, redis-tls-port, redis-tls-dir) from inline run: shell strings to env: blocks, referencing them as $REDIS_PORT, $REDIS_UNIX_SOCKET, $REDIS_TLS_PORT, and $REDIS_TLS_DIR respectively. Fixed missing-permissions in .github/workflows/check-dist.yml: added top-level `permissions: contents: read` block to restrict the workflow to minimal required permissions.

