# OpenCode Control Center (OCC)
# ULTRA-DETAILED IMPLEMENTATION & UI INTEGRATION PROMPT

## 0. PURPOSE

This document is the implementation specification for upgrading the existing OpenCode Control Center (OCC) project. It is intentionally written as an engineering specification rather than a visual mockup request.

The existing repository, `Unknowmyt1M/OpenCode-ChatGpt`, must be treated as the source code being upgraded. Before changing anything, inspect the entire repository and compare the real implementation against `project.md` and this specification.

IMPORTANT:
- Do not assume that a feature described in documentation already exists.
- Do not replace working architecture merely because a different implementation is easier.
- Preserve working OpenCode SDK, event, approval, TOCTOU, process-control, SSE, SQLite/WAL, MCP, routing, audit, and frontend foundations where they are sound.
- Where documentation and code disagree, inspect the code and make the implementation correct.
- Do your own current technical research before making provider/API/SDK decisions.
- Think through failure modes, race conditions, reconnects, duplicated events, stale bindings, process lifetime, authorization, and provider unavailability before implementing.

The final product must be a REAL OCC application, not a polished fake UI.

---

# 1. CORE PRODUCT MODEL

OCC is a web application that acts as a controlled orchestration and observation layer between a human user, OpenCode, and ChatGPT.

Conceptually:

    Human
      |
      v
     OCC
      |
      +--------------------+
      |                    |
      v                    v
   OpenCode             ChatGPT
   EXECUTOR       OBSERVER / ADVISOR / REVIEWER

A single OCC session may bind to:

    - one OpenCode project/workspace
    - one exact OpenCode session
    - one ChatGPT conversation

Bindings must be explicit and persistent.

OCC must never silently guess the destination of a message when an explicit provider binding is required.

---

# 2. MOST IMPORTANT BEHAVIOR: EXECUTOR VS OBSERVER

The default coding workflow is:

    Human message
        |
        v
    OCC Router
        |
        +--> OpenCode = EXECUTOR
        |
        +--> ChatGPT = OBSERVER

If the user explicitly asks ChatGPT for advice/review, ChatGPT may become ADVISOR/REVIEWER while OpenCode remains executor unless the user changes the routing policy.

Example:

User:
    "Create a new file called config.ts"

Expected behavior:

    OpenCode receives executable instruction.
    ChatGPT does NOT independently execute the instruction.
    ChatGPT may receive an observation/event/context copy if configured.

The UI must clearly show:

    OpenCode · EXECUTOR
    ChatGPT · OBSERVER

An observer must never be able to mutate the project merely because it received the same human message.

This is a policy requirement, not just a visual distinction.

---

# 3. EXPLICIT USER ROUTING MUST BE AUTHORITATIVE

The current frontend has routing controls, but the backend must actually honor them.

When the user selects:

    Target: OpenCode
    Mode: Execute

the backend must receive and validate that explicit selection.

Do not silently discard routingTarget/routingMode and re-infer everything only from message text.

The backend must remain authoritative and must validate:

    - allowed target
    - allowed mode
    - session binding
    - actor permissions
    - provider availability
    - task state
    - approval requirements
    - safety/policy restrictions

Frontend controls are UX controls, not security boundaries.

If the frontend asks for an invalid combination, the backend must reject it cleanly.

---

# 4. OCC SESSION CREATION AND PROVIDER BINDING

When the user clicks `+ New Session`, show a real session creation workflow.

The workflow must include:

    OCC Session Name

    OpenCode Binding:
        [ Create New OpenCode Session ]
        [ Select Existing OpenCode Session ]
        [ Leave Unlinked ]

    ChatGPT Binding:
        [ Create New ChatGPT Chat ]
        [ Select Existing ChatGPT Chat ]
        [ Leave Unlinked ]

For OpenCode existing-session selection, show:

    Project
      -> Session

For ChatGPT existing-chat selection, do NOT blindly render hundreds/thousands of chats.

