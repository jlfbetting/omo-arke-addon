# Arke — Prometheus Planning Brief for OpenCode + Oh My OpenAgent

> **Purpose of this file:** Give this entire file to **Prometheus**, the Oh My OpenAgent interview-mode strategic planner, and ask it to create the implementation plan for Arke.
>
> **This is an approved product/architecture specification, not an implementation plan.** Prometheus must ground the implementation plan in the live Arke repository and the current upstream OmO/OpenCode code before choosing concrete files, APIs, adapter seams, dependencies, or task ordering.

## Instructions to Prometheus

You are planning **Arke**, a public addon for Oh My OpenAgent (OmO), in:

- Arke repository: `https://github.com/jlfbetting/omo-arke-addon`
- Upstream OmO: `https://github.com/code-yeongyu/oh-my-openagent`

Work in **Prometheus / ulw-plan interview mode**. Do not implement product code. The final deliverable is one decision-complete OmO work plan under `.omo/plans/` that downstream OpenCode/OmO workers can execute without further interview.

### 1. Treat the architecture below as approved

The design decisions and **Final invariants** in this document are owner-approved. Do not reopen them merely because another architecture is interesting.

You may challenge an approved decision only when your live exploration establishes that it is technically impossible, conflicts with a current hard OpenCode/OmO contract, creates a serious security/correctness problem, or would make the addon fundamentally unmaintainable. If that happens:

1. show the concrete repository/API evidence;
2. explain the smallest compatible adjustment;
3. interview the user about that owner decision before planning around the change.

Otherwise preserve the approved behavior.

### 2. Explore the live codebase before planning

OmO is undergoing an active multi-harness refactor. **Do not trust file paths or internal interfaces in this design document as current implementation truth.** They are architectural hints only.

Before asking implementation questions, inspect the current state of both repositories and trace the relevant call paths.

At minimum, inspect the current equivalents of:

- upstream root `AGENTS.md` and `ROADMAP.md`;
- `packages/omo-opencode/src/AGENTS.md`;
- `packages/omo-opencode/src/agents/AGENTS.md`;
- `packages/omo-opencode/src/agents/prometheus/AGENTS.md`;
- `packages/shared-skills/skills/ulw-plan/SKILL.md`;
- `packages/omo-opencode/src/testing/create-plugin-module.ts`;
- `packages/omo-opencode/src/plugin-interface.ts`;
- `packages/omo-opencode/src/create-managers.ts`;
- `packages/omo-opencode/src/create-tools.ts`;
- `packages/omo-opencode/src/create-hooks.ts`;
- `packages/omo-opencode/src/tools/delegate-task/AGENTS.md`;
- `packages/omo-opencode/src/tools/delegate-task/tools.ts`;
- the current delegation executor/session creation/polling/continuation files;
- the current background-agent manager/lifecycle;
- the current model resolution / fallback / model-core packages;
- the current session event, retry, abort, continuation, and model-selection APIs;
- OmO config loading and plugin registration;
- OmO installer/update/uninstall behavior;
- current OpenCode plugin/SDK version and relevant APIs.

Use CodeGraph first if available, per Prometheus/ulw-plan conventions. Use `explore` for internal code mapping, `librarian` for current OpenCode/OmO external contracts where local source does not answer them, and Metis/Momus/Oracle only according to the normal planning workflow.

### 3. Current upstream facts to verify, not blindly assume

At the time this brief was prepared, current OmO `dev` documented the following:

- the OpenCode-facing plugin lives under `packages/omo-opencode/`;
- the repository is in a major multi-harness refactor, and its root instructions explicitly require reading `ROADMAP.md` before changes;
- OpenCode-connected changes require real harness QA and evidence, not only unit tests/typechecking;
- `createPluginModule()` constructs managers, tools, hooks, and the plugin interface through a dependency-oriented factory;
- the task delegation engine is centered under `packages/omo-opencode/src/tools/delegate-task/`, with `createDelegateTask()` as a main entry point;
- harness-neutral task/retry primitives are being extracted into `packages/delegate-core/`;
- Prometheus is a primary strategic planner whose planning mechanics live in the path-backed `ulw-plan` skill;
- Prometheus plans only and writes plan artifacts under `.omo/`.

Because upstream is changing quickly, verify every one of these before using it in the final plan.

### 4. Preserve the addon distribution model

Arke is **not an OmO fork**.

The implementation plan must target an independently published/installable addon from `jlfbetting/omo-arke-addon` that a user installs after vanilla OmO.

Prefer, in order:

1. an official/stable generic OmO extension seam if one exists by implementation time;
2. a narrow wrapper/decorator/adapter around supported or sufficiently stable OmO/OpenCode surfaces;
3. a tiny generic upstream OmO extension PR if necessary and maintainable.

Do not plan a normal installation that patches generated `dist/`, edits arbitrary cached package files in place, or requires users to maintain a custom OmO fork.

If current OmO makes the fully independent wrapper technically impossible without a small upstream seam, explicitly split the work into:

- the smallest generic upstream OmO change/PR;
- the independent Arke addon that consumes it;
- a temporary compatibility strategy only if safe and justified.

### 5. The plan must obey OmO's own repository workflow

If the implementation requires changes to upstream OmO, incorporate the current upstream `AGENTS.md` requirements into the plan exactly as they exist at planning time.

At the time of this brief, that includes mandatory exploration, a written plan before editing, isolated worktrees, real OpenCode QA, evidence under `.omo/evidence/`, and PR-based workflow. Verify the current rules before finalizing the plan.

For the Arke repository itself, establish equivalent engineering discipline:

- TDD/failing-first where practical;
- atomic reviewable tasks;
- exact automated verification;
- real OpenCode/OmO integration smoke tests;
- compatibility CI;
- no task whose acceptance criterion is merely “user manually tests”.

