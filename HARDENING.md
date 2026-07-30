<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-redis/v1.56.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-redis/v1.56.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### missing-permissions (severity: medium)

The workflow file check-dist.yml has no top-level `permissions:` key and the single job `check-dist` also has no job-level `permissions:` key. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially write) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/check-dist.yml:1`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in test.yml directly interpolate `${{ steps.setup.outputs.* }}` expressions inside shell command strings. The `steps.*.outputs.*` context flows through YAML template substitution before the shell processes it, allowing a compromised or malicious action step to inject arbitrary shell commands. Offending lines:
- Line 63: `run: redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-port }}" ping`
- Line 65: `run: redis-cli -s "${{ steps.setup.outputs.redis-unix-socket }}" ping`
- Line 109: `redis-cli -h 127.0.0.1 -p "${{ steps.setup.outputs.redis-tls-port }}" \`
- Lines 111-113: `--cert "${{ steps.setup.outputs.redis-tls-dir }}/redis.crt"`, `--key "${{ steps.setup.outputs.redis-tls-dir }}/redis.key"`, `--cacert "${{ steps.setup.outputs.redis-tls-dir }}/ca.crt"`
These should be moved to `env:` variables and referenced as `"$ENV_VAR"` in the shell.

Locations:

- `.github/workflows/test.yml:63`
- `.github/workflows/test.yml:65`
- `.github/workflows/test.yml:109`
- `.github/workflows/test.yml:111`
- `.github/workflows/test.yml:112`
- `.github/workflows/test.yml:113`

## Iteration Notes

### Iteration 1

**Fixes applied:** missing-permissions, script-injection

**Notes:**

1. check-dist.yml: Added top-level `permissions: contents: read` block before the jobs section to enforce least-privilege for the GITHUB_TOKEN.
2. test.yml: Fixed 6 script-injection locations across two steps:
   - 'connect via tcp port': moved `${{ steps.setup.outputs.redis-port }}` to env var REDIS_PORT
   - 'connect via unix domain socket': moved `${{ steps.setup.outputs.redis-unix-socket }}` to env var REDIS_UNIX_SOCKET
   - 'connect via tls': moved `${{ steps.setup.outputs.redis-tls-port }}` to REDIS_TLS_PORT and `${{ steps.setup.outputs.redis-tls-dir }}` to REDIS_TLS_DIR, then referenced them as plain env vars in the shell script.

