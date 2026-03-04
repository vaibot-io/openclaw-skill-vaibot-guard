---
name: vaibot-guard
description: Localhost policy guard + tamper-evident audit log for OpenClaw. Uses VAIBOT_GUARD_TOKEN; optional VAIBOT_API_KEY anchoring. Opt-in user-level install files.
---

# VAIBot-Guard (OpenClaw Skill)

VAIBot-Guard is a **local** policy decision service that can gate OpenClaw operations and write a tamper-evident audit trail under `.vaibot-guard/`.

## Read this first (credentials + side effects)

### Sensitive credentials (treat as secrets)

- `VAIBOT_GUARD_TOKEN` — bearer token for Guard endpoints (recommended)
- `VAIBOT_API_KEY` — bearer token used to anchor receipts to VAIBot `/prove` (optional)

### Persistent changes this skill may make (opt-in)

Depending on which commands you choose to run, VAIBot-Guard may:

- write an env file: `~/.config/vaibot-guard/vaibot-guard.env` (recommended `chmod 600`)
- generate a **systemd user** unit: `~/.config/systemd/user/vaibot-guard.service`
- write local logs/state: `${VAIBOT_GUARD_LOG_DIR}` (default: `${VAIBOT_WORKSPACE}/.vaibot-guard`)

If you want **zero persistence**, run the service in the foreground with env vars set in your shell (see Manual Quick Start below).

## HTTP API

- `GET /health`
- `POST /v1/decide/exec` + `POST /v1/finalize` (shell exec flows)
- `POST /v1/decide/tool` + `POST /v1/finalize/tool` (generic tool gating; used by OpenClaw bridge plugin)
- `POST /v1/flush` (checkpoint flush)
- `POST /api/proof` (Merkle inclusion proofs)

Auth:
- If `VAIBOT_GUARD_TOKEN` is set, protected endpoints require `Authorization: Bearer <token>`.

## Manual Quick Start (recommended for evaluation)

This path avoids installing systemd units or writing config files.

1) From this directory, set env and start the service:

```bash
export VAIBOT_GUARD_HOST=127.0.0.1
export VAIBOT_GUARD_PORT=39111
export VAIBOT_POLICY_PATH=references/policy.default.json
export VAIBOT_WORKSPACE="$(pwd)"
export VAIBOT_GUARD_LOG_DIR="$VAIBOT_WORKSPACE/.vaibot-guard"

# Recommended: set a token so endpoints require auth
export VAIBOT_GUARD_TOKEN="<generate-a-random-token>"

node scripts/vaibot-guard-service.mjs
```

2) Smoke test:

```bash
curl -s http://127.0.0.1:39111/health
```

3) Try a decision:

```bash
node scripts/vaibot-guard.mjs precheck --intent '{"tool":"system.run","action":"exec","command":"/bin/echo","cwd":".","args":["hello"]}'
```

## Optional: user-level persistence (systemd user service)

If you want Guard to run continuously, use the installer helper:

```bash
node scripts/vaibot-guard.mjs install-local
```

This **will**:
- create/update `~/.config/vaibot-guard/vaibot-guard.env` (chmod 600)
- generate `VAIBOT_GUARD_TOKEN` if missing
- generate `~/.config/systemd/user/vaibot-guard.service`

Then:

```bash
systemctl --user daemon-reload
systemctl --user enable --now vaibot-guard
systemctl --user status vaibot-guard --no-pager
```

## OpenClaw enforcement (recommended)

For real enforcement (intercepting tool calls regardless of what the model tries), use the **Gateway plugin bridge**.

See:
- `references/openclaw-bridge.md`

## Configuration reference

Common env vars:

- `VAIBOT_GUARD_HOST` (default `127.0.0.1`)
- `VAIBOT_GUARD_PORT` (default `39111`)
- `VAIBOT_GUARD_TOKEN` (recommended)
- `VAIBOT_POLICY_PATH` (default `references/policy.default.json`)
- `VAIBOT_WORKSPACE` (default `process.cwd()`)
- `VAIBOT_GUARD_LOG_DIR` (default `${VAIBOT_WORKSPACE}/.vaibot-guard`)

Anchoring/proving (optional):

- `VAIBOT_API_URL` (e.g. `https://www.vaibot.io/api`)
- `VAIBOT_API_KEY` (only required when `VAIBOT_PROVE_MODE=required`)
- `VAIBOT_PROVE_MODE` (`off|best-effort|required`, default `best-effort`)

If `VAIBOT_PROVE_MODE=required`, Guard becomes **fail-closed** when it cannot anchor proofs.

## More docs

- `README.md`
- `SECURITY.md`
- `references/persistence-and-install.md`
- `references/ops-runbook.md`