### 6. Do not invent concrete APIs before exploration

The architecture requires capabilities such as:

- delegation interception;
- Arke ownership of managed retries;
- same-session main-agent continuation with another model;
- safe termination of a failed/stuck inference;
- session/event observation;
- available-model inventory;
- provider/account/quota inference;
- durable cross-process state.

But this document deliberately does **not** dictate exact function names or hook APIs.

Your job is to inspect current OpenCode + OmO and identify the narrowest, most stable implementation surfaces. Cite exact paths/symbols/SDK methods in the final plan.

### 7. Interview only unresolved owner decisions

The product decisions in the specification are already settled.

Do not ask the user questions that repository exploration or current public documentation can answer.

Do interview the user about genuine surviving owner decisions such as:

- a required upstream OmO contribution if multiple materially different public integration strategies remain;
- adding a persistent background service/daemon if that is truly required for overnight recovery and there are meaningful lifecycle/installation tradeoffs;
- public packaging/release choices that cannot be derived from repository conventions;
- new paid dependencies/services or real budget commitments;
- security/privacy tradeoffs that alter retained data or system permissions;
- scope changes that would contradict the approved specification.

When a reversible internal implementation choice has one defensible best practice, adopt it and record it rather than offloading it to the user.

### 8. Treat true overnight recovery as a hard requirement

A key user problem is unattended work getting stuck at a rate limit overnight.

The final plan must prove what process remains alive to:

- detect a blockage;
- maintain timers/leases;
- probe recovery;
- wake waiting tasks;
- resume the same main-agent session or a replacement subagent attempt.

Do not write “persist state and resume later” without identifying the actual runtime/lifecycle mechanism that can do so after terminals, plugin instances, or OpenCode processes exit.

If the requirement cannot survive a full OpenCode process exit without a daemon/service, make that explicit and ask the owner only if there is a genuine product tradeoff.

### 9. Main-agent and subagent semantics must stay separate

Never merge these into one generic fallback routine:

- **subagent:** one Arke LogicalTask, potentially multiple worker sessions/models, intelligent handoff;
- **main agent:** same OpenCode session, model changes in place, sticky fallback until a safe boundary.

The final task graph and tests must cover both independently.

### 10. Plan the full approved scope, but stage implementation safely

The implementation plan should cover the entire approved design, not silently reduce it to an MVP.

However, organize it into dependency-ordered waves so each major subsystem becomes testable before later sophistication is layered on.

The approved rollout concept is:

1. compatibility proof/fail-open foundation;
2. shared health registry + subagent routing;
3. Main Session Guardian;
4. validation + reputation;
5. OmO guidance + model intelligence;
6. subscription economics;
7. public installer/update/uninstall + CI polish.

Change this ordering only where live dependency analysis proves a better sequence.

### 11. Required final-plan qualities

The plan must be decision-complete for downstream OmO/OpenCode workers.

For each implementation task include, according to the current `ulw-plan` format:

- exact files to create/modify;
- exact symbols/interfaces to add/change;
- dependencies on earlier tasks;
- Must-Have and Must-NOT-Have behavior;
- specific failing-first tests;
- exact commands to verify;
- real OpenCode/OmO QA where the surface requires it;
- evidence artifacts;
- compatibility/regression checks;
- commit boundary;
- recommended task executor category and skills.

Explicitly map every major requirement in this design to at least one task.

### 12. High-accuracy review is recommended

This is a large cross-cutting orchestration feature that touches session lifecycle, provider failures, concurrency, persistence, economics, packaging, and upstream compatibility.

After the plan is written, use Prometheus's current high-accuracy review workflow (Momus plus independent Oracle review, or whatever the installed current `ulw-plan` skill requires) before handoff.

### 13. Required final verification wave

The final plan must include agent-executed verification proving at least:

- vanilla OmO works with Arke disabled;
- install is transactional and idempotent;
- uninstall restores vanilla OmO;
- subagent initial routing uses the cheapest model that passes the hard floor;
- shared quota failure propagates between processes;
- quota recovery probing is single-flight;
- subagent operational failure creates a new session with a safe handoff;
- quality failure selectively escalates requirements;
- main-agent failure preserves the same OpenCode session;
- a stuck provider retry loop is actually broken before continuation;
- healthy running tools are not blindly killed;
- fallback ownership does not race vanilla OmO;
- manual main-model changes override Arke's old preference;
- sticky fallback does not bounce mid-turn when the preferred provider recovers;
- all-models-blocked work waits and later resumes;
- crash/restart behavior matches the promised durability level;
- OmO maintainer guidance changes routing where appropriate;
- malformed config / migration failure / bridge incompatibility fail open;
- logs and `doctor --report` redact secrets;
- subscription recommendations are advisory and economically reproducible;
- current OmO release and `dev` compatibility CI run.

---

# Approved Architecture Specification

Everything below this line is the owner-approved architecture that the implementation plan must satisfy.

---

# Arke — Complete Architecture and Product Design

**Repository:** `jlfbetting/omo-arke-addon`  
**Date:** 2026-08-16  
**Status:** Approved design; implementation plan not yet written  
**Target:** Oh My OpenAgent (OmO) Ultimate / OpenCode first

## 1. Executive summary

Arke is a public, independently distributed add-on for Oh My OpenAgent (OmO) that makes multi-model orchestration resilient, cost-aware, subscription-aware, and adaptive without requiring users to maintain a custom OmO fork.

Arke has two execution roles:

