<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-redis/v1.52.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-redis/v1.52.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### missing-permissions (severity: medium)

The workflow file check-dist.yml has no top-level `permissions:` key and its only job (`check-dist`) also has no job-level `permissions:` block. Without explicit permissions, the job inherits the default repository permissions, which may be overly broad (e.g., write access to contents). A minimal permissions block (e.g., `contents: read`) should be added at the top level or job level.

Locations:

- `.github/workflows/check-dist.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in test.yml directly interpolate `${{ steps.setup.outputs.* }}` expressions (sub-rule a). These values flow through YAML template substitution before the shell sees them, allowing an attacker who can influence step outputs to inject shell metacharacters. Affected steps:
- 'connect via tcp port': `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-port }}" ping`
- 'connect via unix domain socket': `run: redis-cli -s "${{ steps.setup.outputs.redis-unix-socket }}" ping`
- 'connect via tls': `run: redis-cli ... -p "${{ steps.setup.outputs.redis-tls-port }}" ... --cert "${{ steps.setup.outputs.redis-tls-dir }}/redis.crt" ...`
Fix: move the outputs into `env:` variables and reference them as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/test.yml:56`
- `.github/workflows/test.yml:58`
- `.github/workflows/test.yml:107`

## Iteration Notes

### Iteration 1

**Fixes applied:** missing-permissions, script-injection

**Notes:**

1. check-dist.yml: Added top-level `permissions: contents: read` block to restrict the job to read-only repository access.
2. test.yml: Fixed three script-injection vulnerabilities by moving `${{ steps.setup.outputs.* }}` expressions into `env:` blocks and referencing them as plain shell variables ($REDIS_PORT, $REDIS_UNIX_SOCKET, $REDIS_TLS_PORT, $REDIS_TLS_DIR) in the run: scripts.

