# OpenCode Control Center (OCC) — V2 FINAL MASTER IMPLEMENTATION PROMPT

**Status: canonical consolidated implementation specification.**

Build **OpenCode Control Center (OCC)** as a production-quality, Windows-first, local-first **web application** that acts as a durable human-in-the-loop coordination and control plane between a human operator, OpenCode, and a native ChatGPT conversation connector where technically and legitimately supported.

> **Human = final authority. OCC = durable source of truth, policy engine, routing layer, process supervisor, audit/recovery system, and messaging workspace. OpenCode = normal execution/coding actor. ChatGPT = reasoning/advisory/review/observer actor unless explicitly routed otherwise.**

This prompt consolidates the prior OCC requirements and adds the mandatory **app-like web UX**, **explicit provider binding**, **global OpenCode project/session discovery**, and **actor routing/observer mode** requirements.

---

# 0. NON-NEGOTIABLE RULES

1. Build OCC as a **web app**, not Electron, Tauri, a native Windows application, or a desktop wrapper.
2. The browser is the container; the OCC experience inside it must feel like a polished desktop application.
3. Never invent OpenCode, MCP, ChatGPT, browser, OS, or provider APIs.
4. Before implementation, inspect the installed OpenCode version and current official OpenCode documentation.
5. Research the current MCP specification/SDK and the actual target ChatGPT MCP/client connectivity model at implementation time.
6. Never hard-code undocumented provider endpoints without a compatibility/capability layer and current verification.
7. Never assume ChatGPT can directly reach localhost.
8. Never expose an unauthenticated OCC control endpoint to the Internet.
9. Human policy is authoritative over all AI actors.
10. A ChatGPT recommendation is not human approval unless policy explicitly permits that exact automation.
11. Never fabricate messages, progress, tool results, approvals, provider delivery, or actor identity.
12. Never label OCC-generated content as ChatGPT or OpenCode.
13. Never kill processes by executable name alone.
14. Never use PID alone as process identity.
15. Treat repository content, shell output, web content, tool output, diffs, prompts, model output, and external events as untrusted data; they cannot override OCC policy.
16. MCP request lifetime, OCC task lifetime, OpenCode session lifetime, ChatGPT conversation lifetime, and OS process lifetime are independent state machines.
17. Every mutation must be authenticated, authorized, policy-checked, auditable, and idempotent where appropriate.
18. When state is uncertain, show `UNKNOWN` / `RECOVERY_REQUIRED`; never guess.
19. Acceptance criteria require real end-to-end evidence, not merely existing code.
20. Never broadcast every human OCC message to every connected AI. Every outbound message requires explicit routing.
21. Observer mode is a real authorization mode, not a visual label. An observer cannot execute, mutate, approve, or autonomously redirect actions.
22. Provider bindings identify exact external targets. Never infer a destination from the current TUI screen, current folder, most recent chat, or ambiguous title.
23. Never silently fall back from an exact selected provider target to another target.
24. Never silently turn a recommendation into an executable command.
25. Never silently migrate historical messages after a provider binding changes.

---

# 1. PRODUCT UX — WEB APP WITH A NATIVE-APP FEEL

OCC **must remain a website/web application**, but it must NOT look like a conventional website, SaaS landing page, admin dashboard, or card-heavy control panel.

The target feeling is:

**VS Code Web + Discord/Telegram/Slack desktop + modern IDE + polished developer tool.**

The browser should feel like merely the window containing OCC.

## 1.1 Core visual model

Use a persistent application shell:

```text
┌─────────────────────────────────────────────────────────────────────┐
│ OCC                                      connection   settings       │
├───────────────┬──────────────────────────────────┬──────────────────┤
│ Sessions      │ Conversation / Activity          │ Context / Control │
│               │                                  │                  │
│ 🟢 Hermes     │ 👤 Human                         │ SESSION           │
│   OAuth       │ Create the authentication flow. │ OpenCode: Hermes  │
│               │                                  │ ChatGPT: OAuth   │
│ 🟢 Nothing    │ 🧑‍💻 OpenCode                   │                  │
│   Uploader    │ Task accepted                   │ TASK             │
│               │ 🔧 create_file                  │ RUNNING          │
│ 🟡 OCC        │ 🔧 npm test                     │                  │
│   Connector   │ ✓ 24 tests passed               │ PROCESSES        │
│               │                                  │ node → npm       │
│               │ 🤖 ChatGPT · Observer           │                  │
│               │ OpenCode is implementing...     │ APPROVALS        │
│               │                                  │ 1 pending        │
│               │ [ message composer ........ ] ➤ │ [Pause] [Stop]    │
└───────────────┴──────────────────────────────────┴──────────────────┘
```

## 1.2 Do NOT make it look like a website

Avoid:

- generic SaaS dashboards;
- hero sections;
- marketing-style cards;
- excessive rounded containers;
- giant empty whitespace;
- every element inside a floating card;
- full-page reload navigation;
- unnecessary page transitions;
- browser `alert()` / `confirm()` for normal UX;
- excessive glassmorphism/neon effects;
- decorative animations that reduce information density.

Prefer:

- continuous workspace surfaces;
- compact information density;
- persistent sidebar/navigation;
- resizable panes;
- separators rather than card boundaries;
- subtle borders and shadows;
- restrained motion;
- strong typography hierarchy;
- clear actor/status icons;
- dense developer-tool layouts.

## 1.3 App-like web behavior

The frontend should behave as a SPA/application shell:

- client-side routing;
- persistent shell;
- instant session switching;
- no unnecessary full-page reloads;
- persistent sidebar state;
- collapsible/resizable panels;
- keyboard-first navigation;
- command palette (`Ctrl+K`);
- context menus;
- drag/drop where useful;
- optimistic UI only where safe;
- background synchronization;
- SSE/WebSocket realtime updates as appropriate;
- virtualized lists;
- skeleton states instead of full-page loading screens;
- toasts for transient status;
- modal dialogs only for genuinely important decisions;
- persistent user UI preferences;
- deep links to OCC sessions;
- browser back/forward support without destroying application state;
- automatic reconnect and recovery banners;
- system/light/dark theme support;
- remember pane widths, sidebar state, selected session, density, and safe preferences.

PWA support may be added if useful, but **installing a PWA must not be required** for normal use.

## 1.4 Information density

Support three density preferences:

```text
Compact      → developer/terminal-like
Comfortable  → default
Spacious     → touch-friendly
```

The default should be compact-to-comfortable, not spacious.

## 1.5 Messaging-first interaction

OCC is primarily a group messaging/control workspace.

Participants:

```text
👤 Human
🤖 ChatGPT
🧑‍💻 OpenCode
⚙ OCC System
🔧 Tool
🖥 Process
```

Messages must visually distinguish sender, role, route, delivery status, and execution state.

Support:

- Markdown;
- syntax-highlighted code;
- copy buttons;
- file/diff cards;
- tool cards;
- process cards;
- question cards;
- recommendation cards;
- approval cards;
- reply/thread relationships;
- search;
- unread indicators;
- new-message indicator when scrolled away from bottom;
- sticky date separators;
- streaming responses;
- delivery/retry state;
- virtualized long histories.

The composer must clearly show where the message will go before sending.

Example:

```text
Target: 🧑‍💻 OpenCode   Mode: ⚡ Execute
```

or:

```text
Target: 🤖 ChatGPT     Mode: 🧠 Ask
```

## 1.6 UI must never lie

If OCC has not confirmed provider delivery, do not show the message as delivered.

If a provider disconnects, show that state.

If a process state is uncertain, show `UNKNOWN`.

If a provider binding is stale, visibly mark it stale/recovery-required.

---

# 2. OCC SESSION MODEL — EXPLICIT PROVIDER BINDINGS