1. **Subagent supervisor.** When a main OmO agent delegates work to Librarian, Metis, Momus, Oracle, Explore, Sisyphus-Junior, or another worker, the main agent still chooses *which agent* should perform the task. Arke chooses *which model* should power that worker. It chooses the cheapest currently usable model that first clears a hard capability and quality floor for the specific task and agent. If the worker becomes blocked or produces a selectively validated bad result, Arke transparently creates a new worker session on another suitable model and performs an intelligent handoff.
2. **Main Session Guardian.** For primary agents such as Sisyphus, Hephaestus, Prometheus, and Atlas, Arke does **not** normally choose the initial model. User/vanilla OmO configuration remains authoritative. Arke monitors the session and intervenes only when the selected model/provider becomes blocked or otherwise unusable. Recovery preserves the **same OpenCode session**, changes only the inference model, and continues the existing work in place.

Arke also maintains one **machine-wide persistent health and quota registry** shared by every running Arke instance and every OpenCode process. A rate-limit observation made by one session can therefore prevent all other sessions from wasting requests against the same exhausted account/quota group.

Arke has a lightweight free/cheap **Reasoner** for fuzzy task classification and a deterministic **Supervisor/Router** for policy and mechanics. LLM judgment never owns locking, quota state, retry timers, purchases, or other correctness-critical control flow.

Arke tracks real model/provider performance conservatively, refreshes model/pricing knowledge, respects the installed OmO version's own model-fit guidance, and contains a **Subscription Advisor** that can determine when upgrading or downgrading a subscription is economically preferable to paying for overflow through OpenRouter, OpenCode, or token-priced APIs. All purchases remain manual.

The product must be installable after vanilla OmO with an agent-friendly installation flow inspired by OmO itself. Arke is transactional, idempotent, reversible, fail-open, and designed so upstream OmO updates normally require only compatibility validation at a narrow adapter boundary.

---

## 2. Product principles

These are hard design constraints, not suggestions.

### 2.1 Preserve OmO's role as orchestrator

The main OmO agent decides *which worker agent* should be used. Arke decides *which model* powers that chosen worker. Arke must not replace Sisyphus's decision to call Librarian with a decision to call Momus instead.

### 2.2 Quality floor before cost

Arke must never choose a cheaper model that does not clear the task's required capability/quality floor.

Routing order is conceptually:

1. technical compatibility;
2. task/agent capability floor;
3. OmO agent-model fit constraints/guidance;
4. current provider/account/quota health;
5. effective marginal cost and scarcity;
6. empirical reliability;
7. quality headroom;
8. latency/context efficiency.

Cost is considered **only after** suitability.

### 2.3 Main-agent continuity is sacred

Subagents may migrate to new sessions. Main agents must not.

- **Subagent failure:** new worker session + intelligent handoff.
- **Main-agent failure:** same OpenCode session + replacement model + continuation.

### 2.4 Machine-wide shared knowledge

All Arke processes share health, quota, recovery-probe, reputation, and economics data through one persistent machine-wide store.

### 2.5 Temporary blocks are never permanent bans

Provider/account/quota exclusions are leases/circuit-breaker states with recovery conditions and periodic probes. If the reset time is unknown, Arke keeps probing conservatively.

### 2.6 Respect OmO maintainer guidance

The installed OmO version's machine-readable requirements and documented agent-model recommendations are high-confidence evidence. Arke may deviate when availability, task requirements, economics, or empirical evidence justify doing so, but deviations must be deliberate and explainable.

### 2.7 Vanilla OmO compatibility

Arke must be independently installable after vanilla OmO. Users must not need an Arke-specific OmO fork.

When Arke is disabled or cannot safely initialize, vanilla OmO must continue working.

### 2.8 Deterministic mechanics, LLM-assisted judgment

An LLM can classify fuzzy task requirements. It must not own:

- concurrency locks;
- quota state;
- cooldowns;
- retry counters;
- billing actions;
- state-machine transitions that must be exact;
- database transactions;
- single-flight probe ownership.

### 2.9 Explainability

Every material routing, failover, escalation, quota-block, and subscription recommendation must be explainable after the fact.

### 2.10 No autonomous purchasing

Arke may recommend changing subscriptions. It must not purchase, upgrade, downgrade, or cancel a subscription in v1.

---

## 3. Goals and non-goals

### 3.1 Goals

Arke v1 shall:

- dynamically route delegated OmO workers to the cheapest capable model;
- understand connected OpenCode models and provider/account relationships;
- maintain a persistent cross-process provider/account/quota health registry;
- distinguish model-, quota-group-, account-, and provider-wide failures;
- automatically reroute subagents on operational failure;
- selectively escalate successful-but-inadequate subagent results;
- perform intelligent handoff between replacement worker sessions;
- avoid duplicate side effects across handoffs;
- monitor main-agent sessions for provider/rate-limit blockage;
- preserve the main session while switching its model;
- keep failed main sessions alive overnight instead of idling indefinitely;
- wait durably when all suitable models are blocked and resume later;
- probe blocked quota groups and restore them when capacity returns;
- use OmO maintainer guidance in model decisions;
- learn conservative model/task reliability reputations from real runs;
- model subscriptions, dynamic quotas, scarcity, and pay-as-you-go overflow;
- advise whether subscription upgrades/downgrades are economically sensible;
- provide CLI diagnostics and decision explanations;
- be independently distributable from `jlfbetting/omo-arke-addon`;
- be easy to install through an LLM-guided workflow;
- be transactional, idempotent, reversible, and fail-open;
- test compatibility continuously against upstream OmO.

### 3.2 Non-goals for v1

Arke v1 shall not:

- replace OmO's agent taxonomy or prompts;
- choose a different agent than the one OmO requested;
- autonomously purchase or cancel subscriptions;
- act as a general cloud scheduler across multiple physical machines;
- copy or vendor large parts of OmO internals;
- rewrite OmO's global model-fallback implementation for non-Arke work;
- judge every successful response with another LLM;
- treat benchmarks as exact scientific truth;
- guarantee recovery from arbitrary external side effects that cannot be observed;
- automatically change the initial model of a main agent merely to save money.

