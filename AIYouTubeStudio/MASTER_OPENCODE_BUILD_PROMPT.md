# Draxen Motion — Master OpenCode Build Prompt

> Project codename: **Draxen Motion**
> Mission: build a data-first, AI-driven autonomous YouTube animation/content company.

## 0. FIRST: DO NOT BUILD YET

Before writing substantial code, **fully analyze this entire prompt**. Treat it as a proposal, not unquestionable truth.

Then independently perform deep, current research (prefer official/primary sources and current 2026 documentation) across:

- AI video generation
- AI 3D generation
- text/image-to-3D
- character consistency
- environment/location generation
- rigging and motion generation
- text/image-to-video
- video-to-video
- camera/action control
- AI voice/TTS
- AI music and SFX
- video editing/assembly
- upscaling/interpolation
- AI thumbnail generation
- YouTube Data API
- YouTube Analytics API
- AI agent/orchestration frameworks
- workflow engines and queues
- PostgreSQL/Supabase and alternatives
- object storage
- serverless/container infrastructure
- free GPU compute
- free CPU compute
- legitimate free tiers that do not require a credit card
- open-source/local models
- model routing
- observability
- ML experimentation and feature stores

For every serious candidate evaluate current limits, API access, automation support, free-tier requirements, credit-card requirement, commercial/YouTube usage, licensing, watermarks, quality, reliability, latency, rate limits, storage, GPU requirements, ToS restrictions, and replaceability.

**Do not trust outdated tutorials or SEO lists when official documentation is available.** Record important sources, dates, caveats, and confidence in `docs/research/`.

## 1. CHALLENGE THE PLAN

After research, produce a review of this prompt before implementation.

Explicitly identify:

1. What is strong in the proposed architecture.
2. What is outdated.
3. What is unnecessarily complicated.
4. What should be removed.
5. What is missing.
6. Which newer 2026 technologies should replace proposed components.
7. Which free services are actually practical.
8. Which assumptions are risky.
9. Which AI 3D/video approaches are realistically capable of production automation.
10. Whether Blender is actually needed.
11. Better alternatives to Blender if available.
12. The cheapest reliable architecture.
13. The best free-first architecture.
14. The architecture you would personally choose after research.

**Give these recommendations to the user before making major irreversible architectural choices.**

Do not blindly follow this prompt if research demonstrates a substantially better approach.

## 2. MISSION

The final system should eventually be capable of:

Research → trend discovery → topic selection → story planning → script → storyboard → character/location/asset planning → visual/3D/video generation → animation → voice → music/SFX → editing → QA → thumbnail → title/description/metadata → YouTube upload → analytics collection → experiments → ML analysis → strategy optimization → improved future content.

The goal is not merely an AI video generator.

The goal is a **software-defined autonomous media company**.

Human involvement should decrease gradually, while safety, observability, reversibility, and quality increase.

## 3. BUILD IN PHASES

Never attempt the whole system in one pass.

### Phase 0 — Research & Architecture

Deliver:

- ecosystem research
- provider comparison
- infrastructure comparison
- architecture proposal
- alternatives
- risks
- cost/free-tier analysis
- ERD
- event model
- security model
- agent architecture
- provider abstraction
- roadmap

No large implementation until the architecture has been reviewed.

### Phase 1 — Data Foundation

Build first:

- database
- migrations
- event/telemetry system
- core entities
- experiment tracking
- prompt versioning
- model/provider versioning
- agent-run logging
- API foundation
- tests

**Data collection starts on Day 1.**

### Phase 2 — Orchestration

Build:

- job system
- queue
- workflow/state machine
- retries
- timeouts
- provider fallback
- resumable jobs
- concurrency/resource controls
- dead-letter handling

### Phase 3 — Content Intelligence

Build:

- research agent
- trend agent
- topic agent
- story agent
- script agent
- storyboard agent

### Phase 4 — Visual Production

Research first, then integrate the best current providers.

Support an abstraction for:

- character generation
- environment generation
- asset generation
- animation
- scene generation
- camera control
- voice
- music/SFX
- editing
- rendering

Blender is **optional**, not mandatory. Use it only if research proves it is useful.

### Phase 5 — QA & Publishing

Build:

- automated video QA
- thumbnail generation
- metadata generation
- YouTube OAuth
- upload/scheduling
- analytics ingestion

Initially require human approval before publishing.

### Phase 6 — Analytics

Collect time-series performance data and build dashboards/reports.

### Phase 7 — ML

Start with statistical analysis and baseline models before advanced RL.

Potential models:

- topic performance prediction
- CTR prediction
- retention prediction
- provider selection
- production cost/time prediction
- upload-time optimization

Later evaluate contextual bandits, Bayesian optimization, evolutionary methods, and RL only if data volume justifies them.

### Phase 8 — Autonomous Optimization

Build:

observe → analyze → hypothesis → experiment → measure → update strategy → repeat.

### Phase 9 — Autonomous Media Company

High-level goal in, continuous content strategy/production/learning out.

## 4. DATA-FIRST RULE

**Do not wait for ML to start collecting data.**

Every meaningful action must emit structured telemetry.

Preserve:

- raw events
- normalized data
- derived features
- experiments
- decisions
- outcomes
- model predictions
- provider usage
- failures

Architecture:

`Raw Events → Processed Data → Features → ML Dataset → Models → Decisions → Outcomes`

Never delete raw telemetry merely because a feature has already been derived.

## 5. CORE DATA

Track production data:

- project/channel/video IDs
- topic/niche/subniche
- story type/structure
- script length
- duration
- scene count
- character/location/asset IDs
- provider/model/version
- prompts and parameters
- generation time
- render time
- failures/retries
- resource usage
- credits/cost estimate
- human intervention
- QA scores

Track publishing data:

- upload/schedule/publish timestamps
- title/description/metadata
- thumbnail variant
- language
- target audience
- duration

Track YouTube performance snapshots:

- views
- impressions
- CTR
- average view duration
- average percentage viewed
- watch time
- likes
- comments
- shares
- subscribers gained
- returning viewers
- traffic sources
- audience retention
- legally/officially available aggregate audience metrics
- revenue metrics where officially available

Collect snapshots at useful ages such as 1h, 6h, 24h, 48h, 7d, 30d, rather than storing only final numbers.

## 6. EVENT SYSTEM

Implement a generic event schema similar to:

```json
{
  "event_id": "evt_001",
  "event_type": "script_generated",
  "project_id": "project_001",
  "video_id": "video_001",
  "agent_id": "script_agent",
  "provider": "provider_x",
  "model": "model_y",
  "timestamp": "ISO-8601",
  "duration_ms": 12450,
  "success": true,
  "metadata": {}
}
```

Track events including:

- research_started/completed
- topic_selected
- script_started/generated
- storyboard_generated
- asset_requested/generated
- scene_generated
- animation_generated
- voice_generated
- music_generated
- render_started/completed/failed
- QA_started/passed/failed
- thumbnail_generated
- metadata_generated
- upload_started/completed
- analytics_collected
- experiment_created/completed
- agent_decision
- provider_failure
- fallback_triggered
- retry
- workflow_started/completed/failed

## 7. DECISION LOGGING

Every important AI decision must be traceable.

Store:

- agent
- input
- decision
- reason
- alternatives
- selected option
- confidence
- model/provider
- prompt version
- timestamp
- eventual outcome

This data will later allow the system to learn whether its own decisions were good.

## 8. DATABASE

Design an extensible relational schema. Evaluate PostgreSQL/Supabase and alternatives before locking the choice.

Likely entities:

- projects
- channels
- videos
- video_versions
- scripts
- storyboards
- scenes
- characters
- locations
- assets
- asset_versions
- generation_jobs
- render_jobs
- voice_jobs
- music_jobs
- experiments
- experiment_variants
- youtube_metrics
- youtube_metric_snapshots
- audience_metrics
- agent_runs
- agent_decisions
- model_runs
- provider_usage
- events
- errors
- prompts
- prompt_versions
- workflow_runs
- workflow_steps

Create an ERD before implementation.

Use proper indexes, constraints, migrations, timestamps, versioning, and data-retention strategy.

## 9. AGENTS

Do not create agents merely to increase the agent count.

Potential specialized agents:

- CEO/Orchestrator
- Research
- Trend
- Topic
- Story
- Script
- Storyboard
- Character
- Environment
- Scene
- Animation
- Voice
- Music/SFX
- Editing
- Thumbnail
- Metadata
- QA
- Publishing
- Analytics
- Experiment
- ML
- Cost Optimization

Each agent must have:

- explicit responsibility
- typed inputs/outputs
- tool permissions
- timeout
- retry policy
- telemetry
- cost tracking
- failure behavior

## 10. ORCHESTRATOR

Implement a central workflow layer managing:

- dependencies
- state
- queues
- priorities
- retries
- timeouts
- provider fallback
- concurrency
- resource requirements
- cancellation
- resumability

A failed step must not unnecessarily regenerate completed work.

## 11. STATE MACHINE

Use explicit states such as:

IDEA → RESEARCHED → SELECTED → SCRIPTED → STORYBOARDED → ASSETS_READY → SCENES_READY → VOICE_READY → ASSEMBLY_READY → RENDERING → QA → READY_TO_UPLOAD → UPLOADING → PUBLISHED → ANALYTICS_COLLECTION → LEARNING.

Support recoverable failure states.

## 12. VISUAL/3D ABSTRACTION

Create a provider-independent scene specification.

Example:

```json
{
  "scene_id": "scene_001",
  "duration": 8,
  "location": {
    "id": "abandoned_city",
    "time": "night",
    "weather": "rain"
  },
  "characters": [
    {"id": "kai", "action": "running", "emotion": "fear"},
    {"id": "wolf", "action": "chasing"}
  ],
  "camera": {"type": "tracking", "movement": "forward"},
  "lighting": "cinematic",
  "style": "stylized_realistic"
}
```

The scene specification must not depend on a particular vendor.

Adapters may target:

- AI video providers
- AI 3D providers
- Blender
- other renderers
- open-source pipelines

## 13. CHARACTER & WORLD CONSISTENCY

Characters are persistent entities with:

- identity
- canonical appearance
- reference assets
- 3D assets if available
- voice
- personality
- clothing
- animation profile
- version history

Locations/worlds are persistent entities with:

- canonical description
- references/assets
- geometry where applicable
- lighting presets
- camera presets
- environmental details
- version history

Never regenerate recurring characters/locations from scratch without canonical identity context.

## 14. EXPERIMENTATION

Treat content production as experimentation.

Potential variables:

- topic
- hook
- story structure
- duration
- title
- thumbnail
- voice
- animation style
- pacing
- camera style
- upload time

Store hypothesis, variants, metrics, winner, confidence, and follow-up decision.

## 15. QA

Before upload verify:

- file exists
- resolution
- FPS
- codec
- duration
- audio presence
- audio/video sync
- black/corrupt frames
- missing scenes
- silent sections
- volume
- subtitle alignment if used
- visual artifacts
- duplicate frames
- watermark presence
- prompt leakage
- character consistency
- dialogue correctness
- scene ordering
- metadata consistency
- content safety

Failed QA must block publishing.

## 16. HUMAN APPROVAL & AUTONOMY

Use configurable autonomy levels:

- Level 0: manual
- Level 1: AI-assisted
- Level 2: AI-generated + human approval
- Level 3: automatic publishing with QA
- Level 4: autonomous operation

Start at Level 2.

Only increase autonomy after measurable reliability.

## 17. YOUTUBE

Use official YouTube APIs.

Implement OAuth, secure token storage, upload, scheduling, metadata, thumbnails, playlists if useful, and analytics ingestion.

Never reverse-engineer authentication or bypass platform security.

Never commit OAuth tokens, cookies, API keys, or service credentials.

## 18. PROVIDER ROUTING

Build provider adapters and a router.

Router inputs should consider:

- availability
- quota
- cost
- latency
- quality
- historical success rate
- task type

If a provider fails, use a configured fallback.

Track provider performance so routing can eventually learn from historical outcomes.

## 19. FREE-FIRST INFRASTRUCTURE

Research and prefer legitimate free resources where practical.

Evaluate current 2026 options such as:

- Cloudflare Workers/Queues
- GitHub Actions
- Google Colab
- Kaggle
- legitimate free container/serverless tiers
- legitimate cloud free tiers
- local PC
- existing compute
- open-source/local models

Do not design around questionable RDP/VPS offers, fake accounts, ToS circumvention, or free-tier abuse.

Temporary GPU environments should be treated as disposable workers.

Design for provider replacement.

## 20. COST TRACKING

Even free services must have a theoretical cost record.

Track:

- provider
- model
- tokens/credits
- compute time
- storage
- bandwidth
- estimated monetary cost

Eventually optimize quality/cost instead of blindly selecting the largest model.

## 21. ML

Do not immediately jump to complicated reinforcement learning.

Start with:

analytics → statistical analysis → feature engineering → baseline models → prediction → experimentation → optimization.

Potential ML targets:

- topic score
- CTR prediction
- retention prediction
- provider selection
- production cost/time
- upload timing

Advanced methods are allowed only when justified by sufficient data.

## 22. SELF-IMPROVEMENT

Implement this feedback loop eventually:

Observe → Analyze → Hypothesis → Experiment → Measure → Update knowledge → Change future strategy → Observe again.

Do not allow unrestricted autonomous production-code modification.

AI-generated code changes must pass tests, validation, security checks, and have rollback capability. Human approval should be required initially.

