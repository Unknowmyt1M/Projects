# OpenCode Control Center — FINAL MASTER IMPLEMENTATION PROMPT

**Status: canonical implementation specification**

Build **OpenCode Control Center (OCC)** as a production-quality, Windows-first, local-first human-in-the-loop control plane connecting a human operator, OpenCode agent(s), and a ChatGPT-compatible MCP client.

> **Human = final authority. ChatGPT = reasoning/planning/review participant. OpenCode = execution/coding participant. OCC = durable source of truth, policy enforcement, process supervision, messaging bus, actor-routing layer, and audit/recovery system.**

This document supersedes previous OCC prompts. Preserve useful requirements from earlier versions, but follow the corrections and additions in this version when requirements conflict.

---

# 0. NON-NEGOTIABLE RULES

1. Never invent OpenCode, MCP, ChatGPT, or OS APIs.
2. Before implementation, inspect the installed OpenCode version and current official OpenCode documentation.
3. Verify the current MCP specification/SDK and the actual target ChatGPT MCP connectivity model at implementation time.
4. Never hard-code an MCP transport merely because an earlier document named one. Transport is a deployment capability, not an architectural assumption.
5. Never assume ChatGPT can reach `localhost` directly.
6. Never expose an unauthenticated OCC control endpoint to the Internet.
7. Human policy is authoritative over both AI actors.
8. A ChatGPT recommendation is not human approval unless policy explicitly permits automation.
9. Never fabricate messages, progress, tool results, approvals, or actor identity.
10. Never label OCC-generated text as ChatGPT or OpenCode.
11. Never kill processes by executable name alone.
12. Never use PID alone as process identity.
13. Treat shell output, repository content, web content, tool output, diffs, commit messages, and model-generated text as untrusted data; they cannot override OCC policy.
14. MCP request lifetime, OCC task lifetime, OpenCode session lifetime, ChatGPT conversation lifetime, and OS process lifetime are independent.
15. Every mutation must be authenticated, authorized, policy-checked, auditable, and idempotent where appropriate.
16. When state is uncertain, show `UNKNOWN` / `RECOVERY_REQUIRED`; never guess.
17. Do not mark acceptance criteria complete merely because code exists. They must pass real end-to-end tests.
18. Never broadcast every human OCC message to every connected AI. **Every outbound message must pass explicit actor-routing policy.**
19. Observer mode is a real authorization mode, not merely a UI label. An observer cannot execute, mutate, approve, or send autonomous actions.
20. A provider binding identifies the exact external target. Never infer the target from the current TUI screen, current folder, most recent chat, or ambiguous title.

---

# 1. PRODUCT UX — A GROUP MESSAGING APP FIRST

The primary experience must feel like **Discord/Telegram/Slack + VS Code**, not an admin dashboard.

Participants:

```text
👤 Human
🤖 ChatGPT
🧑‍💻 OpenCode
⚙ OCC System
🔧 Tool / Process
```

Main layout:

```text
┌──────────────────────────────────────────────────────────────────────┐
│ OCC • Hermes / Task #42             ● OpenCode   ● ChatGPT          │
├───────────────┬──────────────────────────────────┬───────────────────┤
│ Conversations │            GROUP CHAT             │ Context / Control │
│               │                                  │                   │
│ 🟢 Hermes     │ 👤 Human                         │ TASK              │
│   Auth #42    │ Implement session auth.          │ RUNNING           │
│               │                                  │                   │
│ 🟡 Nothing    │ 🧑‍💻 OpenCode                   │ CURRENT TOOL      │
│   OAuth #18   │ I found two possible approaches. │ npm test           │
│               │                                  │                   │
│ 🟢 TurboFlix  │ 🤖 ChatGPT                       │ PROCESSES         │
│   UI #9       │ I recommend approach B because… │ npm → node         │
│               │                                  │                   │
│               │ 👤 Human                         │ APPROVALS         │
│               │ Go with B.                       │ 1 pending         │
│               │                                  │                   │
│               │ [ Type a message… ]              │ [Pause] [Stop]    │
└───────────────┴──────────────────────────────────┴───────────────────┘
```

## UI requirements

