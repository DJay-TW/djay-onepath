---
name: djay-onepath
description: Apply Djay OnePath v0.6, a KISS-first, invariant-first architecture guard for non-trivial UI, API, state-flow, data model, migration, integration, or service-boundary work. Use when choosing a frontend pattern or framework, adding or changing an API/request, composing dashboard data, adding a table or migration, introducing a worker/service, or reviewing a design for needless layers, endpoints, duplicated sources of truth, stored duplicates, unscoped tenant data, shims without removal conditions, or invalid lifecycle states.
---

# Djay OnePath v0.6

Apply this rule: **one user or operator intent should have one short, clear, verifiable responsibility path.**

Make invalid states unrepresentable: prefer a structure that prevents an invalid state over a self-check, watchdog, repair job, wrapper, or cleanup process that detects it later.

Use it to reduce accidental complexity, not to force every system into one file, one process, or one HTTP endpoint. Rules guide design and review; they do not replace enforcement. Back every important rule with a test, constraint, or type.

## First pass

Before proposing or changing an implementation, state compactly:

1. **Intent** — what the user or operator is trying to achieve.
2. **Truth owner** — the authoritative source for the required state.
3. **Path** — UI/event -> request or command -> domain handler -> truth owner -> observable result.
4. **Boundary** — what is deliberately not being added: route, framework, worker, cache, adapter, or sync channel.
5. **Proof** — the smallest static, browser, API, or live verification that can establish the result.

For live state, deployments, or host diagnostics, verify on the real host before claiming a result and get explicit approval before any release or destructive step. In Codex environments, invoke `$host-verification-gate`; elsewhere follow the applicable repository or organization instruction. For mature-repo work, follow the repository's existing conventions and instruction files (AGENTS.md, CLAUDE.md); in Codex, use `$agent-drift-guard` when its triggers apply.

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
- For internal operational surfaces, use one established request surface with explicit, allowlisted operations. Do not add a per-view, per-panel, or per-button HTTP route when that surface can express the intent.
- Split that surface by safety boundary: reads are queries, mutations and other side effects are explicit commands.
- Give every query and command validated input, authorization, stable errors, version metadata, request tracing, and appropriate depth, row, cost, and timeout limits.
- Keep health checks, event streams, uploads, and static/HTML responses separate only when their wire behavior genuinely differs.
- Never turn the request surface into arbitrary code invocation, raw SQL, client-chosen function names, an unbounded query language, or a hidden generic dispatcher.

### Reads: named read models first

- Start with **named read models**: one server-owned query function per view or consistent snapshot, returning exactly the fields that view needs. This keeps ownership obvious and needs no query compiler.
- Introduce a generic, schema-bound declarative query only when many similar reads have accumulated and named models are clearly duplicating each other. A query compiler with one caller is the over-engineering this guard exists to prevent.
- If a declarative query is introduced, keep its schema server-owned and deny by default: map stable domain fields to trusted plans, grant display, filter, sort, aggregate and relationship traversal separately, record each field's owner and sensitivity, and never reflect tables or newly migrated fields automatically.
- Keep a read inside one truth domain and consistency zone. An approved cross-source read is a named read model that returns per-source `observedAt` and state, labeled complete, partial, stale, unavailable, or truncated.

### Command boundaries

- Make an explicit command the only path for a business state change. Require authorization, validation, preconditions, idempotency, concurrency/version rules, audit, and clear retry semantics.
- Encode lifecycle invariants in the owning structure: distinct identities, storage constraints, schemas, types, or explicit state transitions. Do not leave completed, cancelled, archived, or failed data in an active identity that every consumer must remember to ignore.
- Centralize each lifecycle transition behind one command, handler, transaction, or state machine. Reject illegal transitions at that boundary; self-checks are for verification and alerting, never the mechanism that makes an ambiguous model safe.
- For durable or asynchronous work, report accepted versus completed truthfully. Give the background path one persistent owner, timeout, retry policy, observable status, and a recovery path; do not fake synchronous completion.
- Money and credit movements are append-only ledgers; balances are derived, never edited in place.

### Data ownership and storage

- **One truth, stored once.** Do not copy unchanging or derivable content into every record (for example a fixed prompt saved again with every turn). Store it once and reference it by content hash or id.
- Store snapshots so that unchanged parts are shared (per-component, content-addressed) rather than rewriting the whole state each time.
- Give diagnostic and audit payloads an explicit retention rule (keep compressed, summarize after N days, or keep only recent) decided by the product owner.
- Put budgets in tests: bytes written per user action and provider cost per request have ceilings, and exceeding them fails the build.

### Multi-tenant isolation

- Tenant isolation is a structural invariant, not a convention. Every user-owned row carries its owner; enforce it in the database where possible (for example PostgreSQL Row-Level Security) instead of relying on every query to remember an owner filter.
- Admin or impersonation views are explicit, audited capabilities, never an implicit bypass.

### Contracts, versions, and migrations

- Every compatibility shim or version-gated branch names its owner and its removal condition (version, date, or data state). A retired contract without a removal plan is debt, not safety.
- Tests guard current contracts. When a contract retires, retire its tests with it instead of keeping them alive through shims.
- Use expand/contract migrations: first add what both old and new code can read, deploy, confirm the old version is gone, then remove the old shape. This is what makes zero-downtime releases possible.
- Destructive migrations (drop, delete, value-rewriting update) need an approved plan and a verified backup.

### KISS boundaries

- Do not add a framework, service, worker, cache, registry, factory, adapter layer, compatibility shim, or dependency unless it removes more real complexity than it creates now.
- Prefer hard guardrails—database constraints, unique or foreign keys, schema validation, narrowed types, explicit ownership boundaries—over scattered status checks or repair logic.
- Prefer one explicit domain handler over a general-purpose abstraction that has only one caller.
- Keep each truth owner and side effect identifiable. Do not let the frontend become a second integration or reconciliation layer.
- Preserve necessary security, validation, idempotency, accessibility, error handling, and rollback behavior; KISS never means removing a real safety boundary.
- Define a short path by logical traceability, not by the fewest physical hops. Add a worker, service, cache, or adapter only for a durable/long-running operation, a distinct security or availability boundary, an independent deployment need, or a genuinely separate domain owner.

## Change gate

Before adding a route, process, framework, table, or integration boundary, answer:

```text
Can an existing path express this same intent safely?
If not, what exact responsibility does the new path own?
Which duplicated path will disappear or remain explicitly excluded?
What is the single truth owner, and is anything stored twice?
Which structure (constraint, type, transition, tenant rule) makes the invalid state impossible?
If this adds a shim or version branch, what is its removal condition?
Does every state change use an explicit command with a truthful completion model?
How will a maintainer verify the resulting truth and side effect?
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
- invariants and tenant scope:
- storage and cost impact:
- removal conditions for any shim:
- avoided complexity:
- verification:
```

When reviewing, flag complexity only when it obscures ownership, duplicates a path or a stored truth, creates an unnecessary protocol surface, leaves a shim without a removal condition, or makes verification harder. Do not demand architecture paperwork for isolated copy, CSS, or one-function fixes.
