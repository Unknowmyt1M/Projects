# ChatGPT ↔ OpenCode ↔ Human — Complete Message, MCP, Tool & Event Flow Specification

## Purpose

This document is a mandatory companion to `MASTER_PROMPT.md` and `UI_MESSAGING_APP_SPEC.md`.

The implementation MUST explicitly define how messages and events travel between:

- 👤 Human/User
- 🤖 ChatGPT through MCP
- 🧑‍💻 OpenCode
- 🧩 OpenCode plugin/adapter
- 🧠 OCC Control Hub
- 🌐 Messaging UI
- 🔧 Shell/process supervisor

Do NOT implement this product as a vague “ChatGPT ↔ OpenCode bridge”. The Control Hub is the source of truth and the messaging application is a real conversation client over that state.

---

# 1. THE REAL ARCHITECTURE

```text
                         👤 HUMAN
                            │
                            │ browser / mobile UI
                            ▼
                 ┌────────────────────────┐
                 │   OCC Messaging UI     │
                 │   Group Conversation   │
                 └───────────┬────────────┘
                             │
                      WebSocket/SSE
                             │
                             ▼
                 ┌────────────────────────┐
                 │     OCC CONTROL HUB    │
                 │                        │
                 │ Conversation Service  │
                 │ Task Service           │
                 │ Event Bus              │
                 │ Policy Engine          │
                 │ Question Service       │
                 │ Process Supervisor     │
                 │ Persistence             │
                 └──────┬─────────┬───────┘
                        │         │
                 OpenCode API     │ MCP
                        │         │
                        ▼         ▼
               🧑‍💻 OpenCode   🤖 ChatGPT
                        │         │
                 OpenCode        MCP client
                 Plugin/Adapter
                        │
                        ▼
                 tools / messages /
                 permissions / shell
```

The browser MUST NOT communicate directly with OpenCode or ChatGPT for core state. It communicates with OCC.

ChatGPT MUST NOT be treated as the database or message broker.

OpenCode MUST NOT be treated as the database or message broker.

OCC is the durable source of truth.

---

# 2. THREE DIFFERENT COMMUNICATION PLANES

Keep these planes separate.

## A. Human/UI plane

```text
Browser/mobile
    ↕
WebSocket/SSE + HTTP API
    ↕
OCC Control Hub
```

Used for:

- displaying the group conversation;
- sending human messages;
- approvals;
- task controls;
- process controls;
- live events.

## B. OpenCode integration plane

```text
OCC Control Hub
    ↕
OpenCode Adapter / Plugin
    ↕
OpenCode runtime
```

Used for:

- sending user/ChatGPT instructions to OpenCode;
- receiving OpenCode messages/events;
- observing tool calls;
- observing permission requests;
- observing session state;
- controlling supported session lifecycle operations.

Use only currently supported OpenCode APIs/plugin hooks. Inspect the installed OpenCode version before implementation.

## C. MCP / ChatGPT plane

```text
ChatGPT MCP client
       ↕
MCP server endpoint
       ↕
OCC MCP adapter
       ↕
OCC services
```

Used for:

- ChatGPT discovering OCC capabilities;
- reading authorized conversations/context;
- reading OpenCode questions;
- proposing answers;
- creating/controlling tasks when permitted;
- sending messages;
- inspecting selected task/process state.

MCP is an AI-facing protocol interface. It is NOT the browser's realtime message bus.

---

# 3. IMPORTANT: A MESSAGE DOES NOT “MAGICALLY” REACH CHATGPT

This must be explicitly understood by the implementation.

If OpenCode sends:

> “I need to know whether we should use SQLite or PostgreSQL.”

that event first reaches OCC.

```text
OpenCode
  ↓
Plugin/Adapter
  ↓
OCC event
  ↓
Persistent question/message
```

ChatGPT can then access that state through the MCP interface using the supported MCP request/task/interaction mechanism.

Do NOT implement an imaginary internal API such as:

```text
sendMessageDirectlyIntoChatGPT("...")
```

unless the actual connected ChatGPT/MCP client supports an equivalent standardized interaction mechanism.

The system MUST work even if ChatGPT is temporarily disconnected.

---

# 4. HOW OPENCODE MESSAGES ARE CAPTURED

OpenCode produces multiple classes of events.

The OpenCode adapter must normalize supported events into OCC domain events.

At minimum support, when available in the installed version:

