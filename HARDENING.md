<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-redis/v1.54.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-redis/v1.54.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three `run:` steps in test.yml directly interpolate `${{ steps.setup.outputs.* }}` expressions inside shell command strings. These expressions flow through YAML template substitution before the shell parses them, allowing an attacker who can influence step outputs to inject arbitrary shell commands.

Offending lines:
- Line 63: `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-port }}" ping`
- Line 65: `run: redis-cli -s "${{ steps.setup.outputs.redis-unix-socket }}" ping`
- Line 126: `redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-tls-port }}" \`
- Line 128: `--cert "${{ steps.setup.outputs.redis-tls-dir }}/redis.crt" \`
- Line 129: `--key "${{ steps.setup.outputs.redis-tls-dir }}/redis.key" \`
- Line 130: `--cacert "${{ steps.setup.outputs.redis-tls-dir }}/ca.crt" \`

Fix: Move the step outputs into `env:` variables and reference them as quoted shell variables (e.g., `"$REDIS_PORT"`) instead of interpolating `${{ ... }}` directly in the `run:` script.

Locations:

- `.github/workflows/test.yml:63`
- `.github/workflows/test.yml:65`
- `.github/workflows/test.yml:126`

### missing-permissions (severity: medium)

The workflow file `check-dist.yml` has no top-level `permissions:` key and its only job (`check-dist`) also has no job-level `permissions:` key. This means the workflow runs with GitHub's default permissions, which include `contents: write` on push events and other broad grants. A minimal explicit permissions block (e.g., `contents: read`) should be added to the job or at the top level.

Locations:

- `.github/workflows/check-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

1. Fixed script-injection in .github/workflows/test.yml: Moved all ${{ steps.setup.outputs.* }} expressions out of run: shell strings and into env: blocks. The three affected steps now use $REDIS_PORT, $REDIS_UNIX_SOCKET, $REDIS_TLS_PORT, and $REDIS_TLS_DIR as plain shell variables. 2. Fixed missing-permissions in .github/workflows/check-dist.yml: Added `permissions: contents: read` at the job level for the check-dist job, replacing the implicit broad default permissions.