- Virtualized message list for large histories.
- Sticky date separators.
- `New messages` indicator when scrolled away from bottom.
- Markdown + code syntax highlighting.
- Copy buttons.
- Reply/thread support.
- Message search.
- Conversation/task search.
- Live streaming with authoritative reconciliation.
- Unread/attention indicators.
- Mobile-responsive layout.
- Command palette (`Ctrl+K`).
- Keyboard shortcuts must never bypass authorization.
- File/diff cards.
- Tool execution cards.
- Question/recommendation cards.
- Approval cards.
- Process cards.
- Reconnect/recovery banners.
- Real-time task state.
- Visible outbound-target indicator in the composer, e.g. `🧑‍💻 OpenCode • Execute`, `🤖 ChatGPT • Ask`, `👁 Observer`.
- Users must be able to inspect and change routing before sending when policy permits.
- The UI must never imply that a provider received a message when OCC has not confirmed delivery.

### Important correction
A UI button such as **“Revert this chunk”** must NOT silently mutate files. It must create a normal policy-controlled change request showing the exact diff, target revision, and approval requirement.

---

# 2. DOMAIN MODEL

Core entities:

```text
projects
worktrees
conversations
conversation_members
messages
message_revisions
message_routes
provider_bindings
tasks
opencode_projects
opencode_sessions
chatgpt_conversations
agents
questions
question_versions
recommendations
approvals
policies
policy_versions
processes
process_snapshots
events
context_snapshots
cost_records
usage_records
idempotency_keys
reconciliation_runs
audit_log
recovery_snapshots
```

Every externally mutable entity needs ownership, authorization scope, versioning where relevant, and audit metadata.

---

# 3. MESSAGE PROVENANCE — HARD SECURITY INVARIANT

```ts
Message {
  id: string
  conversationId: string
  taskId?: string
  senderType: "human" | "chatgpt" | "opencode" | "system" | "tool" | "process"
  senderId?: string
  targetType?: "human" | "chatgpt" | "opencode" | "both" | "system"
  source: "ui" | "mcp" | "opencode_adapter" | "chatgpt_adapter" | "system" | "tool_event"
  type: "text" | "question" | "recommendation" | "decision" | "tool" | "shell_output" | "diff" | "system_event"
  content: unknown
  replyToMessageId?: string
  correlationId?: string
  causationId?: string
  sequence: number
  createdAt: string
  status: "pending" | "sending" | "sent" | "delivered" | "failed"
}
```

Hard invariants:

- OCC cannot manufacture a ChatGPT/OpenCode message and mark it as genuine.
- Optimistic UI messages are explicitly temporary and reconciled with the server.
- Retries cannot duplicate logical messages.
- Every AI response has verifiable origin metadata.
- Human edits to AI recommendations are stored as a separate human decision, never as an overwritten AI message.
- A message may have multiple route records, but each route has its own target, role, delivery state, and authorization decision.
- Observer-delivered context must be marked as observation/context, never as an instruction originating from the human to execute an action.

---

# 4. ACTOR ROUTING + OBSERVER MODE — MANDATORY SUBSYSTEM

OCC must **not** blindly send every human message to both OpenCode and ChatGPT. This is a core product behavior.

## 4.1 Actor roles

Each provider binding may operate in one of these roles for a message/task:

```text
EXECUTOR  = may perform authorized actions
ADVISOR   = may answer/recommend but may not directly execute actions
OBSERVER  = read-only participant; may receive execution context but cannot act
REVIEWER  = may inspect completed/in-progress work and produce a review
CONTROLLER = human-only policy authority
```

Human is the final controller. OpenCode will normally be the executor for coding/file/process tasks. ChatGPT will normally be an observer/advisor/reviewer unless the user explicitly routes a request to ChatGPT.

## 4.2 Message routing modes

Every outbound human message must resolve to one explicit routing mode before dispatch:

```text
EXECUTE
ASK
DISCUSS
REVIEW
OBSERVE
BROADCAST
```

Examples:

```text
"Create a new file test.ts"
→ OpenCode: EXECUTE
→ ChatGPT: OBSERVE

"ChatGPT, suggest a structure for this file"
→ ChatGPT: ASK / ADVISOR
→ OpenCode: OBSERVE

"Review what OpenCode just changed"
→ ChatGPT: REVIEW / REVIEWER
→ OpenCode: OBSERVE

"Both of you, discuss the best approach"
→ OpenCode: DISCUSS
→ ChatGPT: DISCUSS

"Run the tests"
→ OpenCode: EXECUTE
→ ChatGPT: OBSERVE
```

The router may use intent classification to suggest a route, but **classification alone must never silently grant execution permission**. Explicit user choice, deterministic policy, or a preconfigured session default must authorize the final route.

