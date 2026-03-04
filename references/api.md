# VAIBot-Guard API (v1)

This document describes the VAIBot-Guard HTTP API endpoints used by:
- the `vaibot-guard` CLI wrapper, and
- the OpenClaw Gateway bridge plugin.

> Security model: VAIBot-Guard is intended to run on **localhost** (`127.0.0.1`).
> If `VAIBOT_GUARD_TOKEN` is set, protected endpoints require `Authorization: Bearer <token>`.

## Common response envelope

Most endpoints return a common envelope:

```json
{
  "ok": true,
  "runId": "run_<uuid>",
  "risk": { "risk": "low|high", "reason": "..." },
  "decision": { "decision": "allow|deny|approve", "reason": "...", "approvalId": "appr_<uuid>" },
  "audit": { "eventId": "<uuid>", "hash": "...", "prevHash": "..." },
  "prove": { "ok": true }
}
```

Notes:
- `prove` is best-effort unless `VAIBOT_PROVE_MODE=required`.
- `audit` is an append-only, hash-chained event reference; exact fields may evolve.

## Health

### `GET /health`

Returns:

```json
{ "ok": true, "service": "vaibot-guard", "ts": "<iso8601>" }
```

## Exec flow (shell commands)

### `POST /v1/decide/exec`

Precheck a shell command.

Request body:

```json
{
  "sessionId": "<string>",
  "cmd": "/bin/echo",
  "args": ["hello"],
  "intent": {
    "cwd": ".",
    "files": { "read": [], "write": [], "delete": [] },
    "network": { "destinations": [] },
    "env_keys": []
  }
}
```

Response:
- Common envelope with `decision`.

### `POST /v1/finalize`

Finalize an exec run.

Request body:

```json
{
  "sessionId": "<string>",
  "runId": "run_<uuid>",
  "result": {
    "exitCode": 0,
    "stdout": "...",
    "stderr": "..."
  }
}
```

Response:
- `{ ok: true, audit, prove }` or an error when `VAIBOT_PROVE_MODE=required`.

## Tool flow (generic OpenClaw tools)

These endpoints are used by the OpenClaw Gateway bridge plugin that intercepts tool calls.

### `POST /v1/decide/tool`

Precheck a generic tool call.

Request body:

```json
{
  "sessionId": "<string>",
  "agentId": "<optional>",
  "channelId": "<optional>",
  "toolName": "read|write|exec|message.send|...",
  "runId": "<optional openclaw run id>",
  "toolCallId": "<optional provider tool call id>",
  "workspaceDir": "<optional>" ,
  "params": { "...": "..." }
}
```

Response:
- Common envelope with `decision`.

### `POST /v1/finalize/tool`

Finalize a tool call (best-effort audit).

Request body:

```json
{
  "sessionId": "<string>",
  "runId": "run_<uuid>",
  "toolName": "<string>",
  "toolCallId": "<optional>",
  "runIdOpenClaw": "<optional>",
  "params": { "...": "..." },
  "result": { "ok": true, "durationMs": 12, "result": "..." }
}
```

Response:
- `{ ok: true, audit, prove }` (or error on required prove failure).

## Ops endpoints

### `POST /v1/flush`

Attempt to flush/anchor checkpoints for a session.

```json
{ "sessionId": "<string>" }
```

### `POST /api/proof`

Compute an inclusion proof for a given leaf index at a checkpoint sequence.

```json
{ "sessionId": "<string>", "index": 0, "checkpointSeq": 1 }
```
