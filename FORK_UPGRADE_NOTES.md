# Fork Upgrade Notes for Local Hermes Runtime

_Date: 2026-04-23_

This note records the Hermes lane of the fork upgrade program.

Primary plan: `codysumpter-cloud/prismtek-apps/docs/plans/2026-04-23-fork-upgrade-program.md`

## Purpose

Use owned forks as donors to strengthen the MacBook Hermes runtime without turning Hermes into a second product shell.

## Primary donor queue

- `agentmemory` — MCP/plugin memory integration path
- `openclaw` — gateway, node/device, voice/canvas donor pass
- `learn-hermes-agent` — clean subsystem references for loop, tools, sessions, memory, skills, scheduler, MCP
- `hermes-workspace` — workspace/runtime posture donor pass
- `hermes-desktop` / `hermes-webui` — control-surface ideas that actually improve the Mac runtime

## Guardrails

- Keep Hermes the runtime base
- Only port patterns that improve local runtime behavior
- Push shared contracts/glue into `bmo-stack` when they are not Hermes-local
- Do not absorb unrelated Buddy product-surface responsibilities here

## First slices

- [ ] Evaluate `agentmemory` MCP/plugin integration path
- [ ] Evaluate which `openclaw` gateway/device patterns belong here vs in `bmo-stack`
- [ ] Use `learn-hermes-agent` as the clean-room reference while porting subsystems