An OCC session is a durable workspace, not merely a browser tab.

When the user clicks **New Session**, the UI must ask for:

```text
Session name

OpenCode:
  ○ Create New Session
  ○ Select Existing Session
  ○ Unlinked

ChatGPT:
  ○ Create New Chat
  ○ Select Existing Chat
  ○ Unlinked
```

Example:

```text
OCC Session: Hermes OAuth Fix

OpenCode
  mode: existing
  projectId: project_x
  sessionId: session_x

ChatGPT
  mode: existing
  conversationId: conversation_x
```

The selected provider targets become deterministic destinations for future messages until explicitly changed.

## 2.1 Why this is mandatory

Never ask at message-send time:

> “Which ChatGPT chat should receive this?”

The answer is established when the OCC session is created/bound.

A message is routed through:

```text
Current OCC Session
        ↓
Routing policy
        ↓
Exact provider binding
        ↓
Exact provider target
```

## 2.2 Binding changes

Changing a binding affects future messages only.

Historical messages retain their original target/provenance.

Show a confirmation before switching an active provider binding.

Example:

```text
Current ChatGPT: OAuth Research
New ChatGPT: OAuth Debugging

Future messages will use the new conversation.
Existing messages will not be migrated.

[Cancel] [Switch]
```

## 2.3 Human-readable names

Show names/titles by default, not opaque IDs.

Advanced details may show:

```text
OpenCode projectId
OpenCode sessionId
ChatGPT conversationId
provider capability
last verified time
```

---

# 3. OPENCODE GLOBAL PROJECT + SESSION DISCOVERY

OCC must **not use the OpenCode TUI/folder presentation as its global source of truth**.

The TUI may display a scoped subset. OCC must independently discover all accessible projects and sessions through the current supported OpenCode server/API capabilities.

Before implementation, inspect the installed OpenCode version and current official documentation. Investigate and verify the current equivalents of:

```text
GET /project
GET /session
GET /session/:id
GET /global/event
```

Do not assume these paths remain unchanged.

## 3.1 Global vs project mode

Provide:

```text
🌎 All Projects
📁 Selected Project
```

Global mode must not accidentally inherit the current working directory.

## 3.2 OpenCode workspace browser

Example:

```text
OPEN CODE
────────────────────────────
🔍 Search projects & sessions

📁 Hermes
  🟢 Authentication
  🟢 OAuth debugging
  🟡 Database migration

📁 Nothing
  🟢 Upload flow
  🟢 OAuth callback

📁 OCC
  🟢 Native ChatGPT connector
```

Project click shows its sessions.

Support:

- server/index-backed search;
- pagination/infinite scrolling;
- virtualization;
- sort by recently updated/created;
- status filtering;
- project filtering.

## 3.3 Lightweight session index

Maintain metadata, not duplicate transcripts:

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

Fetch authoritative conversation content only when a session is opened.

## 3.4 Live synchronization

If supported by the installed OpenCode server, use its global event stream for live updates.

Do not rely on aggressive polling when a supported event stream exists.

Handle:

- session creation;
- session updates;
- message events;
- status events;
- reconnects;
- duplicate events;
- out-of-order events;
- missed events.

## 3.5 Startup/reconnect reconciliation

On startup/reconnect:

1. Connect to OpenCode.
2. Discover all projects.
3. Discover all globally accessible sessions.
4. Compare with OCC's index.
5. Add missing sessions.
6. Update changed metadata.
7. Detect unavailable/deleted sessions.
8. Preserve uncertainty as `UNKNOWN`.
9. Re-subscribe to events.
10. Record a reconciliation run.

Periodically reconcile to recover from missed events/OCC downtime.

## 3.6 Never silently hide sessions

If a discovered session has no currently resolvable project, display:

```text
⚠️ Unknown / Unresolved Project
```

Do not silently drop it.

Session identity must use stable IDs such as:

```text
(projectId, sessionId)
```

not titles.

Filesystem inspection may exist as a diagnostic/recovery fallback, but must not replace a supported OpenCode server/API as the primary discovery method.

---

# 4. ACTOR ROUTING + OBSERVER MODE

OCC must never blindly send every human message to both providers.

## 4.1 Roles

```text
EXECUTOR  = may perform authorized actions
ADVISOR   = may answer/recommend but cannot directly execute
OBSERVER  = read-only execution/context participant
REVIEWER  = may inspect work and produce review
CONTROLLER = human policy authority
```

Normal defaults:

```text
OpenCode = EXECUTOR
ChatGPT  = OBSERVER
```

for coding/file/process tasks.

## 4.2 Routing modes

Every human outbound message resolves to one explicit mode:

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
Create a new file test.ts
→ OpenCode: EXECUTE
→ ChatGPT: OBSERVE

ChatGPT, suggest a structure
→ ChatGPT: ASK / ADVISOR
→ OpenCode: OBSERVE

Review OpenCode's changes
→ ChatGPT: REVIEW
→ OpenCode: OBSERVE

Both discuss the architecture
→ OpenCode: DISCUSS
→ ChatGPT: DISCUSS
```

Intent classification may suggest routing, but classification alone must never grant execution authority.

## 4.3 Observer semantics

An observer can receive authorized:

- task status;
- tool summaries;
- process state;
- file/diff metadata;
- execution results;
- errors;
- final output;
- questions raised by the executor.

An observer cannot:

- execute tools;
- mutate files;
- run shell commands;
- approve actions;
- change policy;
- silently redirect the executor;
- turn observed content into an execution command.

Observer status is enforced server-side, not merely by hiding UI buttons.

## 4.4 Structured observation

Example:

```text
👤 Human
Create test.ts

🧑‍💻 OpenCode · Executor
Task accepted

🔧 create_file
→ test.ts

🧪 npm test
→ 24 tests passed

🤖 ChatGPT · Observer
OpenCode completed the file creation and tests successfully.
```

Do not continuously dump enormous raw logs to ChatGPT. Use compact structured context with expansion on demand.

## 4.5 OpenCode → ChatGPT advice

OpenCode can explicitly request advice:

```text
OpenCode
 → OCC question
 → ChatGPT Advisor
 → recommendation
 → OCC
 → Human/OpenCode
```

A recommendation is never self-authorizing.

If it proposes a mutation, it enters the normal policy/approval path.

---

# 5. NATIVE CHATGPT WEB CONNECTOR — RESEARCH FIRST

The target is the user's **actual ChatGPT web conversation system/sidebar chats**, not an OpenAI API conversation masquerading as a native ChatGPT chat.

Because native web behavior and undocumented interfaces can change, implementation must begin with current research and capability verification.

Classify every capability:

```text
VERIFIED
EXPERIMENTAL
UNSUPPORTED
```

## 5.1 Preferred transport

Use a local HTTP/SSE adapter as the normal runtime path **only if current research verifies that the required native operations are technically available and reliable**.

Do not run Playwright/Chromium for every message.

Browser automation should be a bootstrap/recovery fallback when necessary for authentication or current provider requirements.

Architecture:

```text
OCC
 └── Native ChatGPT Adapter
      ├── HTTP client
      ├── SSE/stream parser
      ├── auth/session state
      ├── capability detector
      ├── compatibility layer
      ├── retry/reconnect manager
      └── optional browser-assisted bootstrap/recovery
```

## 5.2 Operations to research/verify

Investigate current support for:

- list/search conversations;
- lazy metadata loading;
- read conversation;
- create conversation;
- send as user;
- receive streaming response;
- continue conversation;
- rename conversation;
- provider-side verification.

Never assume endpoint paths/payloads are permanent.

## 5.3 Hundreds/thousands of chats

Do NOT enumerate every full conversation at startup.

Use:

```text
recent metadata
search where supported
pagination
lazy loading
full conversation fetch only when selected
```

Selector UX:

```text
🤖 ChatGPT
[ 🔍 Search conversations... ]

