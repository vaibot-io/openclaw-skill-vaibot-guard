# OpenClaw integration (bridge plugin)

VAIBot-Guard can be used as a local decision service for OpenClaw.

## Recommended enforcement model

For real enforcement (not just “model follows instructions”), use an **in-process OpenClaw Gateway plugin** that intercepts tool calls via plugin hooks (`before_tool_call` / `after_tool_call`) and asks VAIBot-Guard for a decision.

This repo’s companion plugin project:
- `packages/openclaw/vaibot-guard-bridge/`

## What the bridge does

- Intercepts **all tools** by default.
- Calls Guard for a decision:
  - `POST /v1/decide/tool`
- Best-effort finalize/audit:
  - `POST /v1/finalize/tool`

## Notes on UX

Even with intercept-all, policy defaults should auto-allow low-risk operations and only gate high-risk actions (exec, writes, outbound messaging, remote runs, non-allowlisted network).