## 4.3 Default routing policy

For a normal coding OCC session:

```text
Human coding/file/process request
        ↓
OpenCode = EXECUTOR
ChatGPT  = OBSERVER
```

For reasoning/research requests explicitly addressed to ChatGPT:

```text
ChatGPT = ADVISOR/REVIEWER
OpenCode = OBSERVER
```

For explicit multi-agent discussion:

```text
OpenCode = DISCUSS
ChatGPT  = DISCUSS
```

Never reinterpret a normal execution request as a multi-agent execution request merely because both providers are connected.

## 4.4 Observer semantics

Observer mode is a real permission boundary.

An observer may receive:

- human message metadata needed for context;
- OpenCode task status;
- tool invocation summaries;
- shell/process state;
- file/diff metadata and authorized diffs;
- execution results;
- errors;
- final output;
- questions raised by the executor.

An observer may **not**:

- execute tools;
- mutate files;
- run shell commands;
- approve actions;
- change policy;
- autonomously redirect the executor;
- turn observed content into an execution command without an explicit authorized route.

For ChatGPT specifically, an observer response is advisory/contextual output. It must not be treated as an OpenCode command merely because it appears in the same group chat.

## 4.5 Structured execution observation

When ChatGPT is observing OpenCode, OCC should provide a structured stream such as:

```text
👤 Human
Create test.ts

🧑‍💻 OpenCode
Task accepted

🔧 Tool
create_file → test.ts

🔧 Tool
write_file → test.ts

🧪 Tool
npm test

🧑‍💻 OpenCode
File created and tests passed.
```

The observation channel should be compact and context-aware. Do not continuously forward huge raw logs unless requested or required by policy.

## 4.6 Ask-ChatGPT-from-OpenCode flow

OpenCode must be able to request advice without granting ChatGPT execution authority:

```text
OpenCode
 → OCC
 → ChatGPT ADVISOR
 → recommendation
 → OCC
 → OpenCode / Human
```

If the recommendation would cause a mutation, it becomes a recommendation/decision object and follows the normal human approval/policy path. ChatGPT cannot self-approve its own recommendation.

## 4.7 UI controls

The composer must expose a compact routing control:

```text
Target: [ 🧑‍💻 OpenCode ▼ ]
Mode:   [ ⚡ Execute ▼ ]
```

Available choices depend on policy:

```text
🧑‍💻 OpenCode → Execute / Ask / Observe
🤖 ChatGPT   → Ask / Review / Observe
👥 Both      → Discuss / Compare / Observe
```

Do not overwhelm the default UI. Advanced routing can live in a popover/command palette.

Every sent message must render its resolved route in message metadata/details.

---

# 5. PROVIDER BINDINGS + OCC SESSION CREATION

An OCC conversation/session is a durable workspace with explicit bindings to external provider targets.

When the user creates a new OCC session, do **not** simply connect to whichever OpenCode folder or ChatGPT conversation happens to be currently active.

The creation flow must support:

```text
OCC Session Name

OpenCode:
  ○ Create New Session
  ○ Select Existing Session
  ○ Unlinked

ChatGPT:
  ○ Create New Chat
  ○ Select Existing Chat
  ○ Unlinked
```

Example internal binding:

```text
OCC Session
├── id: occ_xxx
├── name: "Hermes OAuth Fix"
│
├── OpenCode binding
│   ├── mode: existing
│   ├── projectId: project_xxx
│   └── sessionId: ses_xxx
│
└── ChatGPT binding
    ├── mode: existing
    └── conversationId: conv_xxx
```

The binding is the deterministic destination for future routed messages until the user explicitly changes it.

### Binding changes

Changing a provider binding affects **future messages only**. Existing messages remain associated with their original route. Never silently migrate historical messages between provider conversations.

### Provider target visibility

Show human-friendly names by default; IDs remain available in advanced details.

Example:

```text
OpenCode: 🟢 Hermes OAuth
ChatGPT:  🟢 OAuth Research
```

Advanced details:

```text
OpenCode projectId: ...
OpenCode sessionId: ...
ChatGPT conversationId: ...
```

---

# 6. OPENCODE GLOBAL PROJECT + SESSION DISCOVERY

**OCC must not use the OpenCode TUI's folder/project presentation as its global source of truth.** The TUI may show only a scoped or filtered subset. OCC must independently discover all projects and all accessible sessions using the current supported OpenCode server/API capabilities.

