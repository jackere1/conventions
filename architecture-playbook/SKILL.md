---
name: architecture-playbook
description: Bootstrap conventions for new backend projects — gating, orchestration, auth, validation, observability, error handling. Use whenever starting a new backend project, scaffolding a monolith, designing service boundaries, or making early architectural decisions on a Node.js/Next.js/Postgres/Docker stack. Apply at project kickoff before writing the first endpoint. Trigger even when the user doesn't explicitly say "architecture" — phrases like "starting a new project," "bootstrap," "set up the backend," "scaffold," "fresh codebase," or any greenfield backend work should activate this skill.
---

# Architecture Conventions for Bootstrapping Backends

This skill codifies a set of architectural conventions for backend projects, organized by expected daily-active-user tier. Apply at project kickoff. Re-read at major scaling inflections.

## Core philosophy (applies to every tier)

1. **Gate at semantic boundaries, not implementation boundaries.** Wrap things whose concepts aren't yours (LLM providers, payment gateways, social platforms) or whose uniformity matters across the codebase (log format, error envelope, response shape). Do not wrap owned infrastructure (Postgres, Redis) generically — gate the *policy* (event contract, idempotency expectations) and leave *mechanism* free.

2. **Gates are stateless translators.** The moment a gate coordinates state or orchestrates a flow, it's not a gate — it's a service. Name it differently.

3. **Two gate categories, never mixed:**
   - **External adapter gates** — translate foreign concepts into domain vocabulary (anti-corruption layer).
   - **Internal discipline gates** — enforce cross-cutting uniformity (logging, errors, response shape, event producer/consumer contracts).

4. **Errors carry handling intent, not just type.** Centralized handler dispatches on intent (retry / user-fix / report-and-fail / log-and-swallow), not on exception class.

5. **User-count tiers below are a proxy.** The real drivers are write rate, consistency requirements, team size, and failure tolerance. **Override the tier upward** if the project involves:
   - Payments or financial transactions
   - Regulated data (health, government)
   - Real-time requirements (sub-second SLAs)
   - Multi-region from day one
   - Team larger than the tier assumes

---

## Tier 1: ~1k DAU (MVP, internal tools, early product)

Team size assumed: 1–3 engineers. Failure tolerance: hours of downtime acceptable.

**Apply:**
- Single monolith, single Postgres, single Redis, one Docker compose stack.
- RBAC at API gate via middleware. ABAC inline in business logic — do not build an abstraction yet.
- Validation: Zod/Pydantic at API gate for shape; semantic checks inline in handlers.
- External adapter gates from day one for: LLM providers, payment, social platforms, anything billable or vendor-locked. Even at 1k DAU these are the things that change. Cheap to do now, expensive to retrofit.
- Internal discipline gates from day one for: structured JSON logging with correlation ID, central error handler with intent-tagged errors, global response envelope.
- Orchestration: don't. Inline multi-step flows. If a flow has >5 steps or requires resumability, jump to Tier 2 orchestration patterns for that flow only.
- Observability: JSON logs → Loki → Grafana. Correlation ID propagated via middleware.
- Testing: integration tests on the happy path of each gate, unit tests on pure domain functions. Skip the pyramid theatre.

**Do not apply yet:**
- Process managers / sagas
- Event sourcing or event-driven messaging
- Read replicas, caching layers beyond Redis-as-cache
- Multi-tenancy abstractions (use a `tenant_id` column if needed, defer schema-per-tenant)
- Hierarchical orchestrators
- Policy engines (OPA/Cedar)

**Red flag at this tier:** building abstractions for variations that don't exist yet. If there's one LLM provider and one payment gateway, the gate is justified by *future* swap, not by current variation — and that's fine, but resist generalizing further.

---

## Tier 2: ~100k DAU (real product, paying users, growth phase)

Team size assumed: 4–15 engineers. Failure tolerance: minutes of downtime visible to users.

**Add to Tier 1:**
- Process managers for any flow with: multiple state transitions, external provider calls mid-flow, or human-in-the-loop steps. State externalized to Redis with TTL; completed flows snapshotted to Postgres for audit.
- Idempotency keys propagated through every external gate call. Generated at the orchestrator level, persisted with flow state.
- Flow versioning: every state record carries a flow-definition version. New deploys must handle in-flight flows from the previous version (drain or forward-migrate, decided per flow).
- ABAC patterns become explicit. Three placements, used deliberately:
  - Entity methods that take an actor for single-resource decisions.
  - Query-time predicates (RLS or hand-rolled `WHERE` clauses) for list/search endpoints.
  - Policy module (in-process, not OPA yet) for cross-cutting rules.