---

## 4. Terminology

### Arke Core
The shared deterministic routing/health/economics/reputation subsystem.

### Arke Reasoner
A cheap/free LLM used to classify fuzzy task requirements and, when needed, help summarize recovery context. It does not directly dispatch models.

### Arke Supervisor
The deterministic controller for delegated logical tasks and worker attempts.

### Main Session Guardian
The controller that observes primary OpenCode/OmO sessions and recovers blocked main-agent inference in place.

### Logical Task
One conceptual delegated task requested by a parent agent. It may span multiple worker attempts/sessions.

### Worker Attempt
One concrete execution of a logical task by one worker agent on one model/session.

### Preferred Model
For a main agent, the model chosen by the user or vanilla OmO. Arke preserves this preference unless the user explicitly changes it.

### Active Model
The model currently powering a main-agent session after zero or more failovers.

### Quota Group
The smallest known shared capacity pool. Several models may share one quota group; one provider may expose multiple independent quota groups.

### Account / Connection
A concrete authenticated provider connection. Provider identity and account identity are distinct.

### Health State
`healthy | degraded | blocked | half_open | unknown`.

### OmO Guidance
Version-bound model requirements, fallback chains, prompt-family compatibility, model-family recommendations, and maintainer warnings from the installed OmO revision.

---

## 5. High-level architecture

```text
                              MACHINE-WIDE
                    +-----------------------------+
                    |       Arke SQLite DB        |
                    | models / accounts / quota   |
                    | health / probes / tasks     |
                    | reputation / economics      |
                    +--------------+--------------+
                                   |
                    +--------------+--------------+
                    |          Arke Core          |
                    | Router / Health / Catalogue |
                    | Economics / Reputation      |
                    | Validation / Scheduler      |
                    +--------+------------+-------+
                             |            |
                 delegated   |            | main-session events
                 work        |            |
                             v            v
                    +----------------+ +--------------------+
                    | Arke Supervisor| | Main Guardian      |
                    +--------+-------+ +----------+---------+
                             |                    |
                             v                    v
                      OmO worker launch     same OpenCode session
                      new sessions on       replacement model only
                      replacement
```

### 5.1 Hybrid reasoning architecture

The Reasoner outputs a structured task-requirement profile. The deterministic router then performs eligibility and ranking.

```text
Delegation request
      |
      v
Arke Reasoner (cheap/free LLM)
      |
      v
Structured requirement profile
      |
      v
Deterministic candidate filter
      |
      v
Hard capability/fit/health gate
      |
      v
Cost/scarcity ordering
      |
      v
Reliability -> headroom -> latency
      |
      v
Chosen model
```

If every Reasoner model is unavailable, a conservative deterministic task classifier keeps Arke operational.

---

## 6. Distribution and relationship with OmO

### 6.1 Independent repository and package

Arke is distributed from:

`https://github.com/jlfbetting/omo-arke-addon`

It must be installable **after** the user installs normal OmO.

The Arke repository should be structured roughly as:

```text
omo-arke-addon/
├── packages/
│   ├── core/
│   ├── omo-bridge/
│   ├── cli/
│   └── installer/
├── compat/
├── docs/
├── tests/
└── .github/workflows/
```

Arke must not require users to clone and maintain a modified OmO repository.

### 6.2 Runtime integration modes

Arke shall support three conceptual compatibility modes.

#### NATIVE
A future OmO exposes a stable generic extension API. Arke plugs into that supported interface.

#### WRAPPER
No official extension API exists, but the installed OmO's public/exported plugin surfaces match a known compatible adapter. Arke loads/decorates OmO through a narrow compatibility bridge.

#### VANILLA FAIL-OPEN
Critical compatibility checks fail. The affected Arke controller is disabled and vanilla OmO behavior remains available.

Patching generated `dist/` files or arbitrary `node_modules` content is not an acceptable normal installation mechanism.

### 6.3 Desired upstream OmO extension seam

Arke should eventually propose a small, generic, non-Arke-specific extension API upstream, conceptually supporting:

- `beforeDelegation`;
- `afterDelegation`;
- `decorateDelegation`;
- session-event observation;
- safe main-session continuation/model substitution;
- ownership metadata so OmO runtime-fallback does not race Arke;
- disposal.

Arke must not depend on this PR being accepted.

### 6.4 Tiny patch/integration surface

Arke should integrate only at boundaries needed for:

1. delegated-task interception;
2. main-session event observation/control;
3. Arke ownership marker/bypass for conflicting fallback logic;
4. CLI/diagnostics registration where useful.

Arke must not modify individual OmO agents/prompts or duplicate their implementations.

---

## 7. Configuration model

### 7.1 Separate config

User-level configuration:

`~/.config/opencode/arke.jsonc`

Project override:

`<project>/.arke/arke.jsonc`

Optional explicit path for testing:

`ARKE_CONFIG=/path/to/arke.jsonc`

Precedence:

1. built-in defaults;
2. user config;
3. project config;
4. narrowly defined environment overrides.

Arke should not force its large schema into OmO's main config.

### 7.2 Minimal configuration

```jsonc
{
  "enabled": true,
  "reasoner": {
    "models": [
      "provider/free-model-a",
      "provider/free-model-b"
    ]
  },
  "subscriptions": {
    "chatgpt-personal": {
      "provider": "openai-chatgpt",
      "plan": "plus"
    },
    "kimi-personal": {
      "provider": "kimi-code",
      "plan": "moderato"
    }
  }
}
```

All other settings have conservative defaults.

### 7.3 Top-level schema

