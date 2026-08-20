# UI / MESSAGING APP SPEC — MANDATORY EXTENSION TO MASTER PROMPT

This document is a mandatory part of the OpenCode Control Center implementation. The product MUST NOT be implemented as a conventional admin dashboard only. Its primary interaction model is a **real-time messaging/workspace application**, visually similar to a modern group-chat app, where the Human, ChatGPT, and OpenCode appear as distinct participants in the same task conversation.

The Control Center is simultaneously:

1. A messaging interface.
2. A live AI-agent control console.
3. A task/session monitor.
4. A human approval/decision surface.
5. A process-control console.
6. An audit/history viewer.

## 1. PRIMARY UX PRINCIPLE

The main screen MUST feel like a messaging application, not a generic SaaS admin panel.

The central experience is a conversation such as:

```text
┌──────────────────────────────────────────────────────────────────────┐
│ Hermes • Task #42                 ● OpenCode   ● ChatGPT   🟢 Live │
├──────────────┬───────────────────────────────────────┬───────────────┤
│ Conversations│             MESSAGE STREAM            │ Task / Context│
│              │                                       │               │
│ ● Hermes #42 │ 👤 Aditya                             │ STATUS        │
│   Auth       │ Implement authentication for Hermes. │ RUNNING       │
│              │                                       │               │
│ ○ Hermes #41 │ 🤖 ChatGPT                            │ Progress      │
│   UI         │ I'll plan the implementation and     │ ███████░░ 73% │
│              │ coordinate with OpenCode.             │               │
│ ○ Nothing #7 │                                       │ Current tool  │
│              │ 🤖 OpenCode                           │ npm test      │
│              │ I found two authentication approaches│               │
│              │ and need a decision.                  │               │
│              │                                       │               │
│              │ 🤖 ChatGPT                            │               │
│              │ I recommend server-side sessions.     │               │
│              │                                       │               │
│              │ 👤 Aditya                             │               │
│              │ Go with sessions.                     │               │
│              │                                       │               │
│              │ ⚙ System                              │               │
│              │ OpenCode resumed execution.           │               │
│              │                                       │               │
│              │ [Type a message...]          [Send]   │               │
├──────────────┴───────────────────────────────────────┴───────────────┤
│ ⏸ Pause   ▶ Resume   ■ Stop   🔴 Kill Task Processes   ⋮ More       │
└──────────────────────────────────────────────────────────────────────┘
```

The exact visual design is up to the implementer, but the **messaging-first interaction model is non-negotiable**.

## 2. PARTICIPANTS / MESSAGE ROLES

The conversation must visually distinguish at minimum:

- 👤 Human operator — Aditya/user.
- 🤖 ChatGPT — planning/reasoning/review participant.
- 🧑‍💻 OpenCode — coding/execution agent.
- ⚙ System — Control Center lifecycle/security/event messages.
- 🔧 Tool/Process — optional technical event messages.

Never make ChatGPT and OpenCode appear as if they are the same agent.

Each message must have:

- message ID
- conversation/task ID
- sender type
- sender identity/name
- timestamp
- content
- message type
- correlation ID where applicable
- reply/thread reference where applicable
- status (sending/sent/failed/delivered if meaningful)
- attachments/artifacts metadata where applicable

## 3. MESSAGE TYPES

Support structured message types, not only plain text:

- text
- system_event
- task_update
- tool_call
- tool_result
- shell_started
- shell_output
- shell_finished
- file_changed
- permission_request
- approval_request
- question
- recommendation
- decision
- error
- warning
- progress
- process_state
- task_state
- code_diff
- artifact
- link/reference

These should render as appropriate message cards/components rather than dumping raw JSON into the chat.

## 4. LIVE STREAMING

The conversation MUST update in real time while OpenCode works.

Examples:

```text
🤖 OpenCode
Analyzing src/auth/...

⚙ System
Tool started: read

🤖 OpenCode
Found existing session middleware.

⚙ System
Tool started: bash
Command: npm test
PID: 18420

🔧 Process
npm test — running...

⚙ System
Tool finished: bash (exit code 1)

🤖 OpenCode
Tests failed in auth/session.test.ts. I need a decision...
```

Do not require the user to refresh the page.

Use a robust realtime transport such as WebSocket/SSE or the best current local equivalent. Reconnection and event replay are mandatory.

## 5. MESSAGE HISTORY / EVENT REPLAY

The UI must not lose messages when the browser disconnects.

On reconnect:

1. Authenticate/reconnect.
2. Determine the last acknowledged event/message ID.
3. Request missed events.
4. Replay them in order.
5. Reconcile with the current durable task/session state.
6. Avoid duplicate rendering using idempotent event IDs.

A conversation is durable data, not an in-memory UI stream.

## 6. THREADS / REPLIES

Support message replies or contextual threads for important events.

Example:

```text
🤖 OpenCode
Should I use SQLite or PostgreSQL?

  ↳ 🤖 ChatGPT
  Recommendation: SQLite.

  ↳ 👤 Aditya
  Approved.
```

A user must be able to jump from a decision to the original question/context.

## 7. QUESTIONS AS CHAT MESSAGES

OpenCode questions must appear directly inside the conversation, not only on a separate dashboard page.

Example:

```text
┌──────────────────────────────────────────────────────────┐
│ 🤖 OpenCode — DECISION REQUIRED                         │
│                                                          │
│ Should authentication use JWT or server-side sessions?  │
│                                                          │
│ 🤖 ChatGPT recommendation:                               │
│ Server-side sessions.                                    │
│                                                          │
│ [Approve] [Reject] [Edit Answer] [Ask ChatGPT Again]    │
│ [Answer Myself]                                          │
└──────────────────────────────────────────────────────────┘
```

The message card must show whether the recommendation is:

- informational;
- auto-approved by policy;
- waiting for human approval;
- rejected;
- superseded;
- expired;
- answered by human.

## 8. HUMAN OVERRIDES

The user must be able to override either AI at any time.

Example actions on a message/task:

- Reply.
- Correct ChatGPT.
- Correct OpenCode.
- Send instruction to OpenCode.
- Ask ChatGPT to review.
- Pause.
- Resume.
- Stop.
- Cancel.
- Approve.
- Reject.
- Retry.
- Request explanation.

Human instructions must be clearly labeled as authoritative operator instructions.

## 9. MESSAGE COMPOSER

The composer should support explicit target selection:

```text
Target: [OpenCode ▼]

or

Target: [ChatGPT ▼]

or

Target: [Both ▼]
```

Also support contextual actions such as:

- `Send to OpenCode`
- `Ask ChatGPT to review`
- `Ask ChatGPT about this error`
- `Tell OpenCode to continue`
- `Create task from message`

Do not make routing ambiguous.

## 10. TASKS AS CONVERSATIONS

Every task should have a primary conversation.

Do NOT separate task execution from communication into unrelated pages.

A task conversation should contain:

- original user request;
- ChatGPT planning;
- OpenCode execution messages;
- questions;
- approvals;
- tool activity summaries;
- process events;
- errors;
- decisions;
- completion summary;
- artifacts/diffs.

A task can have child threads for individual decisions or investigations.

## 11. LEFT SIDEBAR — CONVERSATION LIST

The left sidebar should behave like a messaging application's conversation list.

Each item should show:

- project name;
- task title;
- task number/short ID;
- last message preview;
- timestamp;
- status indicator;
- unread count;
- pending approval/question indicator;
- running/paused/stopped state.

Example:

```text
CONVERSATIONS

🟢 Hermes — Authentication
   OpenCode: Running tests...       2m

🔴 Nothing — OAuth fix
   OpenCode: Build failed           8m

🟡 TurboFlix — UI redesign
   Needs your approval             15m

✅ SmartClip — Clipboard tagging
   Completed                        1h
```

## 12. RIGHT CONTEXT PANEL

The right panel is contextual, not the primary interaction surface.

Show:

### Task
- status
- progress
- project
- branch/worktree
- creation time
- elapsed time
- current agent

### Session
- OpenCode session ID
- connection state
- agent/model if available
- last heartbeat

### Processes
- task-owned processes
- PID
- command
- state
- runtime
- actions

### Pending decisions
- questions
- approvals

### Files
- recently changed files
- diff summary

## 13. PROCESS CONTROL INSIDE THE CHAT

The shell-process edge case MUST be visible directly in the conversation.

Example:

```text
┌──────────────────────────────────────────────────────────┐
│ 🔧 OpenCode started shell command                       │
│                                                          │
│ npm run build                                             │
│ PID 18420 • 3 child processes • Running for 42s          │
│                                                          │
│ [View Output] [Stop Process] [Keep Running]             │
└──────────────────────────────────────────────────────────┘
```

If the user clicks Stop Agent while this command is active, show the process decision UI before terminating task-owned processes unless the current policy explicitly defines the action.

## 14. PAUSE / STOP / KILL UX

Never use one ambiguous “Stop” action for everything.

Primary controls:

```text
⏸ Pause Agent
▶ Resume Agent
■ Stop Agent
🛑 Stop Task Processes
☠ Kill Task
```

The UI must explain consequences.

Example:

```text
STOP AGENT

OpenCode will stop receiving/performing new work.

Currently running:
• npm run dev — PID 18420
• node server.js — PID 19284

What should happen to them?

○ Keep running
○ Stop selected processes
○ Stop all processes owned by this task

[Cancel] [Continue]
```

## 15. EMERGENCY STOP

Provide a highly visible but protected emergency action.

`STOP ALL OCC-OWNED TASKS`

It must mean:

> Stop all tasks and processes that OCC explicitly owns.

It must NOT mean:

> Kill every Node/Java/Python process on Windows.

Require confirmation and show the number of affected tasks/processes before execution.

## 16. COMMAND OUTPUT UX

Do not flood the chat with unlimited raw shell output.

Show a compact process card with:

- command;
- state;
- PID;
- duration;
- exit code;
- last N lines;
- expandable full output;
- search within output;
- copy output;
- download/save output if appropriate.

Large output should be virtualized/paginated and persisted separately from normal chat messages.

## 17. CODE / DIFF MESSAGES

When OpenCode changes files, render a compact diff card:

```text
🧑‍💻 OpenCode changed 3 files

+ src/auth/session.ts
~ src/server.ts
+ tests/auth.test.ts

[View Diff]
[Open File]
[Review Changes]
```

The user should not need to leave the conversation to understand what changed.

## 18. ARTIFACTS

Support message attachments/references for:

- diffs;
- logs;
- test reports;
- screenshots;
- generated files;
- build artifacts;
- links;
- structured JSON diagnostics.

Never embed enormous files directly into the message body.

## 19. SEARCH

Provide conversation-wide search.

Search should cover:

- messages;
- task IDs;
- session IDs;
- commands;
- errors;
- file paths;
- decisions;
- process IDs.

## 20. UNREAD / ATTENTION SYSTEM

Implement attention indicators for:

- new ChatGPT response;
- new OpenCode response;
- question waiting for answer;
- human approval required;
- task failure;
- process failure;
- disconnected session;
- recovery required.

A notification must identify the exact conversation/task requiring attention.

## 21. MULTI-TASK / MULTI-SESSION CHAT

The UI must support multiple simultaneous OpenCode tasks.

Example:

```text
Hermes #42     🟢
Nothing #18    🟡 waiting
TurboFlix #9   🟢
SmartClip #3   🔴 failed
```

Events must never leak between conversations.

Every event/message must be scoped by durable identifiers such as:

```text
tenant/user
projectId
conversationId
taskId
sessionId
processId
messageId
eventId
```

## 22. CHATGPT / OPENCODE IDENTITY

The UI must make the distinction obvious.

Use different avatars/badges/labels and never fake one agent's text as another's.

Recommended:

```text
👤 Aditya
🤖 ChatGPT
🧑‍💻 OpenCode
⚙ Control Center
🔧 Process
```

## 23. SYSTEM EVENTS

System events should be visually quieter than human/AI messages.

Examples:

```text
⚙ OpenCode connected
⚙ Task started
⚙ Permission approved by Aditya
⚙ Shell process started
⚙ ChatGPT disconnected
⚙ Reconnected — 4 events replayed
```

These should be collapsible to avoid clutter.

## 24. MESSAGE DELIVERY / FAILURE STATES

Support clear states:

- sending
- sent
- acknowledged
- failed
- retrying
- superseded
- cancelled

If sending an instruction to OpenCode fails, the UI must not display it as successfully delivered.

## 25. OFFLINE / DISCONNECTED UX

If ChatGPT disconnects:

```text
⚠ ChatGPT connection lost

OpenCode can continue according to current policy.
Pending questions will remain queued.

[Reconnect]
```

If OpenCode disconnects:

```text
⚠ OpenCode disconnected

Last known state: Running
Last event: npm test started

[Reconnect] [Mark Recovery Required]
```

Never fabricate continuity.

## 26. SECURITY / PROMPT INJECTION IN CHAT

Messages originating from OpenCode, shell output, files, websites, logs, or external tools are **untrusted content**.

The UI must visually distinguish:

- operator instructions;
- trusted control metadata;
- AI-generated text;
- untrusted tool output.

Do not allow text such as “ignore previous instructions” inside a file/log/tool output to become a control command automatically.

Approval actions must be generated from structured system state, not parsed from natural-language messages.

## 27. MOBILE RESPONSIVENESS

The web UI MUST be responsive.

On mobile:

- conversation list becomes a drawer;
- context panel becomes a bottom sheet/drawer;
- approval cards remain fully usable;
- emergency stop remains accessible;
- message composer stays usable;
- process controls remain readable.

The same local dashboard should be usable from a phone over a secure LAN/tunnel without exposing the service unauthenticated.

## 28. ACCESSIBILITY

Support:

- keyboard navigation;
- visible focus states;
- semantic buttons;
- screen-reader labels;
- sufficient contrast;
- non-color-only status indicators;
- reduced-motion preference.

## 29. VISUAL DESIGN

Use a modern developer-tool / messaging aesthetic.

Avoid:

- generic enterprise dashboard templates;
- giant KPI cards dominating the screen;
- excessive charts;
- fake analytics;
- cluttered tables as the primary UI.

The primary visual hierarchy should be:

```text
Conversation > Current action/decision > Context > Historical/administrative details
```

Dark mode should be excellent, with light mode if practical.

## 30. ARCHITECTURAL REQUIREMENT FOR THE UI

The frontend must consume a normalized event/message model from the Control Hub rather than coupling directly to OpenCode internals.

Suggested frontend domains:

```text
ui/
├── conversations/
├── messages/
├── composer/
├── approvals/
├── questions/
├── processes/
├── tasks/
├── sessions/
├── diffs/
├── artifacts/
├── notifications/
└── settings/
```

Suggested backend event types:

```text
message.created
message.updated
message.failed
question.created
question.updated
approval.created
approval.resolved
task.created
task.status_changed
session.status_changed
process.created
process.output
process.status_changed
process.termination_requested
process.terminated
file.changed
tool.started
tool.finished
system.notice
```

All events require stable IDs and timestamps.

## 31. MESSAGE ORDERING / RACE CONDITIONS

The implementation must handle:

- OpenCode message arrives while ChatGPT is responding;
- human sends a message while an agent is waiting;
- task is stopped while a tool result arrives;
- process exits while a stop request is being issued;
- browser reconnects while events are being emitted;
- duplicate events;
- out-of-order network delivery.

Use sequence numbers/versioning/idempotency where appropriate.

The UI must reconcile state rather than trusting event order blindly.

## 32. HISTORY COMPACTION

If a conversation becomes very large:

- preserve the canonical raw history;
- allow UI virtualization/pagination;
- optionally provide AI-generated summaries;
- never replace authoritative raw history with a summary;
- mark summaries clearly;
- allow jumping from summary to source messages.

## 33. NO FAKE CHAT

The UI must never simulate ChatGPT or OpenCode messages merely to make the demo look alive.

Every displayed agent message must have a real source/event/message ID.

If a demo mode exists, it must be explicitly labeled `DEMO MODE`.

## 34. ACCEPTANCE TESTS

The implementation is incomplete unless these scenarios work:

1. User creates a task and sees a messaging conversation immediately.
2. OpenCode sends progress and the conversation updates live.
3. ChatGPT sends a message that appears as ChatGPT, not OpenCode.
4. OpenCode asks a question and it appears inline as an interactive decision card.
5. ChatGPT proposes an answer and the recommendation appears inline.
6. User approves the recommendation and OpenCode receives the approved answer.
7. User overrides ChatGPT with a custom answer.
8. User pauses a task while `npm run dev` is running and can choose whether the process survives.
9. User stops the agent but leaves an approved dev server alive.
10. User terminates only task-owned processes without killing another task's process.
11. Browser disconnects and reconnects; missed messages are replayed exactly once.
12. OpenCode disconnects; UI shows last known state and recovery status.
13. ChatGPT disconnects; pending questions remain durable.
14. Two simultaneous tasks do not cross-contaminate messages or processes.
15. A stale approval cannot approve a newer/different action.
16. A malicious-looking string in shell output cannot trigger an approval or control operation.
17. Large shell output does not freeze the browser.
18. Mobile UI can answer an approval request and stop a task safely.
19. Emergency stop affects only OCC-owned tasks/processes.
20. Restarting the Control Hub preserves task/message/question/process state and reconciles live processes.

## 35. FINAL PRODUCT DEFINITION

The finished application should feel like:

> **A private developer group chat where Aditya, ChatGPT, and OpenCode collaborate on real coding tasks — with Aditya holding the controls, approvals, permissions, process management, and final authority.**

It is NOT merely:

- an admin dashboard;
- an OpenCode wrapper;
- a ChatGPT prompt relay;
- a process viewer;
- a generic chat app.

It is a **messaging-first human-in-the-loop AI coding control center**.
