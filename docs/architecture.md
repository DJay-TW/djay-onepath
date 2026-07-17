# Djay OnePath Architecture Guide

## Purpose

Djay OnePath is an operational architecture discipline, not a ban on REST, frameworks, workers, or services. Its default rule is simple:

> Each user or operator intent has one primary, attributable, and verifiable responsibility path.

The rule is useful when a small team operates systems that combine a database, UI, devices, caches, background work, and remote services. In those systems, the most expensive failures are often not algorithms; they are duplicated integration paths, unclear truth ownership, and a UI that silently becomes a second backend.

## A short path is logical, not physical

OnePath does not require one file, process, or endpoint. A path may correctly include authentication, validation, a queue, a device adapter, persistent work state, and an event stream.

It is still a short path if a maintainer can quickly identify:

```text
caller -> request/command -> owner -> authoritative source or side effect -> evidence
```

It is a bad path when a second UI, cache, endpoint, worker, or fallback silently owns part of the same responsibility.

## Frontend default

Start with semantic HTML, modern CSS, and native ES modules.

- Use server-rendered pages or fragments for forms, CRUD, and independently refreshed panels.
- Use a small local render function for a one-region interaction.
- Add a signal or store when one client-side state genuinely drives several regions or repeated update paths.
- Introduce a framework when an independently maintained, multi-view surface can no longer keep state ownership clear with the smaller approach.

Native code is a default, not a virtue contest. A framework is justified when it removes real state and lifecycle complexity.

## The request surface

The internal request surface has two semantic modes.

### Query: a bounded read language

A query is a client declaration of required data shape, not a request to execute a database command.

```json
{
  "kind": "query",
  "model": "sales.summary",
  "select": ["business_date", "order_count", "gross_amount", "net_amount"],
  "filter": {
    "business_date": { "eq": "2026-07-18" }
  }
}
```

The server owns a deny-by-default logical schema. It decides which models, fields, filters, sort keys, aggregates, and traversals are valid for the authenticated principal. It then compiles a trusted logical request into source-specific physical plans.

Do not expose raw SQL, schema reflection, ORM properties, arbitrary function calls, arbitrary URLs, or unbounded graph traversal.

Sensitive fields need separate permissions for display, filtering, sorting, aggregation, and relationship traversal. New storage fields must remain unavailable until deliberately added to the logical schema.

## Query cost and compatibility

Every query needs enforced limits for depth, field count, page size, scan size, aggregate cost, remote fan-out, CPU, and timeout. Use opaque cursors rather than deep offset pagination when data can grow large.

Record a safe query fingerprint, policy version, schema version, plan summary, cost summary, source state, row count, latency, and trace ID. Never log raw sensitive inputs merely to make query debugging convenient.

Treat a semantic field rename, type change, relationship-cardinality change, or policy change as a compatibility event. Version and deprecate it deliberately.

## Command: the only business write path

Commands are explicit and allowlisted.

```json
{
  "kind": "command",
  "op": "cashbox.reconcile",
  "input": {
    "business_date": "2026-07-18"
  },
  "idempotency_key": "client-generated-key"
}
```

A command must define authorization, validation, preconditions, idempotency, concurrency/version behavior, transaction or durable handoff boundary, audit data, and retry semantics.

If it starts background work or sends a device command, it must not claim completion early. Return an accepted operation and expose a durable, observable path to eventual completion, failure, timeout, cancellation, or recovery.

## Cross-source reads

Generic queries operate within one truth domain and consistency zone. A database read can have a database snapshot; a database, a kiosk, and a remote device do not automatically share one atomic instant.

For approved cross-source views, use a named read model such as `dashboard.snapshot`. It owns composition, but each source retains ownership of its truth.

Return enough metadata to make the consistency model visible:

```text
generatedAt: when the composed response was produced
observedAt: when each source was observed
sourceState: fresh | stale | unavailable
completeness: complete | partial | truncated
snapshotId: identifier for one composition attempt
```

Never silently mix stale fallbacks into a response that appears current and complete.

## When additional structure is justified

KISS rejects accidental complexity, not necessary engineering.

A worker, service, cache, adapter, framework, or new protocol boundary is justified when it has a clear owner and at least one of these reasons:

- The work must survive process restarts, wait, retry, or run for a long time.
- It needs an independent security, availability, or deployment boundary.
- It speaks a materially different external protocol.
- It has a genuinely separate domain owner.

Do not add it merely because an abstraction looks tidier or because a framework is fashionable.

## Review checklist

Before implementing a non-trivial change, answer:

```text
What operator intent is being served?
Which domain owns the truth?
Is this a query, a command, or a named cross-source read model?
Can the existing responsibility path express it safely?
What path is being removed or explicitly excluded?
What are the freshness and partial-failure semantics?
How will the true result be verified?
```

If these answers are vague, the system boundary is not ready for implementation.