Use:

    - search-first selection
    - recent chats
    - pagination/infinite loading
    - optional filters
    - explicit `Create New` action

The user must know exactly which external resources the OCC session is bound to.

Example:

    OCC Session: Hermes OAuth Fix

    OpenCode
      Project: Hermes
      Session: ses_123...

    ChatGPT
      Conversation: chat_456...

Bindings must be persisted durably, not only held in an in-memory Map.

---

# 5. EXACT OPENCODE SESSION TARGETING

A major requirement is that an OCC session must be able to target an exact existing OpenCode session.

Do NOT merely maintain a hidden adapter-local session ID.

Required relationship:

    OCC session ID
          <->
    OpenCode project ID
          <->
    OpenCode exact session ID

When an OCC message is routed to OpenCode:

    1. load OCC session binding
    2. resolve exact OpenCode project/session
    3. verify it is still valid
    4. send to that exact session
    5. correlate returned events with that exact binding

If the bound OpenCode session disappears:

    - do not silently create a replacement session
    - mark the binding stale/missing
    - explain the state to the user
    - offer `Reconnect`, `Select Existing`, or `Create New`

Automatic creation is allowed only when the user explicitly selected `Create New` or an equivalent policy permits it.

---

# 6. GLOBAL OPENCODE PROJECT + SESSION DISCOVERY

Do not rely on what the OpenCode TUI happens to display.

OCC must maintain a global reconciled index of available OpenCode projects and sessions.

The discovery layer must handle:

    - pagination
    - large session counts
    - projects with many sessions
    - sessions whose project metadata is temporarily unavailable
    - deleted sessions
    - stale sessions
    - running sessions
    - recently updated sessions
    - unknown/unresolved project associations

Never use a fixed first-page limit as the definition of "all sessions".

If the provider API has pagination, consume it correctly.

If the provider exposes a better project/session discovery mechanism, research and use it.

Maintain an OCC-side index/cache so the UI can query:

    All Projects
    All Sessions
    Recently Updated
    Running
    Paused
    Failed
    Unknown / Unresolved

Sessions that cannot currently be mapped to a project must not simply disappear.

Display them as:

    Unknown / Unresolved Project

until reconciliation succeeds.

---

# 7. CHATGPT CONVERSATION MANAGEMENT

The current MCP integration must not be confused with native conversation management.

OCC needs a provider abstraction that can represent:

    - list conversations
    - search conversations
    - paginate conversations
    - create conversation
    - rename conversation where supported
    - retrieve conversation metadata where supported
    - send user-authored messages where supported
    - receive/stream responses where supported
    - bind an OCC session to a conversation
    - detect unavailable/unsupported capabilities

Do NOT invent an official API that does not exist.

Before implementation, research the currently available official and technically reliable mechanisms.

If a native API is unavailable, document the exact capability boundary and implement the safest supported connector architecture.

Do not build a brittle browser automation dependency unless there is no viable alternative and the user explicitly accepts it.

If a capability cannot be reliably implemented, expose it as:

    Unsupported
    Unavailable
    Experimental
    Requires configuration

rather than faking success.

---

# 8. CHATGPT MESSAGE DESTINATION MUST BE EXPLICIT

When the user sends a message from OCC, the destination must be deterministic.

Example:

    OCC Session: Hermes

    ChatGPT binding:
        Conversation `chat_123`

Then a ChatGPT-targeted message must go to:

    `chat_123`

not to a random/new conversation.

If the OCC session is configured with:

    ChatGPT = Create New Chat

then the system must create/bind that chat as part of session setup, not on every message.

If no ChatGPT binding exists and the user targets ChatGPT:

    ask for a binding or follow the explicit session policy.

Never silently choose an arbitrary conversation.

---

# 9. CHATGPT OBSERVER PIPELINE

For a normal coding task:

    Human
      |
      v
    OCC
      |
      +--> OpenCode EXECUTOR
      |
      +--> ChatGPT OBSERVER