```text
message.updated
message.part.updated
message.removed
session.created
session.status
session.idle
session.error
session.updated
permission.asked
permission.replied
tool.execute.before
tool.execute.after
file.edited
command.executed
shell.env
todo.updated
```

The exact event names MUST be verified against the installed/current OpenCode API rather than copied blindly.

Normalization example:

```text
OpenCode event
   ↓
OpenCodeAdapter.normalize()
   ↓
OCCEvent {
   eventId,
   sequence,
   conversationId,
   taskId,
   sessionId,
   actor: "opencode",
   kind: "message.text",
   payload,
   createdAt,
   correlationId
}
```

Then:

```text
OCCEvent
   ├── persisted
   ├── task state updated
   ├── conversation projection updated
   ├── browser event emitted
   └── available to MCP context retrieval
```

---

# 5. HOW USER MESSAGES REACH OPENCODE

The user types into the group chat:

> “Implement login with server-side sessions.”

The browser sends:

```json
{
  "conversationId": "conv_42",
  "sender": "human",
  "target": "opencode",
  "type": "text",
  "content": "Implement login with server-side sessions.",
  "clientMessageId": "client_msg_123"
}
```

OCC MUST:

1. authenticate the user;
2. authorize the conversation/project;
3. validate task state;
4. assign a durable `messageId`;
5. persist the message;
6. publish the message-created event;
7. route the instruction to the correct OpenCode session;
8. record delivery status;
9. reconcile the resulting OpenCode events.

Conceptually:

```text
User
 ↓
UI
 ↓
POST / conversation command
 ↓
OCC
 ├── persist message
 ├── authorize
 ├── route
 └── emit event
 ↓
OpenCode Adapter
 ↓
OpenCode session prompt/instruction
```

If OpenCode rejects/fails the instruction, the message MUST become `failed` or an equivalent explicit state. Never show it as successfully delivered.

---

# 6. HOW USER MESSAGES REACH CHATGPT

The composer must let the user select:

```text
Target: OpenCode | ChatGPT | Both
```

If the user selects ChatGPT:

```text
User
 ↓
OCC
 ↓
persist human message
 ↓
MCP-facing conversation state
 ↓
ChatGPT reads/receives it through the supported MCP interaction
 ↓
ChatGPT response
 ↓
OCC persists ChatGPT message
 ↓
UI renders it
```

Do not pretend that an MCP server can force arbitrary text into a ChatGPT conversation if the connected client does not support that interaction pattern.

The implementation must verify the actual ChatGPT MCP integration behavior available at deployment time and adapt the UX accordingly.

If the user sends a message to ChatGPT while ChatGPT is unavailable:

```text
WAITING_FOR_CHATGPT
```

The message remains durable and visible.

---

# 7. HOW CHATGPT READS OPENCODE'S MESSAGES

ChatGPT should receive the minimum useful context, not the entire raw event database.

Provide MCP context operations such as:

```text
occ_get_conversation
occ_get_messages
occ_get_recent_events
occ_get_task_summary
occ_get_pending_questions
occ_get_selected_diff
occ_get_relevant_tool_activity
```

A context response should be structured:

```json
{
  "conversation": {
    "id": "conv_42",
    "taskId": "task_42",
    "project": "Hermes",
    "status": "WAITING_FOR_CHATGPT"
  },
  "messages": [
    {
      "id": "m101",
      "sender": "human",
      "content": "Implement authentication"
    },
    {
      "id": "m102",
      "sender": "opencode",
      "content": "I found two possible approaches."
    }
  ],
  "pendingQuestions": [
    {
      "id": "q7",
      "question": "JWT or sessions?",
      "options": ["JWT", "sessions"]
    }
  ],
  "cursor": {
    "lastSequence": 102
  }
}
```

Never expose secrets merely because they happen to be in logs/files.

---

# 8. HOW OPENCODE ASKS CHATGPT A QUESTION

This is a dedicated protocol flow.

```text
OpenCode
  ↓
question/decision request
  ↓
OpenCode plugin/adapter
  ↓
OCC Question Service
  ↓
persist question
  ↓
conversation event
  ↓
UI shows question card
  ↓
MCP exposes pending question
  ↓
ChatGPT proposes answer
  ↓
OCC policy engine
  ↓
Human approval if required
  ↓
OpenCode receives final decision
```

The question must have:

```text
questionId
conversationId
taskId
sessionId
source
question
context
options[]
recommendedAnswer?
status
requiresHumanApproval
createdAt
expiresAt
resolvedBy
resolvedAt
answer
```

---

# 9. CHATGPT ANSWER IS NOT AUTOMATICALLY HUMAN AUTHORITY

Example:

```text
OpenCode:
Which database should I use?

ChatGPT:
SQLite.
```

OCC policy decides what happens next.

### Policy A — AI auto-answer allowed

```text
ChatGPT answer
 ↓
policy check
 ↓
approved automatically
 ↓
OpenCode
```

### Policy B — Human approval required

```text
ChatGPT answer
 ↓
recommendation stored
 ↓
Human sees:
“ChatGPT recommends SQLite”
 ↓
Approve / Edit / Reject
 ↓
final answer
 ↓
OpenCode
```

The UI must never label an unapproved AI recommendation as the user's decision.

---

# 10. HOW OPENCode SEES THE FINAL ANSWER

After policy resolution:

```text
FinalDecision
{
  questionId,
  answer,
  source: human | chatgpt-approved | policy,
  approvedBy,
  timestamp,
  correlationId
}
```

OCC sends the final answer through the supported OpenCode session/plugin mechanism.

The OpenCode conversation should then show:

```text
👤 Aditya
Use SQLite.

⚙ Control Center
Decision delivered to OpenCode.

🧑‍💻 OpenCode
Understood. Continuing with SQLite.
```

This closes the loop and prevents invisible AI-to-AI communication.

---

# 11. TOOL CALL FLOW

OpenCode tools such as `read`, `grep`, `glob`, `edit`, `write`, `apply_patch`, `bash`, `task`, `webfetch`, `websearch`, `question`, and MCP-backed tools are OpenCode-side capabilities.

OCC MUST NOT impersonate these tools unless necessary.

For an OpenCode tool execution:

```text
OpenCode model decides to call tool
          ↓
OpenCode permission evaluation
          ↓
plugin/adapter observes supported tool event
          ↓
OCC records tool activity
          ↓
UI shows tool card
          ↓
tool executes inside OpenCode
          ↓
result event
          ↓
OCC persists result summary
          ↓
UI updates
          ↓
relevant result can become ChatGPT context
```

For high-volume output, store output separately and show only a summary/preview in the conversation.

Do NOT send every byte of shell output to ChatGPT by default.

---

# 12. OPENCode PERMISSION FLOW

OpenCode already has its own permission system. The integration must respect it rather than inventing a competing permission protocol blindly.

Current OpenCode permissions can allow, ask, or deny tool actions, with granular rules depending on version/configuration.

When OpenCode requests approval:

```text
OpenCode
 ↓
permission request
 ↓
OCC adapter
 ↓
OCC policy layer
 ↓
Human UI approval card
 ↓
approved/rejected
 ↓
OpenCode permission response
```

If the installed OpenCode version already exposes a public permission event/control API, use it.

Do not fake a “permission granted” event in OCC without actually resolving the OpenCode permission request.

---

# 13. SHELL TOOL + PROCESS SUPERVISION FLOW

When OpenCode executes `bash` or equivalent:

```text
OpenCode
 ↓
bash/tool execution
 ↓
OCC observes/associates execution
 ↓
Process Supervisor registers owned process tree
 ↓
UI displays process card
 ↓
OpenCode receives output
 ↓
OCC receives tool/process completion
```

Every task-owned process must have ownership metadata.

For Windows, prefer robust process ownership/isolation mechanisms such as Job Objects where practical. Never kill processes solely by executable name.

---

# 14. PAUSE FLOW

User clicks `Pause Agent`.

```text
UI
 ↓
OCC pause request
 ↓
Task state = PAUSING
 ↓
OpenCode interrupt/pause mechanism supported by installed version
 ↓
OpenCode acknowledgement/status
 ↓
Task state = PAUSED
```

If a shell command is still running and cannot be safely paused:

```text
Task = PAUSE_REQUESTED
Process = RUNNING
```

UI must explain this.

Do not silently kill the process.

---

# 15. STOP FLOW

User clicks `Stop Agent`.

OCC:

1. requests OpenCode interruption/termination using supported API;
2. inspects task-owned processes;
3. if processes remain, presents process policy/confirmation unless an explicit policy already resolves it;
4. stops only selected/owned processes;
5. reconciles final state.

Never execute:

```text
taskkill /IM node.exe /F
```

for a task-specific action.

---

# 16. EVENT BUS

All planes feed one internal event bus:

```text
                    ┌── UI
                    │
OpenCode ─┐         ├── MCP context
ChatGPT ─┼─► EVENT ┼── persistence
Human ───┘         ├── audit log
                    └── state projections
```

Every event should include:

```text
eventId
sequence
source
actor
kind
conversationId?
taskId?
sessionId?
messageId?
processId?
correlationId?
causationId?
createdAt
payload
```

Use monotonic per-conversation/per-task sequence numbers for replay/order.

---

# 17. MESSAGE DELIVERY AND CURSORS

MCP/browser/OpenCode connections can disconnect.

Therefore:

```text
MCP connection lifetime != conversation lifetime
WebSocket lifetime != task lifetime
OpenCode session lifetime != OCC task lifetime
```

Persist events/messages first.

Clients maintain a cursor:

```text
conversationId
lastSeenSequence
```

On reconnect:

```text
client → OCC: lastSeenSequence = 120
OCC → client: events 121..147
OCC → client: current snapshot
```

The client must deduplicate by event ID/sequence.

---

# 18. CHATGPT DISCONNECTED

If ChatGPT is unavailable:

- OpenCode may continue according to policy.
- Questions remain queued.
- Human can answer directly.
- The UI shows `ChatGPT unavailable`.
- No question is silently discarded.
- When ChatGPT reconnects, it can retrieve unresolved questions/context through MCP.

Do not make OpenCode dependent on a permanent ChatGPT socket.

---

# 19. OPENCode DISCONNECTED

If OpenCode disappears:

- preserve the last known conversation state;
- mark session disconnected/recovery-required as appropriate;
- preserve task/process ownership information;
- attempt reconnection/reconciliation;
- never fabricate messages indicating that work continued.

---

# 20. HUMAN MESSAGE TARGETING

The UI composer MUST make routing explicit.

```text
Target:
[ OpenCode ▼ ]
```

Options:

```text
OpenCode
ChatGPT
Both
Control Center
```

For `Both`, the system must create explicit recipient metadata and deliver through each supported channel separately.

Never rely on parsing text such as:

> “ChatGPT, tell OpenCode...”

as a privileged routing mechanism.

Natural-language mentions can be convenience UX, but structured target metadata controls routing.

---

# 21. CHATGPT CONTEXT POLICY

ChatGPT should not receive everything by default.

Context selection should include only what is relevant:

```text
recent messages
+ unresolved questions
+ task summary
+ recent OpenCode actions
+ relevant tool results
+ selected files/diffs
+ human constraints
```

Sensitive paths, secrets, tokens, cookies, environment values, and unrelated project data must be filtered.

---

# 22. MCP TOOL DESIGN RULES

MCP tools are not raw RPC endpoints for every internal OCC function.

Tools must:

- be descriptive;
- have strict JSON schemas;
- return structured results;
- be authorization-aware;
- be idempotent where retries are possible;
- expose stable identifiers;
- avoid returning secrets;
- avoid unrestricted arbitrary shell execution.

Suggested tools:

```text
occ_status
occ_list_conversations
occ_get_conversation
occ_get_messages
occ_get_recent_events
occ_send_message
occ_create_task
occ_get_task
occ_start_task
occ_pause_task
occ_resume_task
occ_stop_task
occ_list_pending_questions
occ_get_question
occ_propose_answer
occ_request_more_context
occ_submit_human_answer
occ_list_pending_approvals
occ_approve
occ_reject
occ_list_task_processes
occ_get_task_process
occ_request_process_stop
occ_get_task_summary
occ_get_selected_diff
```

Exact names may be adjusted after checking the current MCP SDK and ChatGPT connector/tooling constraints.

---

# 23. MCP SPECIFICATION COMPATIBILITY

Verify the current MCP specification and SDK before implementation.

As of August 2026, the current MCP specification is the `2026-07-28` release, which introduced a stateless protocol core, Tasks, MCP Apps, Multi Round-Trip Requests, and updated authorization behavior.

The implementation must NOT assume that an old stateful MCP session is the only way to deliver this architecture.

Use the current standard mechanisms where the actual ChatGPT/MCP client supports them.

If a feature such as server-initiated interaction, Tasks, elicitation, streaming, or MCP Apps is not supported by the target ChatGPT integration, implement a durable polling/retrieval fallback through normal MCP tools instead of inventing a proprietary protocol.

