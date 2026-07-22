<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-redis/v1.53.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-redis/v1.53.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in test.yml directly interpolate ${{ steps.setup.outputs.* }} expressions into shell commands. These are steps.*.outputs.* context values — a workflow-controllable source — and are interpolated before the shell sees them, enabling script injection. Offending lines include:
- `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-port }}" ping` (sub-rule a)
- `run: redis-cli -s "${{ steps.setup.outputs.redis-unix-socket }}" ping` (sub-rule a)
- `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-tls-port }}" ...` (sub-rule a)
- `--cert "${{ steps.setup.outputs.redis-tls-dir }}/redis.crt"` and similar (sub-rule a)
These should be moved to env: vars and the env vars double-quoted in the shell script.

Locations:

- `.github/workflows/test.yml:51`
- `.github/workflows/test.yml:53`
- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:115`
- `.github/workflows/test.yml:116`
- `.github/workflows/test.yml:117`

### missing-permissions (severity: medium)

The workflow file check-dist.yml has no top-level permissions: key and the single job (check-dist) also has no job-level permissions: key. Without explicit permissions, the job inherits the default repository permissions (which can be write-all for older repositories), violating the principle of least privilege. A permissions: block with minimal scopes (e.g. contents: read) should be added.

Locations:

- `.github/workflows/check-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

1. Fixed script-injection in .github/workflows/test.yml: Moved all ${{ steps.setup.outputs.* }} expressions out of run: shell strings and into env: blocks for three steps: 'connect via tcp port' (REDIS_PORT), 'connect via unix domain socket' (REDIS_UNIX_SOCKET), and 'connect via tls' (REDIS_TLS_PORT, REDIS_TLS_DIR). Shell scripts now reference plain env vars. 2. Fixed missing-permissions in .github/workflows/check-dist.yml: Added top-level `permissions: contents: read` block to enforce least privilege.