```jsonc
{
  "enabled": true,
  "reasoner": {},
  "routing": {},
  "main_agents": {},
  "subagents": {},
  "quota_groups": {},
  "subscriptions": {},
  "catalogue": {},
  "research": {},
  "reputation": {},
  "validation": {},
  "recovery": {},
  "economics": {},
  "persistence": {},
  "observability": {}
}
```

### 7.4 Manual overrides win

Manual declarations of subscription plan, account identity, quota-group membership, model disablement, explicit provider independence, and budget ceilings override automatic inference. Arke may warn when observed behavior strongly conflicts with a declaration, but must not silently rewrite it.

---

## 8. Persistent machine-wide registry

### 8.1 Storage technology

Use SQLite in WAL mode under:

```text
~/.local/state/arke/
├── arke.db
├── logs/
│   └── arke.log
└── exports/
```

### 8.2 Entity hierarchy

```text
Provider
  └── Account / Connection
        └── Quota Group
              └── Model memberships
```

Do not assume `provider == account == quota group`.

### 8.3 Broad table groups

Catalogue: `models`, `model_capabilities`, `competence_estimates`, `omo_agent_guidance`, `pricing`, `catalogue_revisions`.

Accounts/runtime: `providers`, `accounts`, `quota_groups`, `quota_memberships`, `subscriptions`, `health_state`, `provider_observations`, `probe_leases`, `guardian_leases`, `wakeups`.

Tasks: `logical_tasks`, `worker_attempts`, `task_events`, `handoffs`, `side_effects`, `validations`.

Learning: `reputation_observations`, `reputation_rollups`, `latency_statistics`.

Economics: `usage_costs`, `capacity_observations`, `billing_periods`, `counterfactuals`, `recommendations`.

### 8.4 Conversation storage

Do not duplicate full raw conversations unless absolutely necessary. OpenCode's session store remains the source of conversational truth.

### 8.5 Migrations

Use forward-only numbered, transactional migrations. On migration failure, disable Arke and keep vanilla OmO operational.

---

## 9. Provider/account/quota health model

### 9.1 Health states

`healthy | degraded | blocked | half_open | unknown`

Track provider/account/quota identifiers, blocked/reset/probe timestamps, last success/failure, failure evidence, backoff level, consecutive failures/successes, confidence, and update timestamps.

### 9.2 Failure classes

At least:

- `QUOTA_EXHAUSTED`
- `RATE_LIMITED`
- `PROVIDER_UNAVAILABLE`
- `ACCOUNT_UNAVAILABLE`
- `MODEL_UNAVAILABLE`
- `AUTH_FAILURE`
- `CONTEXT_OVERFLOW`
- `REQUEST_TIMEOUT`
- `STALLED_RETRY`
- `TOOL_INFRA_FAILURE`
- `UNKNOWN_OPERATIONAL`

Also classify scope: request, model, quota group, account, or provider.

Deterministic evidence has priority; LLM classification is a last resort.

### 9.3 Circuit breaker

```text
HEALTHY
  | quota/rate-limit failure
  v
BLOCKED
  | next_probe_at
  v
HALF_OPEN
  | success -> HEALTHY
  | failure -> BLOCKED with later probe
```

### 9.4 Recovery probing

If reset metadata is known, use it. Otherwise use configurable conservative probing approximately `5m -> 15m -> 30m -> 60m -> 60m ...`.

### 9.5 Passive vs active probes

Passive: an actual task acts as canary when probe-eligible. Active: a tiny harmless request only when pending work is blocked because all suitable candidates are unavailable.

### 9.6 Single-flight probing

Only one process probes a quota group at a time using an expiring lease.

### 9.7 Auto-inference and manual groups

Priority: explicit config, provider/account metadata, OpenCode connection identity, provider error metadata, empirical correlation, conservative fallback.

---

## 10. Model catalogue

### 10.1 Source layers

1. OpenCode connected models;
2. OmO/Models.dev factual metadata;
3. Arke static competence estimates;
4. daily research/enrichment;
5. empirical reputation;
6. user overrides.

Retain provenance and confidence.

### 10.2 Canonical model record

Include canonical identity, provider/account/quota relationships, OpenCode availability, technical capabilities, competence estimates, cost model, OmO fit/guidance, reputation, provenance, and confidence.

### 10.3 Hard technical features

Prefer deterministic metadata for context/output limits, tools, reasoning, vision, structured output, provider restrictions, and option normalization.

### 10.4 Competence taxonomy

Levels: `0 unsuitable`, `1 weak`, `2 basic`, `3 competent`, `4 strong`, `5 exceptional`.

Dimensions include reasoning, coding, debugging, architecture, instruction following, agentic tool use, long-horizon work, research, web/document synthesis, planning, critique, structured-output reliability, hallucination resistance, context efficiency, tool reliability, and latency.

### 10.5 Unknown/new models

Use confidence-aware controlled exploration; never assume new/expensive means strong.

---

## 11. OmO maintainer guidance layer

### 11.1 Installed OmO is authoritative

Fingerprint the actual installed OmO revision. Rebuild version-matched guidance immediately after a detected OmO change.

### 11.2 Guidance taxonomy

Fit: `PREFERRED | RECOMMENDED | COMPATIBLE | DISCOURAGED | INCOMPATIBLE`.

Confidence is separate: maintainer-validated, maintainer-recommended, community evidence, inferred.

### 11.3 Routing effect

Normal routing rejects incompatible fits and strongly penalizes discouraged fits. Emergency recovery may override a discouraged fit only when compatible candidates are unavailable, capability headroom is strong, required features exist, and enhanced validation is enabled.

### 11.4 Upstream monitoring

Newer upstream guidance is informational until the local OmO revision actually supports it.

---

## 12. Arke Reasoner and task requirements