- Read replicas for read-heavy endpoints. Write path stays on primary.
- Circuit breakers on external gates. Retries with exponential backoff at the outermost gate only — inner gates surface retryability, do not retry themselves.
- Domain events introduced *carefully* — only for genuinely async, decoupled use cases (notifications, analytics, audit). Do not event-source the core domain yet.
- Testing: contract tests at each gate, scenario tests at orchestrator level, property tests on domain invariants.

**Watch for:**
- Cross-domain flows emerging (checkout-style flows that touch multiple domains). Resist normalizing hierarchical orchestrators across the codebase — apply only where the cross-domain flow exists. Accept that gate signatures become coordination points across teams.
- The orchestrator-as-god-object pattern. Split when a process manager exceeds ~300 lines or handles more than one business outcome.
- Logs-with-correlation-ID hitting its ceiling on long-running flows. If flows last hours or branch heavily, evaluate Temporal or a flow-as-first-class-entity store.

**Do not apply yet:**
- Microservices (the monolith still fits)
- Event sourcing of the core domain
- Multi-region
- Custom DSL/YAML flow engines (always a trap)

---

## Tier 3: ~1M DAU (scale phase, established product)

Team size assumed: 15–50 engineers. Failure tolerance: seconds of downtime measurable in revenue.

**Add to Tier 2:**
- Modular monolith with enforced package boundaries (explicit public/private APIs per module). Defer microservices until team coordination cost exceeds deploy coordination cost — usually past 50 engineers, not user count.
- Policy engine (OPA/Cedar) extracted if authorization rules need to be audited or owned by non-engineers. Otherwise keep in-process policy module.
- Flow observability upgrade: every state transition emits a structured event to a queryable timeline store. Correlation ID alone is insufficient.
- Multi-tenancy decision made explicitly and propagated through every layer (gate, orchestrator, domain). Most products land on shared-schema with `tenant_id` + RLS; pick deliberately, not by default.
- Read/write splitting at the gate level for hot endpoints. Caching layer (Redis or CDN) with explicit invalidation contracts — never TTL-only for user-facing data.
- Integration events distinguished from domain events distinguished from commands. Three different bus shapes, three different lifecycles, never on the same topic.
- Dedicated platform team owning the discipline gates (logging, errors, response, event contracts). Discipline gates become products with versioning and migration paths of their own.

**Watch for:**
- Gate signatures becoming political — every change requires N team approvals. This is the cost of shared gates and is correct; do not solve it by removing gates, solve it with versioned contracts.
- Eventual consistency becoming visible to users (read-your-writes failures). Solutions in priority order: route reads to primary after writes for the affected user, version tokens in responses, optimistic UI with reconciliation.
- The event-sourcing temptation. Resist unless the domain has genuine audit/replay requirements (financial, regulated). Event-driven without event-sourcing is the defensible middle ground.

---

## Universal anti-patterns (every tier)

- **DSL/YAML flow engines.** Looks elegant, becomes a programming language with worse tools than the host language. Flow definitions are code.
- **Generic infrastructure wrappers.** A `DatabaseInterface` that hides Postgres-specific features loses what made Postgres worth picking. Wrap policy, not mechanism.
- **God orchestrators.** Process managers that grow conditional branches per customer segment. Split by business outcome, not by lifecycle phase.
- **Fake uniform interfaces over heterogeneous providers.** A `SocialPlatform` interface that's the intersection of FB/Discord/Telegram capabilities is uselessly small. Share the 60% that genuinely overlaps; expose per-provider operations explicitly.
- **Catch-all exception handlers that return 500.** The handler must dispatch on intent. Generic catch sedates errors instead of strategizing about them.
- **Validation at the gate that requires loading domain state.** That's not validation, that's a domain operation pretending to be middleware. Move it inward.

---

## How to use this skill when bootstrapping

1. Estimate the realistic 12-month tier (not the today tier, not the dream tier).
2. Apply the **override clauses** in the core philosophy section. Bump the tier if any apply.
3. Read the chosen tier's "Apply" section as the starting checklist.
4. Read the next tier's "Add" section to know what's intentionally being deferred — defer deliberately, not accidentally.
5. Re-read this skill at major inflections: first paying customer, first 10x user growth, first team doubling, first cross-domain flow.

The point of tier-bound conventions is not to predict the future. It is to make the *current* decisions deliberate and the *deferred* decisions visible.