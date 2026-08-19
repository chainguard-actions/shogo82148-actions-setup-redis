<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-redis/v1.50.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-redis/v1.50.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `run:` blocks in test.yml directly interpolate `${{ steps.setup.outputs.* }}` expressions inside shell commands. These `steps.*` context values are substituted by the YAML template engine before the shell processes them, allowing any value containing shell metacharacters to be interpreted as shell code. Offending lines include:
- `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-port }}" ping`
- `run: redis-cli -s "${{ steps.setup.outputs.redis-unix-socket }}" ping`
- `redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-tls-port }}" ...`
- `--cert "${{ steps.setup.outputs.redis-tls-dir }}/redis.crt" ...`
Fix: move the values into `env:` variables and reference them as quoted shell variables (e.g. `"$REDIS_PORT"`).

Locations:

- `.github/workflows/test.yml:55`
- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:100`

### missing-permissions (severity: medium)

The workflow file `check-dist.yml` has no top-level `permissions:` key and its only job (`check-dist`) also has no job-level `permissions:` key. Without explicit permissions, the job inherits the default repository token permissions, which may be overly broad (write access to contents and other scopes). Add a `permissions:` block with the minimum required scopes (e.g. `contents: read`).

Locations:

- `.github/workflows/check-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script-injection in .github/workflows/test.yml by moving all ${{ steps.setup.outputs.* }} expressions into env: blocks for three steps: 'connect via tcp port' (REDIS_PORT), 'connect via unix domain socket' (REDIS_UNIX_SOCKET), and 'connect via tls' (REDIS_TLS_PORT, REDIS_TLS_DIR). Fixed missing-permissions in .github/workflows/check-dist.yml by adding a top-level `permissions: contents: read` block.

