# Vexum Core — architecture

Vexum Core is a self-hostable **meeting-intelligence runtime**: bots join meetings, capture and
transcribe them in real time, and sandboxed agents turn transcripts into governed knowledge and
actions. This document is the entry point to the system's design; the deeper, always-current
sources it indexes are listed at the end.

## Actors

| Actor | Role |
|---|---|
| **User** | requests bots, reads live transcripts/cards, chats with their agent over their workspace |
| **Operator** | deploys the stack (Docker Compose / k8s), holds admin + provider credentials |
| **Meeting platform** | Google Meet / Teams / Zoom — the external system a bot joins |
| **Bot** | a headless browser workload that joins the meeting and captures media |
| **Transcription service** | STT unit (self-hosted or hosted) turning audio into `transcript.v1` segments |
| **Agent worker** | an isolated, per-dispatch container that runs LLM turns over a mounted user workspace |
| **LLM provider / agent harness** | reached ONLY through the provider-agnostic `core/agent/llm` ports (completion adapters + CLI-harness adapters) |

## The five domains (`core/`)

`core/` is the runnable platform; each domain is a bounded context that publishes versioned
contracts (`core/<domain>/contracts/*.vN`) and never reaches into a sibling's internals:

| Domain | Role | Key contracts |
|---|---|---|
| [`core/runtime`](core/runtime/) | the kernel — spawn/execute workloads (process · docker · k8s), mount workspaces | `runtime.v1`, `schedule.v1` |
| [`core/meetings`](core/meetings/) | capture — meeting-api, bot lifecycle, transcription | `transcript.v1`, `lifecycle.v1`, `acts.v1`, … |
| [`core/agent`](core/agent/) | execution — one `unit.v1` dispatcher; sandboxed workers with scoped identity over `workspace.v1` git repos | `unit.v1`, `workspace.v1`, `routine.v1`, `tool.v1`, … |
| [`core/identity`](core/identity/) | authN/authZ — accounts, per-dispatch signed tokens, `canAccess` | `identity.v1` |
| [`core/gateway`](core/gateway/) | the edge — auth, routing, WS fan-out | `api.v1`, `ws.v1`, `logevent.v1` |

## Core actions (how the actors meet)

1. **Capture**: user → gateway → meeting-api dispatches a bot (via `runtime.v1`); the bot joins the
   platform meeting; audio → transcription → `transcript.v1` segments on the bus.
2. **Live copilot**: a transcript stream dispatches an agent worker; each beat polishes the window
   and surfaces entity cards through a direct LLM completion (`CompletionPort`).
3. **Governed action**: chat / routines / post-meeting turns run a CLI agent harness
   (`HarnessPort`, selected by `VEXA_RUNNER`) over the user's mounted `workspace.v1` git repo;
   writes are committed, streamed as UnitEvents, and fanned out via `ws.v1`.
4. **Isolation is the enforcement**: agents never run in the control plane — every turn executes in
   a runtime-spawned container carrying a per-dispatch signed identity token.

## Architecture-as-code (the machine-checked truth)

The chart is not prose — it is data, validated in CI:

- [`architecture.calm.json`](architecture.calm.json) — the FINOS **CALM** model of the system,
  validated by `pnpm gate:calm` against [`calm/patterns/meeting-intelligence.pattern.json`](calm/patterns/)
  with reusable control requirements in [`calm/controls/`](calm/controls/).
- [`contracts.seal.json`](contracts.seal.json) + `pnpm gate:schema` / `gate:contract-version` —
  contract goldens ≡ schema, sealed hashes frozen.
- [`.dependency-cruiser.cjs`](.dependency-cruiser.cjs) + the gates in [`scripts/gates.mjs`](scripts/)
  — module boundaries, isolation, and the dependency DAG enforced on every push.

## Deeper design documentation

- [`docs/docs/architecture/`](docs/docs/architecture/) — dispatch, execution, governance,
  identity-and-trust, modules, streaming, architecture-as-code (published at
  https://docs.core.vexa.ai).
- [`core/README.md`](core/README.md) — the domain layout doctrine (bounded contexts, seams).
- Per-domain READMEs (e.g. [`core/agent/README.md`](core/agent/README.md)) — purpose, boundary,
  seams, contracts, and evaluation ladder for each domain (including the provider-agnostic
  LLM/harness ports the agent worker drives).
- [`deploy/compose/README.md`](deploy/compose/README.md) — the reference deployment.