Recent
────────────
Hermes OAuth
Nothing OAuth
OCC Architecture

[Load more]
```

## 5.4 Security

Never store ChatGPT cookies/tokens in GitHub, source control, ordinary logs, or an unencrypted remote database.

Prefer OS-protected local credential/session storage and least-privilege handling.

Never route authentication through an unknown third-party proxy.

## 5.5 Native ChatGPT vs OpenAI API

Keep them completely separate:

```text
Native ChatGPT connector
→ actual ChatGPT web conversation target

OpenAI API connector
→ API target
```

Never represent an API conversation as a native sidebar chat.

---

# 6. COMPLETE MESSAGE FLOW

## 6.1 Human → OpenCode execution

```text
Human UI
 → OCC authentication
 → routing resolution
 → authorization/policy
 → persist human message
 → create OpenCode/EXECUTE route
 → ChatGPT receives observation context only if bound/policy-enabled
 → exact bound OpenCode project/session
 → execution
 → normalized tool/process events
 → OCC persistence
 → observer context stream
 → realtime UI
```

## 6.2 Human → ChatGPT advice/review

```text
Human UI
 → OCC authentication
 → routing resolution
 → policy
 → persist human message
 → ChatGPT/ASK or REVIEW route
 → exact bound ChatGPT conversation
 → genuine response
 → provenance verification
 → persist response
 → optional OpenCode observation context
 → UI
```

## 6.3 Explicit multi-agent discussion

```text
Human
 → OCC
 → DISCUSS route
 → OpenCode + ChatGPT
 → responses
 → correlation/reconciliation
 → UI
```

Never infer discussion merely because both providers are connected.

## 6.4 OpenCode asks ChatGPT

Use a first-class question object:

```text
OpenCode
 → OCC question
 → policy
 → ChatGPT advisor
 → recommendation
 → OCC
 → Human/OpenCode