Before implementation, research and verify the installed OpenCode version and current official server API. At minimum, investigate the current equivalents of:

```text
GET /project
GET /session
GET /session/:id
GET /global/event
```

Do not assume these exact paths remain unchanged; verify them at implementation time.

## 6.1 Global mode vs project mode

OCC must have explicit discovery scopes:

```text
GLOBAL MODE
→ all accessible/discoverable OpenCode projects and sessions

PROJECT MODE
→ selected project and its sessions
```

The global OpenCode browser must never accidentally inherit the current working directory as a filter.

## 6.2 Lightweight session index

OCC should maintain a searchable metadata index containing fields such as:

```text
projectId
projectName
projectPath
sessionId
sessionTitle
createdAt
updatedAt
status
parentSessionId
model
lastMessagePreview
```

Do not copy every transcript into the index. Fetch authoritative conversation details when a session is opened.

## 6.3 Lazy loading

Do not load hundreds/thousands of full sessions and transcripts into the browser at startup.

Use:

```text
projects → metadata
sessions → metadata/pagination
session opened → actual conversation
search → server/index-backed search where possible
```

Support pagination/infinite scrolling and virtualized lists.

## 6.4 Live synchronization

If the current OpenCode server exposes a global event stream, subscribe to it and reconcile events into the OCC index. Do not rely on aggressive polling when a supported event stream exists.

Events may include session creation/update, message creation, status changes, and other relevant state changes. Verify the actual event schema before implementation.

## 6.5 Startup/reconnect reconciliation

On OCC startup and after OpenCode reconnect:

1. Connect to the OpenCode server.
2. Discover all projects available to OCC.
3. Discover all accessible sessions globally, not only the current folder.
4. Compare with the OCC index.
5. Add missing sessions.
6. Update changed metadata.
7. Detect deleted/unavailable sessions.
8. Preserve uncertain state as `UNKNOWN` rather than guessing.
9. Re-subscribe to live events.
10. Record a reconciliation run for audit/debugging.

Periodic lightweight reconciliation should recover from missed events or OCC downtime.

## 6.6 Never silently hide a session

If a session exists but its project cannot currently be resolved, show it under:

```text
⚠️ Unknown / Unresolved Project
```

rather than silently dropping it.

Session identity must not depend on title uniqueness. Use stable provider identifiers such as:

```text
(projectId, sessionId)
```

where supported.

Filesystem inspection of OpenCode's data directories may be implemented as a **diagnostic/recovery fallback**, but it must not be the primary discovery mechanism when a supported server/API exists.

---

# 7. NATIVE CHATGPT WEB CONNECTOR — PERFORMANCE-FIRST

The target for the native ChatGPT integration is the user's actual ChatGPT web conversations/sidebar chats, not an unrelated API-only conversation pretending to be a native chat.

Because undocumented/internal web interfaces may change, OpenCode must research the current implementation before coding and clearly classify capabilities as:

```text
VERIFIED
EXPERIMENTAL
UNSUPPORTED
```

## 7.1 Preferred transport

Use a **local HTTP/SSE adapter as the primary runtime path** when current technical research verifies that the required native web operations can be performed reliably.

Do not keep Chromium/Playwright running for every message.

Browser automation is only a fallback/bootstrap/recovery mechanism when required by current authentication or anti-abuse requirements.

Preferred architecture:

```text
OCC
 │
 └── Native ChatGPT Adapter
       ├── HTTP client
       ├── SSE stream parser
       ├── session/auth state manager
       ├── capability detector
       ├── compatibility layer
       ├── retry/reconnect manager
       └── optional browser-assisted bootstrap/recovery
```

## 7.2 Native conversation operations to research/verify

At minimum investigate current support for:

```text
list/search conversations
read conversation
create new conversation
send message as the user
receive streaming response
rename conversation
open/continue existing conversation
```

Never assume endpoint paths or payloads are stable. Build a compatibility layer and capability probe.

## 7.3 Large chat history

If the account contains hundreds/thousands of ChatGPT conversations, do **not** load all full conversations into OCC.

Use:

```text
recent metadata
server-side/search where supported
pagination
lazy loading
full conversation fetch only when selected
```

The selector should show human-friendly titles, timestamps, and useful metadata; conversation IDs remain an advanced detail.

## 7.4 Security