The observer copy must be clearly marked as an observation/context event.

ChatGPT must not receive the message in a way that makes it an independent executor unless the user explicitly changes the mode.

Observer context should include only the information allowed by the active context policy.

The system must track:

    sender
    target
    role
    mode
    route
    task ID
    OCC session ID
    OpenCode session ID
    ChatGPT conversation ID
    delivery state
    correlation ID
    causation ID
    timestamps

This metadata is essential for debugging and auditability.

---

# 10. MESSAGE EVENT MODEL

The central conversation is a live event stream, not a fake chat transcript.

Every event/message should have enough metadata to answer:

    Who produced it?
    Who was the intended target?
    Which OCC session does it belong to?
    Which provider session does it belong to?
    Is it executable, observational, advisory, or informational?
    Was it delivered?
    Did it fail?
    Is it streaming?
    Which task caused it?
    Which event caused it?

Use durable event IDs and sequence numbers.

Frontend must deduplicate by event identity.

Do not append duplicate events after reconnect.

---

# 11. CONTEXT ENGINE

Implement a real context layer rather than dumping the entire event history into every provider.

The context engine should support:

    - relevant recent messages
    - active task information
    - current OpenCode project/session metadata
    - recent tool events
    - relevant file changes
    - important approvals
    - recent failures
    - user decisions
    - compact historical summaries
    - token/context budgeting

Research provider context limits and design an adaptive strategy.

Use summarization/compression where appropriate.

Do not allow unbounded conversation history to grow provider requests indefinitely.

Context selection must be deterministic enough to debug.

---

# 12. REALTIME EVENT ARCHITECTURE

Use the existing OCC event architecture as the foundation.

Preferred flow:

    OpenCode / ChatGPT / OCC process
            |
            v
       Provider Adapter
            |
            v
        Normalizer
            |
            v
         Event Bus
            |
            +--> Durable Store
            |
            +--> SSE/WebSocket
            |
            v
        Frontend Store
            |
            v
             UI

The frontend should not independently invent provider state.

The backend is authoritative.

---

# 13. RECONCILIATION AND RECOVERY

Reconnect is not enough.

After a provider disconnect/reconnect, OCC must reconcile state.

For OpenCode:

    - sessions
    - projects
    - active tasks
    - recent events
    - questions
    - permissions

For ChatGPT connector:

    - connection status
    - bound conversation
    - delivery status
    - pending requests where supported

For OCC:

    - tasks
    - messages
    - bindings
    - processes
    - approvals

Do not blindly replay events that may already have been persisted.

Use IDs/sequences/correlation IDs to reconcile safely.

---

# 14. PROVIDER HEALTH

Do not infer provider health from unrelated UI state.

Bad examples:

    OpenCode connected = active shell process exists
    ChatGPT connected = session status is working

Instead create explicit provider health state:

    connected
    connecting
    disconnected
    degraded
    authentication_required
    configuration_error
    unsupported
    stale

Provider health must be generated from actual adapter/transport state.

---

# 15. PROCESS CONTROL

Preserve the current ownership/start-time safeguards and improve them where needed.

Required operations:

    Pause
    Resume
    Stop
    Kill Process Tree
    Detach

The frontend must NEVER directly kill arbitrary Windows processes.

All destructive operations go through the backend process supervisor.

Use strong process identity, ideally including:

    PID
    process start time
    owner/task identity

Research Windows Job Objects or other robust process-group mechanisms where appropriate.

Handle:

    process already exited
    PID reuse
    concurrent stop requests
    OCC restart
    orphaned process
    child process survival
    timeout
    forced kill

The UI must distinguish:

    Stop Task
    Stop Process
    Kill Process Tree
    Detach / Leave Running

where those semantics differ.

---

# 16. APPROVAL + TOCTOU

Preserve and strengthen the existing approval/TOCTOU architecture.