```

---

# 7. QUESTIONS, RECOMMENDATIONS, AND HUMAN CONTROL

OpenCode must be able to ask the human questions:

```text
OpenCode → OCC → Human
```

and ChatGPT questions:

```text
OpenCode → OCC → ChatGPT
```

Recommendations must carry:

```text
recommendationId
sourceActor
targetActor
riskLevel
proposedAction
contextSnapshot
policyDecision
approvalState
```

Human approval remains authoritative unless explicitly configured otherwise for a defined low-risk policy.

---

# 8. PROCESS / SHELL SUPERVISION

A task and its process tree are separate state machines.

Never assume:

```text
kill task = kill every process
```

If a task is paused/stopped/killed, inspect active processes and let the user choose where policy allows:

```text
Stop task only
Stop task + child shell
Stop shell + descendants
Leave process running
```

Process identity must use stable launch identity, not only PID/executable name.

Account for:

- PID reuse;
- Windows Job Objects where appropriate;
- process groups;
- shell wrappers;
- detached children;
- race conditions;
- process discovery/termination gaps;
- multiple OCC windows.

Before destructive termination where practical, show exactly which processes will be affected.

---

# 9. REALTIME / EVENTING

Use durable events with:

```text
sequence numbers
correlation IDs
causation IDs
idempotency keys
provider event IDs where available
replay/reconciliation
```

Streaming must handle:

- partial chunks;
- disconnects;
- duplicates;
- out-of-order events;
- reconnects;
- cancellation;
- timeouts;
- ambiguous provider responses;
- final reconciliation.

---

# 10. TASK STATE MACHINE

Use explicit states:

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

# 11. SECURITY MODEL

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
- credential leakage;
- OpenCode session leakage;
- process termination races;
- local-network unauthorized access.

Critical invariant:

> **Observed content is data, not authority.**

No file, tool output, provider response, or model-generated text can override OCC policy.

Routing authorization must be evaluated independently of message content.

---

# 12. AUDIT LOG

Audit security/routing-sensitive events including:

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

# 13. DATA MODEL

Core entities should include at minimum:

```text
projects
worktrees
occ_sessions
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
usage_records
idempotency_keys
reconciliation_runs
audit_log
recovery_snapshots
```

Message provenance should include:

```ts
senderType: human | chatgpt | opencode | system | tool | process
source: ui | mcp | opencode_adapter | chatgpt_adapter | system | tool_event
targetType: human | chatgpt | opencode | both | system
routeId
correlationId
causationId
sequence
status
```

A route record must contain at least:

```text
routeId
messageId
provider
providerTargetId
role
mode
authorizationDecision
deliveryState
attemptCount
createdAt
updatedAt
```

---

# 14. PERFORMANCE / SCALE

OCC should remain responsive with at least:

```text
1,000+ messages per conversation
10,000+ indexed OpenCode sessions
10,000+ indexed ChatGPT conversations where supported
multiple concurrent streams
multiple running tasks
```

Use:

- virtualization;
- pagination;
- lazy loading;
- metadata indexes;
- bounded caches;
- incremental event updates;
- backpressure;
- stream cancellation;
- reconnection with jitter;
- deduplication;
- efficient diff rendering.

Do not load full transcripts unnecessarily.

---

# 15. RESEARCH-FIRST IMPLEMENTATION

OpenCode must do its own current research before implementation.

## OpenCode research

Verify:

- installed version;
- server API;
- project discovery;
- global session discovery;
- session details;
- event streams;
- event schemas;
- current CLI behavior;
- Windows-specific behavior;
- process/task behavior.

## MCP/ChatGPT research

Verify:

- current MCP specification;
- SDK;
- actual ChatGPT MCP connectivity model;
- transport;
- authentication;
- native ChatGPT web behavior;
- conversation operations;
- message operations;
- streaming;
- compatibility limitations.

## Windows process research

Verify:

- Windows Job Objects;
- process groups;
- shell wrappers;
- detached process behavior;
- PID reuse;
- race-safe process identity;
- termination semantics.

Produce a compatibility report:

```text
Capability
Source/version tested
VERIFIED / EXPERIMENTAL / UNSUPPORTED
Test evidence
Known limitations
Fallback
```

Never fake unsupported functionality.

---

# 16. EDGE CASES — MUST DESIGN FOR THEM

At minimum:

1. Coding request while ChatGPT disconnected.
2. ChatGPT request while OpenCode disconnected.
3. Both selected but policy permits only one executor.
4. OpenCode session deleted externally.
5. ChatGPT conversation renamed externally.
6. TUI hides a session that server discovery finds.
7. Session project cannot currently be resolved.
8. OCC misses events while offline.
9. Native ChatGPT interface changes.
10. ChatGPT stream disconnects mid-response.
11. OpenCode emits duplicate events.
12. Human message retry after timeout.
13. Binding changed while task runs.
14. Routing changed after queueing but before dispatch.
15. Observer receives prompt injection inside execution output.
16. ChatGPT recommends a dangerous/unauthorized action.
17. OpenCode asks ChatGPT while human approval is pending.
18. Two OCC windows control the same process.
19. Shell spawns detached children.
20. PID is reused.
21. OCC crashes during delivery.
22. OCC crashes during process termination.
23. Network disappears during streaming.
24. Provider responds ambiguously after timeout.
25. OCC session has both providers unlinked.
26. One new provider + one existing provider.
27. Thousands of OpenCode sessions.
28. Thousands of ChatGPT chats.
29. Duplicate titles across projects/providers.
30. Authentication expiry.
31. Observer attempted as executor.
32. ChatGPT response mistaken as command.
33. OpenCode response attributed to ChatGPT.
34. Historical messages incorrectly migrated after rebinding.
35. Browser fallback launches while HTTP adapter is active.
36. User refreshes browser during a running task.
37. Browser has multiple OCC tabs/windows.
38. Network changes from Wi-Fi to Ethernet while connected.
39. Provider target becomes temporarily unavailable.
40. A stale event arrives after a newer authoritative state.

For each define:

```text
expected state
user-visible behavior
provider behavior
recovery behavior
audit behavior
```

---

# 17. TESTING / ACCEPTANCE

Acceptance must be real, end-to-end, evidence-based.

## UI

- Website loads without full-page navigation for normal interaction.
- Session switching is instant.
- Sidebar is persistent and resizable.
- Message list is virtualized.
- Command palette works.
- Keyboard shortcuts respect authorization.
- Dark/light/system themes work.
- Browser refresh preserves recoverable application state.
- Back/forward navigation works without breaking the app shell.
- Reconnect banner and recovery work.
- UI does not look like a generic SaaS dashboard.

## OpenCode discovery

- Create multiple projects.
- Create sessions in multiple projects.
- Verify all discoverable projects appear.
- Verify sessions hidden by a scoped TUI view can still appear in OCC.
- Verify global/project filtering.
- Verify search/pagination.
- Verify event updates.
- Verify startup reconciliation.
- Verify unresolved sessions remain visible.

## Provider binding

- New + new.
- Existing + existing.
- New + existing.
- One unlinked.
- Both unlinked.
- Binding change while idle.
- Binding change while task active.
- Historical message provenance remains unchanged.

## Actor routing

- `Create a file` → OpenCode executes; ChatGPT observes.
- `Modify code` → OpenCode executes; ChatGPT observes.
- `Ask ChatGPT for architecture` → ChatGPT answers; OpenCode observes.
- `Review OpenCode` → ChatGPT reviews.
- `Both discuss` → both receive explicit discussion route.
- Observer execution attempt → rejected.
- ChatGPT recommendation → never automatically executed without authorized policy.

## Native ChatGPT

Where current research verifies support:

- search/list conversations;
- select exact conversation;
- create conversation;
- send user message;
- receive stream;
- continue;
- rename;
- reconnect;
- verify provider-side result.

Unsupported capabilities must be recorded as `UNSUPPORTED`, never fabricated.

## Process controls

- pause during shell command;
- stop task without unrelated process termination;
- selected process-tree termination;
- detached-child handling;
- PID reuse handling;
- crash recovery during termination.

---

# 18. IMPLEMENTATION DELIVERABLES

Before declaring complete, provide:

```text
architecture document
research/compatibility report
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
frontend messaging workspace
UI/UX specification
test suite
end-to-end verification report
recovery runbook
security documentation
```

Keep provider-specific logic behind adapters so provider changes do not require rewriting OCC's core domain model or frontend.

---

# 19. FINAL ARCHITECTURAL PRINCIPLE

OCC is **not a dumb message relay** and not a normal dashboard.

It is a policy-controlled coordination workspace:

```text
                         👤 HUMAN
                       CONTROLLER
                           │
                           ▼
                    ┌──────────────┐
                    │     OCC      │
                    │              │
                    │ UI / Chat    │
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
       EXECUTOR                    ADVISOR/OBSERVER
              │                         │
       exact project+session      exact conversation
              │                         │
              └────────────┬────────────┘
                           ▼
                    shared OCC context
```

The critical invariants are:

> **Every message has an explicit destination, role, authorization decision, and delivery state.**
>
> **Connection does not imply execution. Observation does not imply authority. Recommendation does not imply approval. Provider binding does not imply that every message must be sent to that provider. The OpenCode TUI does not define OCC's global session universe. The browser contains OCC, but OCC must feel like an application rather than a website.**

Build around correctness, recoverability, security, explicit human control, deterministic routing, real provider verification, global session visibility, and a fast messaging-first app-like web experience.