The Reasoner outputs a structured task profile rather than selecting a model. It runs on a configured free/cheap fallback chain, with a conservative deterministic classifier if all Reasoner models are unavailable.

Each OmO agent has default minimum capability floors. Effective requirements are the maximum of the agent defaults and task-specific requirements.

---

## 13. Routing algorithm

### 13.1 Hard eligibility gate

Reject candidates for unavailability, explicit disablement, blocked health scope, missing features, insufficient context/output, capability below floor, incompatible OmO fit, or known worker incompatibility.

### 13.2 Effective cost

Runtime cost modes: free, local, subscription/unmetered, subscription with scarce allowance, token-priced, request-priced, unknown.

### 13.3 Scarcity

Treat scarce included subscription capacity differently from genuinely unlimited free capacity. The scarcity penalty is an ordering value, not claimed money.

### 13.4 Lexicographic ranking

After eligibility:

1. effective marginal cost;
2. empirical reliability;
3. quality headroom;
4. expected latency;
5. context efficiency.

### 13.5 Budgets

Support per-task/hour/day paid API ceilings and a separate main-agent emergency ceiling.

---

## 14. Conservative adaptive reputation

Track rolling observations by `(model, task_class)` and, when statistically useful, `(model, agent, task_class)`.

Use time decay. Reputation primarily affects reliability, quality-escalation risk, and latency; it does not silently rewrite hard declared capability or maintainer guidance.

---

## 15. Subagent logical-task lifecycle

One parent delegation creates one LogicalTask and one or more WorkerAttempts.

```text
CREATED -> CLASSIFYING -> ROUTING -> LAUNCHING -> RUNNING
RUNNING -> WAITING_ON_TOOL -> RUNNING
RUNNING -> OPERATIONAL_FAILURE -> RECOVERING -> ROUTING
RUNNING -> VALIDATING -> SUCCESS
VALIDATING -> QUALITY_FAILURE -> ESCALATING -> ROUTING
ROUTING -> WAITING_FOR_MODEL -> ROUTING
... -> CANCELLED
```

Allow a tiny same-attempt retry budget for isolated transient failures. Quota/provider/model/context/stalled-retry failures normally create a new model attempt. Do not retry the same unresolved failure scope indefinitely.

---

## 16. Intelligent handoff

Replacement workers receive a structured package containing objective, completed/remaining work, findings, artifacts, tool activity, side effects, validation state, predecessor failure, and do-not-repeat instructions.

Evidence priority:

1. deterministic tool/file/git/test evidence;
2. worker checkpoints;
3. Reasoner-based recovery summarization.

Side effects are classified as `READ_ONLY`, `IDEMPOTENT`, `REVERSIBLE_WRITE`, `NON_IDEMPOTENT_WRITE`, `EXTERNAL_SIDE_EFFECT`, or `UNKNOWN`.

For coding work, actual repository/filesystem state is authoritative over prose.

---

## 17. Selective quality validation and escalation

Run cheap deterministic validators first. Invoke an LLM judge only on concrete ambiguity/failure signals or high-risk tasks without deterministic validation.

Quality escalation raises the specific failed capability requirement rather than blindly choosing the next model. Context overflow raises context requirements, not reasoning strength.

---

## 18. Main Session Guardian

### 18.1 Initial model authority

Main-agent initial model remains user/vanilla-OmO controlled. Track `preferred_model` separately from `active_model`.

### 18.2 Observation-only normal path

Arke observes main sessions and intervenes only on justified blockage.

### 18.3 States

`IDLE`, `RUNNING`, `WAITING_FOR_TOOL`, `WAITING_FOR_USER`, `PROVIDER_RETRY`, `SUSPECTED_STALL`, `RECOVERING`, `WAITING_FOR_MODEL`, `CONTINUING`, `COMPLETED`, `CANCELLED`.

### 18.4 Heartbeats and blockage confidence

Track model/message/tool progress. Distinguish `CONFIRMED_BLOCKAGE`, `PROBABLE_BLOCKAGE`, and `UNKNOWN_STALL`. Unknown stalls get diagnostics first.

### 18.5 Same-session recovery

Hard invariant: a blocked main model does not cause a new main session.

Recovery:

1. classify failure/scope;
2. update shared health;
3. stop the failed inference/retry loop safely;
4. verify it stopped;
5. choose a suitable replacement model;
6. continue the **same OpenCode session**;
7. supply minimal recovery context;
8. record the transition.

### 18.6 Running tools

Prefer letting healthy tools finish before model replacement. If interrupted, record uncertain state/side effects.

### 18.7 Main rescue floor

Main rescue requires a stronger quality/agent-fit floor than ordinary cheap subagent routing.

### 18.8 All candidates blocked

Persist `WAITING_FOR_MODEL`; wake on known reset, successful probe, shared health update, catalogue change, or at most 30 minutes of blind waiting.

### 18.9 Sticky failover

Do not interrupt a healthy fallback when the preferred model recovers. Switch back only at a safe boundary.

### 18.10 Manual model changes win

A manual user choice becomes the new preference and overrides historical Arke state.

### 18.11 Guardian lease

Only one process may recover a given main session at a time.

---

## 19. Durable overnight behavior and crash recovery

Persist waiting/recovery state. On startup reconcile persisted state against actual OpenCode sessions and current provider health. Cancelled tasks never resurrect.

A truly durable v1 implementation may require either a small background helper/service or reconciliation when an Arke/OpenCode process is active; the implementation plan must choose the minimum mechanism that actually satisfies the promised overnight semantics.

---

## 20. Subscription Economics

### 20.1 Subscription identity

Authority: user declaration first, reliable provider metadata second, inference third.

### 20.2 Two optimization problems

**Runtime:** marginal cost + scarcity.