Flow:

    policy check
       |
       v
    approval required?
       |
       v
    human approval
       |
       v
    verify current context/fingerprint
       |
       v
    execute

If relevant context changes after approval, invalidate the approval.

Never assume an old approval remains valid after:

    file change
    command change
    session change
    task change
    provider change
    policy change

---

# 17. TOOL PROVENANCE

Do not hard-code actor attribution such as:

    every tool call = ChatGPT

Tool execution metadata must identify:

    actual provider
    actor
    role
    task
    OCC session
    route
    authorization
    approval
    correlation ID

Audit logs must be truthful.

---

# 18. DURABLE SESSION BINDINGS

Do not store important provider bindings only in an in-memory Map.

Persist at minimum:

    OCC session ID
    OpenCode project ID
    OpenCode session ID
    ChatGPT conversation ID
    binding status
    created time
    updated time
    stale/error information

On OCC restart:

    load bindings
    reconcile them
    mark stale ones
    never silently create replacements

---

# 19. UI: APP-LIKE WEB EXPERIENCE

OCC remains a website/web application.

Do NOT build Electron/Tauri/native desktop software.

However, it must feel like a desktop application.

Visual/interaction inspiration:

    VS Code
    Discord
    Telegram
    modern IDEs

The browser should feel like the container, not the product identity.

Required characteristics:

    persistent application shell
    dense information layout
    instant session switching
    resizable panels
    command palette
    keyboard shortcuts
    context menus
    compact controls
    subtle animations
    persistent UI preferences
    realtime state
    no unnecessary page reloads

---

# 20. UI REFERENCE IMAGE INSTRUCTION

A UI reference image will be attached to the implementation conversation.

IMPORTANT:

THE IMAGE IS A VISUAL REFERENCE, NOT A STATIC MOCKUP.

Do not:

    - hardcode its text
    - hardcode its session names
    - hardcode its project names
    - hardcode its timestamps
    - fake terminal output
    - fake tool events
    - fake provider statuses
    - create a screenshot-like page

Use it only to understand:

    information hierarchy
    layout
    density
    spacing
    actor distinction
    navigation
    panel structure
    message composition
    routing controls
    desktop-app feel

Then implement the real backend-connected application.

The UI should be better than a pixel-for-pixel copy if usability/accessibility/realtime architecture requires changes.

---

# 21. UI STRUCTURE

Recommended shell:

    +--------------------------------------------------------------+
    | OCC | Session | Connection | Search | Command | Settings    |
    +----------+-----------------------------------+---------------+
    |          |                                   |               |
    | Projects |       Conversation                | Context       |
    | Sessions |                                   | Files         |
    | Search   |  Human                            | Changes       |
    | Filters  |  OpenCode                         | Process       |
    |          |  ChatGPT                          | History       |
    |          |  Tool Events                      |               |
    |          |                                   |               |
    |          |  Composer                         |               |
    +----------+-----------------------------------+---------------+

This is conceptual, not a pixel constraint.

---

# 22. SIDEBAR

The sidebar must support:

    Projects
    Sessions
    Search
    Filters
    Recent
    Pinned/Favorites

Large datasets require:

    virtualization
    lazy loading
    search
    filtering
    pagination/reconciliation

Display:

    active session
    unread/attention state
    provider status where useful
    running/paused state

Do not render thousands of session DOM nodes at once.

---

# 23. CONVERSATION VIEW

Message types should include:

    Human
    OpenCode
    ChatGPT
    OCC/System
    Tool
    Approval
    Question
    Error

Every provider message must visually expose role/provenance.

Example:

    OpenCode · EXECUTOR

versus:

    ChatGPT · OBSERVER

Use more than color to distinguish actors.

Support:

    streaming
    tool event expansion
    execution status
    delivery status
    retry where safe
    timestamps
    correlation information
    jump-to-latest
    long-output collapse

---

# 24. MESSAGE COMPOSER

The composer must contain:

    Target: [ OpenCode v ]
    Mode:   [ Execute v ]

    [ message ................................................ ] [Send]