## 23. PROMPT & MODEL VERSIONING

Never silently overwrite important prompts.

Track:

- prompt ID
- version
- content
- model
- parameters
- creation date
- performance

Track model/provider versions and their performance/cost/failure rates.

## 24. SECURITY

Use:

- environment variables
- secret management
- encryption where appropriate
- least privilege
- token rotation
- audit logs
- secure OAuth handling

Never commit `.env`, tokens, cookies, passwords, private keys, or credentials.

## 25. OBSERVABILITY

Every service should produce structured logs with:

- timestamp
- service
- correlation/job ID
- project/video ID
- agent
- log level
- message
- metadata
- duration
- status

A single video should be traceable end-to-end.

## 26. ERROR HANDLING

Implement:

- timeouts
- exponential backoff
- bounded retries
- provider fallback
- circuit breaking where useful
- dead-letter queues
- resumable jobs
- error classification
- human escalation

No infinite retry loops.

## 27. TESTING

Build from the beginning:

- unit tests
- integration tests
- API tests
- schema tests
- workflow tests
- provider adapter tests
- security tests
- end-to-end tests

Use mocked providers so tests do not waste AI credits.

## 28. DOCUMENTATION

Maintain:

```text
docs/
├── architecture.md
├── roadmap.md
├── research/
├── database.md
├── events.md
├── agents.md
├── providers.md
├── security.md
├── ml.md
├── deployment.md
├── experiments/
└── decisions/
```

Use Architecture Decision Records for major choices.

## 29. GIT

Use clean commits such as:

- `feat: initialize project architecture`
- `feat: add database schema`
- `feat: add telemetry`
- `feat: add workflow engine`
- `feat: add provider abstraction`
- `docs: document provider research`
- `fix: handle failed generation jobs`

Never commit secrets.

## 30. EXISTING PROJECT INSPECTION

Before implementation:

1. Inspect the entire repository.
2. Inspect README and docs.
3. Inspect source/configuration.
4. Inspect Git history.
5. Identify reusable code.
6. Do not delete existing work without explicit justification.
7. If another relevant project exists, inspect it and reuse lessons where appropriate.

## 31. OPENCODE-SPECIFIC EXECUTION

Use your strongest available capabilities.

When possible:

- delegate independent research to parallel subagents
- use specialized subagents for architecture, AI-provider research, infrastructure, database design, security, and ML strategy
- synthesize conflicting research before deciding
- verify claims with primary sources
- inspect existing code before modifying it
- use small implementation tasks
- run tests after changes
- inspect generated files
- commit coherent milestones

Do not spawn subagents just for appearance. Parallelize only genuinely independent work.

## 32. FIRST TASK — RESEARCH REPORT, NOT CODE

Your first execution must be a research/architecture pass.

Return:

1. Executive summary
2. Prompt critique
3. Current 2026 AI 3D/video landscape
4. Best candidates and alternatives
5. Whether Blender is needed
6. Best visual-production strategy
7. Free/no-card infrastructure options
8. Recommended architecture
9. Architecture diagram
10. Database ERD
11. Event schema
12. Agent architecture
13. Provider abstraction
14. Security model
15. ML data strategy
16. Cost model
17. Risks/limitations
18. Exact Phase-1 implementation plan
19. Recommended changes to this prompt
20. Questions requiring user decisions

**Do not start major implementation until the user has seen the research-based recommendations.**

## 33. NON-NEGOTIABLE PRINCIPLES

- Research before locking technology.
- Data collection begins immediately.
- Preserve raw data.
- Providers must be replaceable.
- Free tiers are dependencies of convenience, not permanent guarantees.
- Do not rely on a single AI provider.
- Do not assume Blender is mandatory.
- Prefer official APIs.
- Respect every service's Terms of Service and licenses.
- No fake accounts or free-tier abuse.
- No artificial engagement manipulation.
- Protect credentials.
- Make workflows resumable.
- Log autonomous decisions.
- Do not claim completion without testing.
- Do not silently ignore failures.
- Prefer simple working systems over premature complexity.
- Build incrementally.
- Keep the system observable and reversible.

## 34. FINAL VISION

Eventually the human should be able to specify a high-level goal such as:

> Create a series of high-quality 3D animated stories for this audience, in this language and style, at this publishing frequency.

The system should progressively handle:

research → strategy → story → production → QA → publishing → analytics → experimentation → learning → improved strategy.

The final system should behave like an **autonomous media organization implemented as software**, not like a single chatbot with a video API.

Start with research. Challenge this plan. Improve it. Then build the smallest reliable foundation that moves the project toward the final vision.
