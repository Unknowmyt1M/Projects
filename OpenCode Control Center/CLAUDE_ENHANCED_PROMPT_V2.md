# OpenCode Control Center — Claude Enhanced Master Prompt v2.0

> Source-preserved companion document supplied by the project owner on 2026-08-20.
> This document is the Claude-enhanced baseline that the final v3 implementation prompt builds upon.

## Core Principle

Human is the final authority. ChatGPT is the planner/reviewer/reasoning layer. OpenCode is the execution/coding layer. Control Center is the auditable control plane connecting them.

This is not a toy demo or simple prompt relay. It is a local-first developer product with strong process control, explicit permissions, reliable state, recoverability, and messaging-first UX.

## Non-negotiables from the supplied prompt

1. Human control is mandatory; no AI may bypass human-defined permissions or approvals.
2. Never invent OpenCode APIs; inspect the installed version and official documentation first.
3. Detect the installed OpenCode generation/version and isolate version-specific adapters.
4. Verify and implement against the current MCP specification/SDK rather than assuming an older transport/session model.
5. Never assume ChatGPT can directly reach localhost; use a secure supported remote MCP connectivity strategy.
6. The dashboard is the primary control surface and must be a real-time messaging application, not merely an admin panel.
7. Pause and process control are separate. Pausing an agent must not automatically kill user-approved long-running processes.
8. Every spawned/associated process needs ownership metadata so task-specific process trees can be controlled safely.
9. Destructive/high-impact actions require explicit policy handling.
10. Crash recovery must preserve tasks, questions, approvals, process ownership, and conversation history.
11. Secrets must never be stored in plaintext logs/history/browser storage/Git.
12. The system must remain useful if ChatGPT disconnects.
13. The system must remain useful if OpenCode disconnects.
14. Never fabricate progress or agent messages.
15. Local-first modular-monolith architecture is preferred for the single-developer Windows use case.

## Architecture

```text
Human Operator
      │
      ▼
Messaging UI / Dashboard
      │
      ▼
OCC Control Hub
 ┌────┼───────────────┐
 ▼    ▼               ▼
OpenCode Adapter   MCP Server   Process Supervisor
 │                    │
 ▼                    ▼
OpenCode Agent     ChatGPT MCP Client
```

Communication planes remain separate:

- Human/UI ↔ OCC: HTTP + realtime browser transport.
- OCC ↔ OpenCode: version-adapted plugin/adapter transport.
- ChatGPT MCP client ↔ OCC MCP server: MCP only.

The browser never talks directly to OpenCode. OCC must never impersonate ChatGPT, and OpenCode must never impersonate ChatGPT.

## Messaging-first UX

The main UI is a group-chat-style application where:

- 👤 Human/Aditya
- 🤖 ChatGPT
- 🧑‍💻 OpenCode
- ⚙ System

appear as distinct participants in a durable conversation.

The UI includes:

- conversation list
- message stream
- target selector (OpenCode / ChatGPT / Both / Control Center)
- task context panel
- question/decision cards
- approval cards
- tool/process cards
- shell-output cards
- task controls
- process controls
- reconnect/recovery states
- mobile responsive layout

## Bidirectional collaboration

### Human → OpenCode

User sends an instruction through OCC. OCC authenticates/authorizes it, persists it, routes it to the OpenCode adapter, and displays delivery/execution events.

### ChatGPT → OpenCode

ChatGPT uses MCP control-plane tools to create/inspect/control tasks according to human policy.

### OpenCode → ChatGPT

OpenCode emits a structured question/event to OCC. OCC persists it. ChatGPT retrieves the pending work through MCP and submits a recommendation. If human approval is required, the recommendation is only a recommendation until the human approves/edits/rejects it.

### Human → ChatGPT

The UI can send a message explicitly targeted at ChatGPT. The message is persisted as a human-authored message and becomes available to ChatGPT through the supported MCP context mechanism.

## Process-control model

Pause, Stop Agent, Kill Task Processes, and Emergency Stop are distinct operations.

Every process must track:

- stable OCC process ID
- PID
- parent/root IDs
- task ID
- session ID
- owner/creator
- start time
- command (with secret redaction)
- status
- exit code
- termination state

On Windows, prefer Job Objects or another robust ownership mechanism. Never kill by executable name alone. Never implement task-specific stop as `taskkill /IM node.exe /F`.

Support:

- parent exits while child survives
- grandchildren/forked children
- PID reuse
- shell wrappers such as cmd, PowerShell, npm, pnpm, yarn, bun, Python
- long-running dev servers
- interactive stdin commands
- commands waiting for confirmation
- huge output
- process termination timeouts
- already-exited processes
- orphan reconciliation

## Recovery

Persist durable task, session, message, event, question, approval, policy, and process state. Browser reconnect must replay missed events. OpenCode reconnect must reconcile session/process state. MCP reconnect must retrieve pending work without duplicating answers or tasks.

Use idempotency keys, monotonic event sequence numbers, correlation IDs, stale-action checks, and deterministic reconciliation.

## Acceptance scenarios from the supplied prompt

1. Human starts task → OpenCode executes → progress visible.
2. OpenCode question → ChatGPT recommendation → Human approves.
3. Pause agent while npm dev runs → process survives.
4. Stop agent → user chooses owned-process termination → unrelated processes survive.
5. Multiple tasks do not cross-contaminate.
6. Backend crash → restart → state recovers.
7. ChatGPT disconnects → questions queue → ChatGPT answers after reconnect.
8. Malicious shell output cannot override policy.

The supplied Claude prompt also requires Windows installation/setup, OpenCode plugin integration documentation, MCP/ChatGPT connection documentation, permission explanation, process-control explanation, recovery guide, security threat model, known limitations, future roadmap, and end-to-end tests.

## North-star

> “I am the operator. ChatGPT and OpenCode are powerful workers. I can see what they are doing, decide what they are allowed to do, answer their questions, control their sessions, and safely control every process they own.”

The system must feel like an operator-controlled collaboration workspace, not an autonomous system where the human merely watches.
