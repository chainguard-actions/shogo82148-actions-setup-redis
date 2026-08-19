<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-redis/v1.49.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-redis/v1.49.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: `${{ steps.setup.outputs.* }}` expressions are interpolated directly inside `run:` shell command strings in test.yml. These are `steps.*.outputs.*` context values which flow through YAML template substitution before the shell sees them, enabling script injection. Affected steps:
- 'connect via tcp port': `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-port }}" ping`
- 'connect via unix domain socket': `run: redis-cli -s "${{ steps.setup.outputs.redis-unix-socket }}" ping`
- 'connect via tls': `run: |` block using `${{ steps.setup.outputs.redis-tls-port }}` and `${{ steps.setup.outputs.redis-tls-dir }}` (multiple times)
Fix: move the values into `env:` variables and reference them as double-quoted shell variables (e.g., `"$REDIS_PORT"`).

Locations:

- `.github/workflows/test.yml:54`
- `.github/workflows/test.yml:56`
- `.github/workflows/test.yml:108`

### missing-permissions (severity: medium)

The workflow file `check-dist.yml` has no top-level `permissions:` key and its only job (`check-dist`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write-all in some configurations). Add a top-level `permissions:` block with the minimum required scopes (e.g., `contents: read`).

Locations:

- `.github/workflows/check-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

1. Fixed script-injection in test.yml: moved all `${{ steps.setup.outputs.* }}` expressions out of `run:` shell strings and into `env:` blocks for three steps: 'connect via tcp port' (REDIS_PORT), 'connect via unix domain socket' (REDIS_UNIX_SOCKET), and 'connect via tls' (REDIS_TLS_PORT, REDIS_TLS_DIR). Shell scripts now reference plain env vars. 2. Fixed missing-permissions in check-dist.yml: added top-level `permissions: contents: read` block before the `jobs:` key.