Do not store ChatGPT session cookies/tokens in GitHub, source control, ordinary application logs, or an unencrypted remote database. Prefer OS-protected local credential/session storage and least-privilege handling.

Never route authentication through an unknown third-party proxy.

## 7.5 Browser fallback

If browser assistance is necessary:

```text
User connects ChatGPT
 → browser-assisted bootstrap/login
 → establish local authenticated state
 → close browser when possible
 → HTTP/SSE adapter handles normal traffic
```

Do not make Playwright/Chromium the normal per-message transport unless research proves there is no viable lower-resource alternative.

## 7.6 Native ChatGPT vs OpenAI API

Keep these integrations separate:

```text
Native ChatGPT connector
→ actual chatgpt.com conversation target

OpenAI API connector
→ API Conversations/Responses target
```

An OpenAI API conversation must never be represented as though it were a native ChatGPT sidebar conversation.

---

# 8. COMPLETE MESSAGE / AI FLOW

## 8.1 Human → OpenCode execution request

```text
Human UI
 → OCC authentication
 → actor routing resolution
 → policy check
 → persist human message
 → create route: OpenCode / EXECUTE
 → ChatGPT receives observation context only if bound/policy-enabled
 → OpenCode adapter
 → exact bound OpenCode session
 → OpenCode execution
 → normalized tool/process/events
 → OCC persistence
 → optional ChatGPT observer stream
 → realtime UI
```

## 8.2 Human → ChatGPT advisory request

```text
Human UI
 → OCC authentication
 → actor routing resolution
 → policy check
 → persist human message
 → create route: ChatGPT / ASK or REVIEW
 → exact bound ChatGPT conversation
 → genuine response
 → provenance validation
 → persist response
 → optional OpenCode observer context
 → UI
```

## 8.3 Multi-agent discussion

```text
Human
 → OCC
 → explicit DISCUSS route
 → OpenCode + ChatGPT
 → responses
 → OCC correlation/reconciliation
 → UI
```

Do not infer this mode from the mere fact that both providers are connected.

## 8.4 OpenCode asks ChatGPT

```text
OpenCode
 → OCC question object
 → policy check
 → ChatGPT ADVISOR
 → recommendation
 → OCC
 → Human and/or OpenCode
```

If the recommendation proposes an action, it remains a recommendation until the normal authorization/approval path allows execution.

## 8.5 MCP-facing interaction

Do not assume MCP automatically provides arbitrary server-to-ChatGPT push messaging. The implementation must use whatever interaction/delivery mechanism the actual supported ChatGPT MCP client exposes.

---

# 9. QUESTIONS, RECOMMENDATIONS, AND HUMAN CONTROL

OpenCode must be able to ask the human a question through OCC:

```text
OpenCode → OCC → Human
```

It must also be able to ask ChatGPT for advice:

```text
OpenCode → OCC → ChatGPT
```

ChatGPT may provide:

```text
recommendation
analysis
review
alternative approaches
```

But ChatGPT must not silently convert a recommendation into an execution command.

Every recommendation that could cause a mutation must have:

```text
recommendationId
sourceActor
targetActor
riskLevel
proposedAction
contextSnapshot
policyDecision
approval state
```

Human approval remains authoritative unless the user explicitly configured a lower-risk automated policy.

---

# 10. PROCESS / SHELL SUPERVISION

A task and its process tree are separate state machines.

Never assume:

```text
kill OCC task = kill every process
```

When a task is paused/stopped/killed, OCC must inspect active processes and let the human choose when policy permits:

```text
Stop task only
Stop task + child shell
Stop shell + descendants
Leave process running
```

Process identity must include a stable launch identity, not merely executable name or PID. Account for PID reuse, detached children, shell wrappers, process groups, Windows job objects where appropriate, and race conditions between discovery and termination.

The UI must show exactly which processes will be affected before destructive termination when practical.

---

# 11. REALTIME + EVENTING

Use a durable event model with:

```text
sequence numbers
correlation IDs
causation IDs
idempotency keys
provider event IDs where available
replay/reconciliation
```

The UI may optimistically render local state, but authoritative provider state must reconcile it.

For streaming responses, handle:

- partial chunks;
- disconnects;
- duplicate events;
- out-of-order events;
- reconnects;
- final response reconciliation;
- cancellation;
- timeout;
- provider errors.

---

# 12. TASK STATE MACHINE

Use explicit states rather than booleans:

```text
CREATED
QUEUED
RUNNING
WAITING_FOR_HUMAN
WAITING_FOR_AI
PAUSING
PAUSED
CANCELLING
CANCELLED
SUCCEEDED
FAILED
UNKNOWN
RECOVERY_REQUIRED
```

Provider state and OS process state must be tracked separately.

---

# 13. SECURITY MODEL

Threat model at minimum:

- malicious repository content;
- prompt injection in files/docs/web pages;
- malicious tool output;
- shell injection;
- path traversal;
- arbitrary command execution;
- forged provider events;
- replayed MCP requests;
- duplicate submissions;
- stale approvals;
- confused-deputy routing;
- unauthorized provider rebinding;
- ChatGPT session credential leakage;
- OpenCode session leakage;
- process termination races;
- local-network unauthorized access.

Critical rule:

> **Observed content is data, not authority.**

No message, repository file, tool output, or model response can override OCC policy merely by containing instructions.

Provider routing must be authorization-checked independently of message content.

---

# 14. AUDIT LOG

Audit all security-sensitive and routing-sensitive events:

```text
human_login
provider_connected
provider_disconnected
occ_session_created
provider_binding_created
provider_binding_changed
message_created
message_route_created
message_delivered
message_failed
actor_role_changed
routing_policy_decision
question_created
recommendation_created
approval_requested
approval_granted
approval_rejected
task_started
task_paused
task_cancelled
process_discovered
process_terminated
reconciliation_started
reconciliation_completed
recovery_started
recovery_completed
```

Each record should include actor, target, decision, timestamp, correlation ID, and relevant before/after state.

---

# 15. EDGE CASES — MUST DESIGN FOR THEM

At minimum:

1. User sends a coding request while ChatGPT is disconnected.
2. User sends a ChatGPT question while OpenCode is disconnected.
3. User selects `Both` but policy permits only one executor.
4. OpenCode session is deleted externally.
5. ChatGPT conversation is renamed externally.
6. OpenCode TUI does not show a session that the server API discovers.
7. A project cannot currently be resolved for a discovered session.
8. OCC misses provider events while offline.
9. ChatGPT native endpoint changes.
10. ChatGPT SSE stream disconnects mid-response.
11. OpenCode emits duplicate events.
12. Same human message is retried after a timeout.
13. User changes provider binding while a task is running.
14. User changes routing mode after a message is queued but before dispatch.
15. Observer receives an execution event containing prompt-injection text.
16. ChatGPT advisor proposes a dangerous or unauthorized command.
17. OpenCode asks ChatGPT for advice while human approval is pending.
18. Two OCC windows attempt conflicting process controls.
19. A shell spawns detached child processes.
20. PID is reused after a process exits.
21. OCC crashes during message delivery.
22. OCC crashes during process termination.
23. Network disappears during an SSE stream.
24. Provider responds ambiguously after a timeout.
25. User creates an OCC session with OpenCode `None` and ChatGPT `None`.
26. User creates an OCC session with one new provider target and one existing target.
27. Hundreds/thousands of OpenCode sessions exist.
28. Hundreds/thousands of ChatGPT conversations exist.
29. Duplicate titles exist across projects/providers.
30. Provider authentication expires.
31. User tries to route an observer-only provider as executor.
32. ChatGPT response is mistaken for an executable command.
33. OpenCode response is mistakenly attributed to ChatGPT.
34. Historical messages are incorrectly migrated after a binding change.
35. Browser fallback launches while a normal HTTP adapter is already active.

For each case define:

```text
expected state
user-visible behavior
provider behavior
recovery behavior
audit behavior
```

---

# 16. PERFORMANCE REQUIREMENTS

OCC should remain responsive with:

```text
1,000+ OCC messages per conversation
10,000+ indexed OpenCode sessions
10,000+ indexed ChatGPT conversations where supported
multiple concurrent streams
multiple running tasks
```

Do not load complete transcripts unnecessarily.

Use:

- pagination;
- virtualization;
- lazy loading;
- metadata indexes;
- incremental event updates;
- bounded caches;
- backpressure;
- stream cancellation;
- reconnection with jitter;
- deduplication.

Native ChatGPT HTTP/SSE must be preferred over persistent browser automation for normal traffic when technically viable.

---

# 17. RESEARCH-FIRST IMPLEMENTATION REQUIREMENT

Before implementing provider-specific behavior, OpenCode must perform its own current research rather than trusting this prompt as a list of guaranteed API paths.

