<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-redis/v1.52.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-redis/v1.52.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions from `steps.*.outputs.*` context are interpolated directly inside `run:` shell command strings. In the `test` job, `${{ steps.setup.outputs.redis-port }}` and `${{ steps.setup.outputs.redis-unix-socket }}` are embedded directly in shell commands. In the `test-tls` job, `${{ steps.setup.outputs.redis-tls-port }}` and `${{ steps.setup.outputs.redis-tls-dir }}` are embedded in a multi-line shell command. These values flow through YAML template substitution before the shell sees them, enabling command injection if the action outputs contain shell metacharacters.

Locations:

- `.github/workflows/test.yml:62`
- `.github/workflows/test.yml:64`
- `.github/workflows/test.yml:108`

### missing-permissions (severity: medium)

The workflow file `check-dist.yml` has no top-level `permissions:` key and the only job (`check-dist`) also has no job-level `permissions:` key. Without explicit permissions, the job inherits the default repository token permissions, which may be overly broad (write access to contents and other scopes depending on repository settings).

Locations:

- `.github/workflows/check-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script-injection in .github/workflows/test.yml by moving all `${{ steps.setup.outputs.* }}` expressions out of run: shell strings and into step-level env: blocks (REDIS_PORT, REDIS_UNIX_SOCKET, REDIS_TLS_PORT, REDIS_TLS_DIR), then referencing them as plain environment variables in the shell commands. Fixed missing-permissions in .github/workflows/check-dist.yml by adding `permissions: contents: read` to the check-dist job.