Targets may include:

    OpenCode
    ChatGPT
    Both

Modes may include only valid combinations:

    Execute
    Ask
    Review
    Discuss
    Observe

The backend validates every combination.

The composer should explain why an option is unavailable rather than silently failing.

---

# 25. CONTEXT PANEL

Use a resizable contextual panel with tabs such as:

    Context
    Files
    Changes
    Process
    History

Context should show:

    OpenCode
      Role: EXECUTOR
      Project: <real project>
      Session: <real session>
      Status: <real health>

    ChatGPT
      Role: OBSERVER/ADVISOR/REVIEWER
      Conversation: <real chat>
      Status: <real health>

No fake data.

---

# 26. FILE EXPLORER

Files must represent the currently bound OpenCode project/worktree.

Support:

    folders
    files
    modified markers
    selection
    lazy expansion
    refresh
    preview
    diff
    loading/error states

Do not hardcode example filenames from the reference image.

---

# 27. CHANGES / DIFF VIEW

Show actual changes from the bound project/session.

Include:

    modified files
    added files
    deleted files
    renamed files where available
    diff summary
    expandable diff

Connect to actual OpenCode/project state.

---

# 28. PROCESS / TERMINAL VIEW

The terminal/process view must be connected to the actual OCC process supervisor.

Display:

    command
    shell
    process identity
    state
    duration
    output
    exit status
    cancellation status

Large output must be virtualized or bounded.

---

# 29. HISTORY

History should provide useful session/task chronology without duplicating the full event log unnecessarily.

Support:

    recent tasks
    completed tasks
    failed tasks
    approvals
    important events

Clicking a history item should navigate to the relevant durable event/task context.

---

# 30. COMMAND PALETTE

Implement a real command palette.

Examples:

    New OCC Session
    Switch Session
    Search Sessions
    Select OpenCode Session
    Select ChatGPT Conversation
    Create ChatGPT Conversation
    Toggle Context Panel
    Toggle Sidebar
    Open Files
    Open Changes
    Show Processes
    Reconnect Providers
    Stop Current Task
    Kill Process Tree
    Open Settings

Commands must call real application actions.

---

# 31. KEYBOARD SHORTCUTS

At minimum consider:

    Ctrl+K      Command Palette
    Ctrl+N      New OCC Session
    Ctrl+F      Search
    Esc         Close modal/popover
    Enter       Send
    Shift+Enter New line

Make shortcuts discoverable and configurable where appropriate.

---

# 32. PERFORMANCE

Must remain usable with:

    thousands of sessions
    thousands of messages
    long tool outputs
    multiple active tasks
    many realtime events

Use:

    virtualization
    lazy loading
    pagination
    memoization
    bounded caches
    event batching
    debouncing
    cancellation
    backpressure

Do not render unbounded data into the DOM.

---

# 33. ERROR STATES

Every backend operation needs meaningful states:

    loading
    success
    empty
    disconnected
    reconnecting
    timeout
    unauthorized
    forbidden
    unavailable
    stale
    unsupported
    configuration error
    recovery required

Never hide a provider failure behind a generic fake-success UI.

---

# 34. SECURITY

The frontend is never trusted.

Backend must enforce:

    authentication
    authorization
    provider binding ownership
    routing policy
    tool policy
    approval requirements
    session ownership
    process ownership

Protect MCP transport appropriately.

Do not assume a tunnel alone is sufficient unless the tunnel configuration itself provides the required authenticated boundary.

Validate all external/provider data.

Do not log secrets, API keys, cookies, or sensitive provider credentials.

---

# 35. TESTING REQUIREMENTS

Add/expand tests for:

### Routing

    Human -> OpenCode
    Human -> ChatGPT
    Human -> Both
    Observer mode
    Advisor mode
    Invalid route