---

# 24. MCP APPS / UI POSSIBILITY

Investigate whether the deployed ChatGPT MCP environment supports MCP Apps or equivalent interactive tool-rendered UI.

However, the primary OCC UI remains the dedicated messaging web application.

MCP-rendered UI may be used for compact approval/question/context interactions but MUST NOT become the only UI.

---

# 25. IDENTITY / AUTHORIZATION

Every request must be attributable to:

```text
human
chatgpt
opencode
system
```

Do not allow a ChatGPT-originated request to impersonate the human.

Do not allow OpenCode tool output to impersonate ChatGPT.

Do not allow natural-language message text to grant permissions.

The authorization layer operates on structured identity + action + resource.

---

# 26. IDEMPOTENCY

Every mutation that can be retried should have an idempotency key.

Examples:

```text
create_task
send_message
start_task
stop_task
submit_answer
approve_permission
```

A network retry must not create duplicate tasks/messages or resolve the same question twice.

---

# 27. CONCURRENCY / RACE CONDITIONS

Handle:

- human answers while ChatGPT is answering;
- ChatGPT answers after human already answered;
- OpenCode finishes while stop is requested;
- process exits while stop is requested;
- two browser tabs approve the same request;
- two ChatGPT calls mutate the same task;
- OpenCode reconnects with stale state;
- duplicate event delivery.

Use optimistic concurrency/version checks where appropriate.

Final state resolution must be deterministic and auditable.

---

# 28. REQUIRED SEQUENCE DIAGRAMS

The implementation documentation MUST include exact sequence diagrams for:

1. Human → OpenCode message.
2. Human → ChatGPT message.
3. ChatGPT → OpenCode instruction.
4. OpenCode → ChatGPT question.
5. ChatGPT answer → Human approval → OpenCode.
6. OpenCode tool call → OCC → UI.
7. OpenCode shell → process supervisor → UI.
8. Human pause while shell is running.
9. Human stop while shell is running.
10. ChatGPT disconnect/reconnect.
11. OpenCode disconnect/reconnect.
12. Browser reconnect/event replay.
13. Duplicate MCP request.
14. Stale question/approval.

For each diagram document:

- sender;
- receiver;
- transport;
- request/response schema;
- persistent records;
- event IDs;
- authorization checks;
- retries;
- failure states;
- UI rendering.

---

# 29. ACCEPTANCE TEST: THE COMPLETE LOOP

The product is not complete until this exact scenario works:

```text
👤 Aditya
“Implement authentication in Hermes.”

↓

OCC persists user message.

↓

OpenCode receives instruction.

↓

🧑‍💻 OpenCode
“I inspected the project. I have two architecture options.”

↓

OCC stores OpenCode message.

↓

ChatGPT can retrieve it through MCP.

↓

🤖 ChatGPT
“I recommend server-side sessions.”

↓

OCC stores ChatGPT recommendation.

↓

👤 Aditya
“Approved.”

↓

OCC resolves decision.

↓

OpenCode receives final instruction.

↓

🧑‍💻 OpenCode
“Continuing with server-side sessions.”

↓

OpenCode calls read/grep/edit/bash as appropriate.

↓

OCC records tool/process events.

↓

UI shows live tool/process cards.

↓

A shell process is still running when Aditya clicks Pause.

↓

OCC pauses/interupts the agent where supported but does NOT blindly kill the process.

↓

UI shows the process decision.

↓

Aditya chooses “Keep running”.

↓

OpenCode remains paused; process remains owned/running.

↓

Aditya resumes.

↓

OpenCode continues/reconciles state.

↓

Task completes.

↓

UI shows final summary + files changed + tests + process cleanup.
```

If this loop cannot be demonstrated end-to-end with real APIs and real state, the implementation is not finished.

---

# 30. FINAL RULE

Never describe the system internally as merely:

```text
ChatGPT ↔ OpenCode
```

The correct model is:

```text
Human
  ↕
Messaging UI
  ↕
OCC Control Hub
  ↕                 ↕
OpenCode         ChatGPT MCP
  ↕                 ↕
Tools/processes   AI reasoning
```

The Control Hub owns durable state and routing.

The messaging UI is the human-facing conversation.

OpenCode is the execution agent.

ChatGPT is an MCP-connected reasoning/review participant.

The human remains the final authority.
