<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-redis/v1.55.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-redis/v1.55.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### permissions (severity: medium)

missing-permissions: The workflow file check-dist.yml has no top-level `permissions:` key and the only job (`check-dist`) also has no job-level `permissions:` key. This means the workflow runs with GitHub's default token permissions, which include write access to repository contents and other scopes.

Locations:

- `.github/workflows/check-dist.yml:1`

### script-injection (severity: high)

Rule (a) violation: `${{ steps.setup.outputs.* }}` expressions are interpolated directly inside `run:` shell command strings. The values `steps.setup.outputs.redis-port`, `steps.setup.outputs.redis-unix-socket`, `steps.setup.outputs.redis-tls-port`, and `steps.setup.outputs.redis-tls-dir` are all workflow-controllable step outputs that flow through YAML template substitution before the shell sees them. An attacker who can influence these output values (e.g. via a compromised action or supply-chain attack) could inject shell metacharacters. Offending lines:
- `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-port }}" ping`
- `run: redis-cli -s "${{ steps.setup.outputs.redis-unix-socket }}" ping`
- `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-tls-port }}" ...`
- `--cert "${{ steps.setup.outputs.redis-tls-dir }}/redis.crt"` (and key/cacert variants)
Fix: move outputs into `env:` variables and reference them as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/test.yml:55`
- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:108`

## Iteration Notes

### Iteration 1

**Fixes applied:** permissions, script-injection

**Notes:**

1. check-dist.yml: Added top-level `permissions: contents: read` block to restrict the workflow from running with default (overly broad) GitHub token permissions.
2. test.yml: Fixed all three script-injection locations by moving `${{ steps.setup.outputs.redis-port }}`, `${{ steps.setup.outputs.redis-unix-socket }}`, `${{ steps.setup.outputs.redis-tls-port }}`, and `${{ steps.setup.outputs.redis-tls-dir }}` expressions out of `run:` shell strings and into `env:` blocks, then referencing them as plain environment variables (`$REDIS_PORT`, `$REDIS_UNIX_SOCKET`, `$REDIS_TLS_PORT`, `$REDIS_TLS_DIR`) in the shell scripts.