### Session binding

    New OpenCode session
    Existing OpenCode session
    New ChatGPT conversation
    Existing ChatGPT conversation
    Missing binding
    Stale binding
    Deleted provider session

### Reconnect

    SSE reconnect
    OpenCode reconnect
    provider unavailable
    duplicate events
    missed events
    reconciliation

### Process control

    stop
    kill tree
    pause
    resume
    detach
    already exited
    PID reuse
    concurrent stop
    OCC restart

### Approval

    approval required
    approval denied
    approval expires
    TOCTOU mismatch
    successful execution

### Large data

    >500 sessions
    >1000 sessions
    thousands of messages
    large tool logs

### Security

    unauthorized route
    unauthorized tool
    unauthorized process
    forged provider ID
    forged session ID
    invalid binding

---

# 36. IMPLEMENTATION WORKFLOW

Before writing code:

1. Read the entire repository.
2. Read `project.md`.
3. Inspect frontend architecture.
4. Inspect bridge/backend architecture.
5. Inspect event store.
6. Inspect routing engine.
7. Inspect OpenCode adapter.
8. Inspect MCP server.
9. Inspect tool service.
10. Inspect process controller.
11. Inspect session binding storage.
12. Inspect all existing tests.
13. Study the attached UI reference image.
14. Map every visible UI capability to a real backend source.
15. Identify documentation-vs-code mismatches.
16. Research current provider/API capabilities.
17. Produce an internal implementation plan.
18. Implement backend contracts first where required.
19. Implement provider adapters.
20. Implement durable state.
21. Implement event/reconciliation logic.
22. Implement frontend state.
23. Implement UI.
24. Add tests.
25. Run typecheck/build/test/lint.
26. Perform security review.
27. Perform failure-mode review.
28. Re-test all critical flows end-to-end.

Do not stop merely because the UI looks correct.

---

# 37. EDGE CASES YOU MUST THINK THROUGH YOURSELF

Do not limit implementation to this list. Research and identify additional cases.

At minimum consider:

1. User sends a message while OpenCode is disconnected.
2. User sends a message while ChatGPT is disconnected.
3. Both providers disconnect simultaneously.
4. OCC restarts while OpenCode is running.
5. OCC restarts while a ChatGPT delivery is pending.
6. OpenCode session is deleted externally.
7. ChatGPT conversation becomes unavailable.
8. A selected project is renamed/deleted.
9. Two OCC sessions bind to the same OpenCode session.
10. Two users/processes attempt conflicting actions.
11. User clicks Send twice rapidly.
12. Network retries cause duplicate delivery.
13. SSE reconnect replays an already persisted event.
14. Tool event arrives before its task event.
15. Tool result arrives after task cancellation.
16. Process exits between status read and kill request.
17. PID is reused by another process.
18. Approval becomes stale during execution.
19. Provider sends malformed/unknown event.
20. Session count exceeds 500/1000/10000.
21. Long ChatGPT conversation exceeds context limits.
22. OpenCode emits a burst of thousands of events.
23. Browser tab sleeps and reconnects later.
24. Browser opens two OCC tabs.
25. User changes binding while a task is executing.
26. User tries to kill a process owned by another task.
27. Provider authentication expires.
28. API rate limits occur.
29. Provider returns partial/streamed response then disconnects.
30. A task is paused and OCC restarts.
31. A task is killed while a shell command is running.
32. User selects `Observe` but provider attempts mutation.
33. ChatGPT observer receives insufficient context.
34. OpenCode asks a question while user is viewing another OCC session.
35. A provider returns an event for an unknown OCC binding.
36. A stale browser submits an old routing target.
37. User changes session while a streamed response is active.
38. User creates a new session but closes the dialog halfway through.
39. New provider resource creation succeeds but binding persistence fails.
40. Binding persistence succeeds but provider creation fails.

For every case, define the authoritative state transition and recovery behavior.

---

# 38. NO FAKE IMPLEMENTATION

