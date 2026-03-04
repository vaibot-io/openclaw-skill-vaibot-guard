# SECURITY.md — VAIBot-Guard

This document describes **what VAIBot-Guard does**, the **persistent changes** it may make, and how to run it safely.

## What VAIBot-Guard is

VAIBot-Guard is a **local** policy decision service that can gate OpenClaw tool execution (and record decisions/results) while producing a tamper-evident audit trail.

## Defaults / threat model

- Intended to run on **localhost** (`127.0.0.1`) and be called by local clients (OpenClaw Gateway / local CLI).
- By default, it writes audit data to a local directory under your workspace: `.vaibot-guard/`.

## Sensitive credentials

VAIBot-Guard may use the following sensitive values:

- `VAIBOT_GUARD_TOKEN` — bearer token to authenticate to Guard endpoints (recommended).
- `VAIBOT_API_KEY` — bearer token used to anchor receipts/checkpoints to VAIBot `/prove` (optional).

Treat both as secrets.

## Persistent files / side effects

Depending on how you install/run it, VAIBot-Guard may:

- Write an env file:
  - `~/.config/vaibot-guard/vaibot-guard.env` (recommended permissions: `0600`)
- Create a **systemd user** unit:
  - `~/.config/systemd/user/vaibot-guard.service`
- Write runtime logs/state:
  - `${VAIBOT_GUARD_LOG_DIR}` (default `${VAIBOT_WORKSPACE}/.vaibot-guard`)

If you do not want any of these persistent changes, run the service in the foreground with environment variables set in your shell (see README).

## Network egress

VAIBot-Guard is local-first.

- If `VAIBOT_API_URL` and `VAIBOT_API_KEY` are **not** set, Guard will not attempt to anchor receipts.
- If anchoring is enabled, Guard will send **receipt payloads** to the configured VAIBot endpoint.

Make sure you trust the configured `VAIBOT_API_URL` before providing `VAIBOT_API_KEY`.

## Safe operation checklist

- Keep Guard bound to localhost unless you have a strong reason.
- Set `VAIBOT_GUARD_TOKEN` and require auth on decision endpoints.
- Store env files with `chmod 600`.
- Add `.vaibot-guard/` to `.gitignore`.
- If using `VAIBOT_PROVE_MODE=required`, understand it is **fail-closed** when anchoring fails.
