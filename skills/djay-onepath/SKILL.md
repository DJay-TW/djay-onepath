---
name: djay-onepath
description: Apply Djay OnePath, a KISS-first, invariant-first architecture guard for non-trivial UI, API, state-flow, integration, or service-boundary work. Use when choosing a frontend pattern or framework, adding or changing an API/request, composing dashboard data, introducing a worker/service, or reviewing a design for needless layers, endpoints, duplicated sources of truth, or invalid lifecycle states.
---

# Djay OnePath v0.5

Apply this rule: **one user or operator intent should have one short, clear, verifiable responsibility path.**

Make invalid states unrepresentable: prefer a structure that prevents an invalid state over a self-check, watchdog, repair job, wrapper, or cleanup process that detects it later.

Use it to reduce accidental complexity, not to force every system into one file, one process, or one HTTP endpoint.

## First pass

Before proposing or changing an implementation, state compactly:

1. **Intent** — what the user or operator is trying to achieve.
2. **Truth owner** — the authoritative source for the required state.
3. **Path** — UI/event -> request or command -> domain handler -> truth owner -> observable result.
4. **Boundary** — what is deliberately not being added: route, framework, worker, cache, adapter, or sync channel.
5. **Proof** — the smallest static, browser, API, or live verification that can establish the result.

For live state, deployments, or host diagnostics, follow the host-verification procedure supplied by the current environment. In Codex environments, invoke `$host-verification-gate`; in Claude Code, follow the applicable repository or organization instruction. For mature-repo work, use the environment's drift guard when its triggers apply (in Codex, `$agent-drift-guard`).

## Decision rules

### UI and state

- Default to semantic HTML, modern CSS, and native ES modules.
- Use server-rendered HTML or fragments for forms, CRUD, and independently refreshable panels.
- Use a small local render function for one-region interaction. Introduce a signal/store only when one client-side state genuinely drives multiple regions or repeated update paths.
- Evaluate a framework only for a new, independently maintained, multi-view surface after native or fragment-based approaches no longer keep ownership clear.
- Do not rewrite an existing surface merely to adopt a framework.

### Requests and APIs

- Treat an API as an expression of durable user or operator intent, not a mirror of tables, buttons, or implementation files.
- Reuse an existing operation when the intent is unchanged. Extend compatible input deliberately; create a new operation only for a new semantic capability.
- For internal operational surfaces, use the established OnePath request surface and explicit, allowlisted operations as the default. Do not add a per-view, per-panel, or per-button HTTP route when that surface can express the intent. If adopting OnePath for a new internal surface, establish one request endpoint rather than a parallel family of routes.
- Split that request surface by safety boundary: use a schema-bound declarative `query` for reads and an explicit allowlisted `command` for mutations or other side effects.
- Let a query declare only allowed domain models, fields, filters, ordering, aggregates, and relationships. Compile the validated query into source-specific plans; never expose raw database structure or ask the browser to compose source calls.
- Give every query and command validated input, field-level authorization, stable errors, version metadata, request tracing, and appropriate depth, row, cost, and timeout limits.
- Prefer a coherent snapshot or composed query for data that one view needs consistently. Let the client select a safe response shape when the presentation changes without creating a new endpoint.
- Keep health checks, event streams, uploads, and static/HTML responses separate only when their wire behavior genuinely differs.
- Never turn the unified request surface into arbitrary code invocation, raw SQL, client-chosen function names, an unbounded query language, or a hidden generic dispatcher. A bounded, schema-driven declarative read query is intentional; commands remain explicit.

### Query compiler boundaries

- Keep the query schema server-owned and deny by default. Map stable domain fields to trusted source plans; never reflect tables, ORM properties, or newly migrated fields automatically.
- Grant display, filter, sort, aggregate, and relationship traversal separately. Record each field's owner, sensitivity, relationship direction/cardinality, source capability, and compatibility version.
- Limit a generic query to one truth domain and consistency zone. Do not permit arbitrary cross-source joins or aggregates.
- Model an approved cross-source read as a named read model or composed query. Return per-source `observedAt` and state, and label results as complete, partial, stale, unavailable, or truncated as applicable.
- Record a safe query fingerprint, schema/policy version, plan and cost summary, source states, row count, latency, and trace ID. Let operators revoke an exposed model or field without waiting for a client redeploy.

### Command boundaries

- Make an explicit command the only path for a business state change. Require authorization, validation, preconditions, idempotency, concurrency/version rules, audit, and clear retry semantics.
- Encode lifecycle invariants in the owning structure: distinct identities, storage constraints, schemas, types, or explicit state transitions. Do not leave completed, cancelled, archived, or failed data in an active identity that every consumer must remember to ignore.
- Centralize each lifecycle transition behind one command, handler, API, transaction, or state machine. Reject illegal transitions at that boundary; use self-checks for verification and alerting, never as the normal mechanism that makes an ambiguous model safe.
- For durable or asynchronous work, report accepted versus completed truthfully. Give the background path one persistent owner, timeout, retry policy, observable status, and a recovery path; do not fake synchronous completion.

### KISS boundaries

- Do not add a framework, service, worker, cache, registry, factory, adapter layer, compatibility shim, or dependency unless it removes more real complexity than it creates now.
- Prefer hard guardrails—database constraints, unique or foreign keys, schema validation, narrowed types, and explicit ownership boundaries—over scattered status checks or repair logic. If a legacy shim is unavoidable, label its owner and removal condition.
- Prefer one explicit domain handler over a general-purpose abstraction that has only one caller.
- Keep each truth owner and side effect identifiable. Do not let the frontend become a second integration or reconciliation layer.
- Preserve necessary security, validation, idempotency, accessibility, error handling, and rollback behavior; KISS never means removing a real safety boundary.
- Define a short path by logical traceability, not by the fewest physical hops. Add a worker, service, cache, or adapter only for a durable/long-running operation, a distinct security or availability boundary, an independent deployment need, or a genuinely separate domain owner.

## Change gate

Before adding a route, process, framework, or integration boundary, answer:

```text
Can an existing path express this same intent safely?
If not, what exact responsibility does the new path own?
Which duplicated path will disappear or remain explicitly excluded?
How will a maintainer verify the resulting truth and side effect?
Does a generic query remain inside one truth domain and consistency zone?
If it crosses sources, which named read model owns composition and declares partial/stale semantics?
Does every state change use an explicit command with a truthful completion model?
```

If the answer is unclear, simplify the design or ask for the missing product decision before implementing.

## Output and review

For non-trivial work, include a short OnePath summary in the handoff:

```text
OnePath:
- intent:
- truth owner:
- request/handler path:
- query/command boundary:
- consistency and source state:
- avoided complexity:
- verification:
```

When reviewing, flag complexity only when it obscures ownership, duplicates a path, creates an unnecessary protocol surface, or makes verification harder. Do not demand architecture paperwork for isolated copy, CSS, or one-function fixes.