Never implement fake:

    ChatGPT replies
    OpenCode messages
    terminal output
    provider health
    sessions
    projects
    files
    tool events
    process states
    connection states

Never use timeout-based fake activity to make the UI look alive.

If something is unsupported, say so in the UI.

---

# 39. IMAGE REFERENCE IMPLEMENTATION PROMPT

When this prompt and the attached UI reference image are provided together, follow this exact principle:

    IMAGE = VISUAL REFERENCE
    REPOSITORY = IMPLEMENTATION SOURCE OF TRUTH
    BACKEND/API/EVENTS = BEHAVIOR SOURCE OF TRUTH

Study the image for:

    - application shell
    - sidebar structure
    - conversation density
    - message grouping
    - actor identity
    - provider indicators
    - routing controls
    - right-side contextual workspace
    - compact desktop-like controls
    - spacing
    - hierarchy
    - typography
    - interaction affordances

Then recreate the design using the project's real components and backend state.

Do not copy example data.

Do not build a static visual clone.

Do not sacrifice backend correctness for visual similarity.

If the image suggests a feature that the backend does not support, implement the backend capability first if it is technically feasible. If it is not feasible, show an honest unsupported/unavailable state.

The resulting UI should feel like a production-grade developer application, not a dashboard template.

---

# 40. FINAL ACCEPTANCE CRITERIA

The work is NOT complete until all of the following are true:

[ ] OCC sessions are durable.
[ ] OpenCode project/session bindings are durable.
[ ] Exact OpenCode sessions can be selected and targeted.
[ ] OpenCode project/session discovery is global and paginated/reconciled.
[ ] ChatGPT conversation binding is explicit.
[ ] ChatGPT conversation selection is search-first and scalable.
[ ] New ChatGPT conversations can be created where supported.
[ ] OCC messages have deterministic provider targets.
[ ] Explicit UI routing is honored by the backend.
[ ] OpenCode executor behavior is real.
[ ] ChatGPT observer behavior is real where supported.
[ ] ChatGPT advisor/reviewer behavior is explicit.
[ ] Actor provenance is correct.
[ ] Tool provenance is correct.
[ ] Approval/TOCTOU remains enforced.
[ ] Process ownership remains enforced.
[ ] Process control handles race conditions.
[ ] Reconnect and reconciliation are implemented.
[ ] Duplicate events are safely deduplicated.
[ ] Provider health is real.
[ ] Context engine exists.
[ ] Context is bounded and relevant.
[ ] UI is a polished app-like web experience.
[ ] UI is not hardcoded to the reference image.
[ ] Files/Changes/Process/History views are real.
[ ] Session creation/binding workflow is real.
[ ] Command palette is functional.
[ ] Keyboard shortcuts work.
[ ] Large session/message lists are virtualized/paginated.
[ ] Error states are explicit.
[ ] Security boundaries are backend-enforced.
[ ] No secrets are exposed in logs/frontend.
[ ] Automated tests cover critical flows.
[ ] Build/typecheck/lint/test pass.
[ ] End-to-end testing confirms real provider interactions.

---

# 41. FINAL INSTRUCTION TO THE IMPLEMENTING AGENT

Think like a senior systems engineer, product designer, security engineer, and QA engineer simultaneously.

Do your own research.

Do not blindly follow the existing architecture if the code proves that an assumption is wrong.

Do not blindly follow this document if current provider capabilities have changed.

When there are multiple approaches:

    research
    compare
    reason about trade-offs
    choose the most reliable approach
    document the decision
    implement it

Prioritize:

    correctness
    real integration
    deterministic routing
    durable state
    recoverability
    security
    performance
    maintainability
    excellent UX

Most importantly:

    DO NOT BUILD A BEAUTIFUL FAKE UI.

    BUILD A BEAUTIFUL REAL OCC APPLICATION.

The attached image should guide how OCC feels.
The repository should determine what already exists.
The backend should determine what is real.
The provider capabilities should determine what is possible.
The final implementation must make all four work together.