Research must cover:

### OpenCode

- installed version;
- official server API;
- project discovery;
- global session discovery;
- session details;
- event stream;
- session/message/tool event schemas;
- process/task behavior;
- Windows-specific behavior;
- current CLI capabilities.

### MCP / ChatGPT

- current MCP specification;
- current SDKs;
- actual ChatGPT MCP connectivity model;
- current supported transport;
- current authentication requirements;
- current native ChatGPT web behavior;
- current conversation/message operations;
- current streaming behavior;
- current anti-abuse/auth requirements;
- known compatibility changes.

### Process supervision

Research Windows process groups/job objects, shell wrappers, detached processes, termination semantics, PID reuse, and race-safe process identity.

The implementation must produce a research/compatibility report containing:

```text
Capability
Source/version tested
Implementation status: VERIFIED / EXPERIMENTAL / UNSUPPORTED
Test evidence
Known limitations
Fallback
```

If a capability is unsupported, do not fake it. Build the best safe fallback and clearly surface the limitation.

---

# 18. TESTING + ACCEPTANCE

Acceptance must be end-to-end and evidence-based.

## OpenCode discovery

- Start multiple projects.
- Create sessions in multiple projects.
- Verify OCC discovers all projects.
- Verify OCC discovers sessions not visible in a scoped TUI view.
- Verify global/project filtering.
- Verify pagination/search.
- Verify reconnect reconciliation.
- Verify event-driven updates.
- Verify unresolved projects are visible rather than silently omitted.

## Provider binding

- Create OCC session with OpenCode new + ChatGPT new.
- Create with existing + existing.
- Create with one provider unlinked.
- Change binding while idle.
- Change binding while task is active and verify queued/new messages follow explicit policy.
- Verify old messages remain attributed to original targets.

## Actor routing

- `Create a file` → OpenCode executes; ChatGPT observes only.
- `Ask ChatGPT for architecture` → ChatGPT answers; OpenCode observes only.
- `Review OpenCode's change` → ChatGPT reviews.
- `Both discuss` → both receive explicit discussion route.
- Attempt to make observer execute → must be rejected by policy.
- ChatGPT recommendation must not automatically execute.

## Native ChatGPT

Where the current verified connector supports it:

- list/search native conversations;
- select exact conversation;
- create new conversation;
- send a user message;
- receive streamed response;
- continue conversation;
- rename;
- reconnect after stream failure;
- verify provider-side result.

If any operation is unsupported, acceptance must record `UNSUPPORTED` instead of fabricating success.

## Process controls

- pause while shell command runs;
- stop task without killing unrelated process;
- stop selected process tree;
- detached child handling;
- PID reuse simulation;
- crash/recovery during termination.

---

# 19. IMPLEMENTATION DELIVERABLES

Before declaring the project complete, provide:

```text
architecture document
provider capability report
threat model
routing/permission matrix
data model/schema
API contracts
OpenCode adapter
Native ChatGPT adapter
MCP integration
actor-routing engine
observer context pipeline
process supervisor
reconciliation engine
session index
realtime event layer
frontend messaging UI
test suite
end-to-end verification report
recovery/runbook
security documentation
```

The implementation should be modular enough that an undocumented provider change does not require rewriting OCC's core domain model or UI.

---

# 20. FINAL ARCHITECTURAL PRINCIPLE

OCC is **not** a dumb message relay.

It is a policy-controlled coordination layer:

```text
                         👤 HUMAN
                       CONTROLLER
                           │
                           ▼
                    ┌──────────────┐
                    │     OCC      │
                    │              │
                    │ Policy       │
                    │ Routing      │
                    │ Audit        │
                    │ Recovery     │
                    │ Reconcile    │
                    └──────┬───────┘
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
       🧑‍💻 OPENCODE                🤖 CHATGPT
       EXECUTOR                    ADVISOR / OBSERVER
              │                         │
              │                         │
       exact bound                exact bound
       project+session             conversation
              │                         │
              └────────────┬────────────┘
                           ▼
                    shared OCC context
```

The critical rule is:

> **Every message has an explicit destination, role, authorization decision, and delivery state. Connection does not imply execution. Observation does not imply authority. A recommendation does not imply approval. A provider binding does not imply that every message must be sent to that provider.**

Build the system around these invariants, verify every provider capability against the current real environment, and optimize for correctness, recoverability, security, and a fast messaging-first user experience.