**Subscription planning:** future fixed subscription cost + predicted overflow + blocking/reliability.

### 20.3 Observations

Track rate limits, blocked time, reroutes, API/OpenRouter/OpenCode overflow, capacity observations, reset behavior, and workload distribution.

### 20.4 Forecasting

Predict likely quota exhaustion before it occurs and raise scarcity penalties accordingly.

### 20.5 Counterfactual portfolios

Compare whole subscription/provider portfolios, not only one-plan upgrades.

### 20.6 Recommendation thresholds

Use sustained evidence (e.g. 14 days, 20 jobs, >$5 expected saving, confidence >= 0.75 by default), stronger evidence for expensive upgrades, and hysteresis for downgrades.

### 20.7 Advisory only

Arke never purchases, upgrades, downgrades, or cancels a subscription in v1.

---

## 21. Daily catalogue/research refresh

Use a single-flight daily refresh lease.

Sequence: OpenCode inventory -> OmO/Models.dev metadata -> additions/removals -> authoritative pricing/plans -> installed OmO guidance -> optional web/MCP enrichment -> validation -> immutable catalogue revision.

Untrusted web/LLM research never directly mutates live routing state.

---

## 22. Observability

Every material decision stores a structured trace: requirements, candidates, rejection reasons, health, economics, OmO guidance, reputation, and final rationale.

User notifications are sparse. Structured logs are JSONL and pass through a redaction layer. Never log credentials, full prompts by default, private file contents, or raw sensitive tool payloads.

---

## 23. CLI

Independent commands should include:

```text
bunx omo-arke-addon install
bunx omo-arke-addon doctor
bunx omo-arke-addon status
bunx omo-arke-addon health
bunx omo-arke-addon models
bunx omo-arke-addon tasks
bunx omo-arke-addon explain <id>
bunx omo-arke-addon economics
bunx omo-arke-addon refresh
bunx omo-arke-addon events --follow
bunx omo-arke-addon update
bunx omo-arke-addon uninstall
```

`doctor --report` creates a sanitized issue-ready diagnostic artifact.

---

## 24. Installation UX

### 24.1 Prerequisite

Install vanilla OmO first.

### 24.2 Recommended human path

README prominently recommends pasting:

```text
Install and configure Arke for Oh My OpenAgent by following the instructions here:
https://raw.githubusercontent.com/jlfbetting/omo-arke-addon/refs/heads/main/docs/installation.md
```

into a capable coding agent.

**Branch note:** the GitHub repository currently reports its default branch as `master`. Before public release either rename it to `main` or update the canonical installation URL so it is valid.

### 24.3 Installation guide behavior

The installing agent must read the full guide, inspect existing OpenCode/OmO, preserve OmO config/provider credentials, ask only unresolved subscription questions, select Reasoner models from actually available models, run doctor/smoke tests, and roll back if compatibility cannot be established.

### 24.4 Transactional/idempotent installation

`PREPARE -> VALIDATE -> BACKUP -> COMMIT -> DOCTOR -> SMOKE TEST`

On failure after mutation, roll back. Running install twice is safe.

### 24.5 Uninstall/update

Uninstall restores vanilla OmO behavior without touching provider auth. Arke and OmO update independently; an OmO change forces immediate compatibility/guidance revalidation.

---

## 25. Compatibility contract

1. **Disabled equivalence:** Arke disabled behaves like the same upstream OmO revision.
2. **Fail open:** failures disable affected Arke functionality, not OmO.
3. **Reuse, do not fork:** call stable OmO primitives where possible.
4. **Fingerprint:** use versions, revisions, exports/signatures, runtime probes, and OpenCode version as needed.
5. **Compatibility levels:** `VERIFIED`, `COMPATIBLE`, `DEGRADED`, `INCOMPATIBLE`, per subsystem.
6. **Fallback ownership:** Arke-managed execution has exactly one recovery owner; vanilla OmO fallback must not race it.

---

## 26. Compatibility manifest and CI

Maintain `compat/omo-versions.json`, signatures, and adapter metadata.

CI targets current OmO release(s) and `dev`, with a scheduled nightly upstream compatibility run.

Required smoke tests include disabled equivalence, normal routing, subagent failover, same-session main failover, cross-process quota propagation, Reasoner failure, invalid config/DB fail-open, fallback-ownership races, manual model override, wait/resume, idempotent install, clean uninstall, and rollback.

---

## 27. Security and privacy

Do not duplicate OpenCode credentials. Minimize retained data. Treat external research as untrusted. Side-effect recovery must be conservative and must not claim guarantees Arke cannot make.

---

## 28. Core internal boundaries

Keep routing, Reasoner, catalogue, health, subagent supervisor, Main Guardian, reputation, economics, persistence, observability, and OmO/OpenCode adapters independently understandable and testable.

---

## 29. Suggested public package structure

```text
omo-arke-addon/
├── package.json
├── README.md
├── LICENSE
├── packages/
│   ├── core/
│   ├── omo-bridge/
│   ├── cli/
│   └── installer/
├── compat/
├── docs/
│   ├── installation.md
│   ├── configuration.md
│   ├── architecture.md
│   ├── troubleshooting.md
│   └── superpowers/specs/
├── tests/
└── .github/workflows/
```

Names may be adjusted during implementation planning to match the chosen TypeScript build conventions.

---

## 30. Public README UX

Lead with the user problem and installation:

- smart model routing for OmO subagents;
- resilient failover for main agents;
- shared quota awareness;
- cost/subscription optimization;
- respect for OmO model guidance.

Then prerequisite, “For Humans” agent-install prompt, one-command manual install, concise examples, and only then architecture details.

---

## 31. Behavioral examples

### Cheap subagent

Sisyphus delegates Librarian -> Arke determines moderate research/tool requirement -> free Model B clears floor -> premium models remain scarcer -> Librarian runs on Model B -> validation passes -> parent gets one normal result.

### ChatGPT quota exhausted

Sisyphus on GPT hits a shared ChatGPT quota -> quota group becomes BLOCKED machine-wide -> Guardian safely stops failed inference -> same session continues on suitable Kimi model -> later probe reopens ChatGPT -> GPT becomes eligible again at a safe boundary.

### Quality escalation

Momus on cheapest capable Model A completes but deterministic validation shows missed constraints -> Arke raises architecture/critique requirements -> launches new Momus session on Model B with handoff -> succeeds.

### All models blocked overnight

Task enters durable `WAITING_FOR_MODEL` -> capacity probe succeeds later -> registry wakes/re-enables routing -> task continues rather than idling indefinitely.

### Subscription advice

Observed Kimi plan exhaustion repeatedly causes paid overflow -> counterfactual shows next subscription tier costs less than expected overflow -> Arke recommends upgrading at the next billing cycle -> user decides manually.

---

## 32. Testing strategy

Unit-test deterministic components. Property/state-machine test invalid transitions, concurrency, lease expiry, duplicate events, and crash points. Integration-test fake providers/OpenCode for 429/reset metadata, 5xx, shared quota, context overflow, stuck retries, long tools, abrupt death, validation failure, abort verification, same-session continuation, and crash/restart.

Critical invariants include: at most one active worker attempt per logical task unless explicitly designed otherwise; one guardian recovery owner; one probe owner; cancelled tasks never wake; failed migration never partially enables Arke; and no below-floor model can win because it is cheap.

Real OpenCode/OmO smoke tests are required before release.

---

## 33. Rollout strategy

### Phase 0 — compatibility proof
Independent package skeleton, OmO load/decorate path, fail-open wrapper, disabled equivalence, ownership marker.

### Phase 1 — health + subagent MVP
SQLite/WAL, inventory, quota health, circuit breaker, deterministic router, Reasoner, operational failover, basic handoff, CLI/doctor.

### Phase 2 — Main Session Guardian
Events/progress, stuck-retry detection, same-session continuation, sticky switch-back, durable waits, leases.

### Phase 3 — quality/reputation
Validators, selective judge, targeted escalation, reputation decay.

### Phase 4 — model/OmO guidance intelligence
Installed-version guidance, prompt-family fit, catalogue revisions, daily enrichment.

### Phase 5 — subscription economics
Metadata, forecasts, overflow accounting, portfolio simulation, advisory recommendations.

### Phase 6 — public-install polish
LLM-facing install guide, transactional installer/update/uninstall, sanitized reports, compatibility CI, README/releases.

---

## 34. Acceptance criteria for v1

Arke v1 is public-release ready when:

1. It installs after vanilla OmO without manual OmO source edits.
2. Uninstall restores working vanilla OmO.
3. Disabled-equivalence smoke tests pass.
4. Delegated workers are routed to cheapest-capable models.
5. Shared quota failures propagate cross-process.
6. Failed subagents resume through a new session with intelligent handoff.
7. Side-effect duplication risk is recorded/mitigated.
8. Selective validation can escalate capability requirements.
9. Blocked main agents continue in the **same** OpenCode session.
10. All-models-blocked work can wait and later resume.
11. Blocked quota groups recover via resets, passive success, or single-flight probes.
12. Installed-version OmO guidance materially affects routing.
13. Manual main-model changes override old preferences.
14. Decisions are explainable.
15. Subscription/overflow economics produce evidence-based advisory recommendations.
16. Invalid config, migration failure, Reasoner outage, or bridge incompatibility leaves vanilla OmO usable.
17. CI tests release and `dev` compatibility.
18. `doctor --report` is useful and sanitized.

---

## 35. Implementation questions deferred to the implementation plan

These require inspecting the exact current OmO/OpenCode APIs/build layout and are not unresolved product behavior:

- exact wrapper/native bridge hook points;
- exact Arke ownership marker for avoiding fallback races;
- exact OpenCode event fields for retry/stall detection;
- exact same-session continuation and abort verification sequence;
- SQLite library choice;
- exact durable wakeup mechanism needed to satisfy overnight semantics;
- authoritative cost/token observation sources;
- worker checkpoint format;
- JSON schema/config migration details;
- npm package name/release flow;
- repository branch rename from `master` to `main`.

---

## 36. Final invariants

1. **Arke routes models; OmO routes agents.**
2. **Capability/quality floor precedes cost.**
3. **Subagent failover uses a new worker session plus intelligent handoff.**
4. **Main-agent failover preserves the same OpenCode session.**
5. **Main-agent initial model remains user/vanilla-OmO controlled.**
6. **Main failover is sticky until a safe boundary; manual changes always win.**
7. **All Arke instances share one machine-wide registry.**
8. **Quota blocks are temporary circuit-breaker states and are probed for recovery.**
9. **Quota scope is model/account/provider aware and manually overrideable.**
10. **Arke respects installed-version OmO model-fit guidance but remains the final recovery decision-maker.**
11. **The LLM Reasoner advises; deterministic code owns execution mechanics.**
12. **Empirical learning affects reputation conservatively and does not silently rewrite hard capabilities.**
13. **Subscription advice optimizes future total cost; runtime routing optimizes marginal cost/scarcity.**
14. **Arke never buys subscriptions in v1.**
15. **Arke is independently distributed and does not require a maintained OmO fork.**
16. **Installation is agent-friendly, transactional, idempotent, and reversible.**
17. **Arke fails open to vanilla OmO.**
18. **Every material decision is explainable and auditable without leaking secrets.**

This specification is the architectural source of truth for the first Arke implementation plan.
