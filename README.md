# Djay OnePath

**Djay OnePath** is a portable Agent Skill for KISS-first, intent-driven architecture decisions in internal operational systems. It works with both Codex and Claude Code.

> One user or operator intent should have one short, clear, and verifiable responsibility path.

It is designed to prevent accidental complexity in UI state, API design, data ownership, background work, and multi-source operational views. It does not prescribe a framework, a database, or a single HTTP endpoint.

## What it helps with

Use Djay OnePath when designing or reviewing:

- UI data flow, dashboards, forms, and shared client state.
- API requests, declarative read models, and state-changing commands.
- Cross-source views involving a database, device, kiosk, cache, or remote service.
- A proposed framework, worker, cache, adapter, service, or new dependency.
- A design that may be accumulating routes, hidden ownership, duplicate state, or unclear fallback behavior.

## Core idea

For every non-trivial feature, make these answers explicit:

```text
Intent: What is the user or operator trying to do?
Truth owner: Which domain owns the authoritative state?
Path: How does the UI/event reach that owner and return an observable result?
Boundary: Which route, layer, worker, cache, or framework is deliberately not being added?
Proof: How will the result, source freshness, and side effect be verified?
```

The objective is not to minimize file count. The objective is to eliminate unowned, untraceable, or unverifiable second paths.

OnePath also treats invalid states as a structural problem: prefer explicit lifecycle identities, constraints, schemas, and authoritative transitions over self-checks or repair processes that try to make an ambiguous model safe afterward.

## Read and write boundaries

OnePath can use one consistent request envelope, but it separates reads from state changes.

### Queries

Use a bounded, schema-owned declarative `query` for reads. A client may request only server-approved domain models, fields, filters, ordering, aggregates, and relationships. The server validates permissions and limits, then compiles the request into trusted source-specific plans.

Queries are not raw SQL, arbitrary RPC, or an unrestricted GraphQL replacement.

### Commands

Use an explicit, allowlisted `command` for every business state change. Commands require authorization, validation, preconditions, idempotency, concurrency rules, auditability, and an honest completion model.

For durable or asynchronous work, a command must distinguish `accepted` from `completed` and hand responsibility to one persistent, observable recovery path.

## Cross-source rule

A generic query stays within one truth domain and consistency zone. Do not allow arbitrary joins across databases, kiosks, devices, or remote services.

When an operational screen truly needs multiple sources, create a named read model or composed query. Its response must expose source timestamps and state, including whether the result is complete, partial, stale, unavailable, or truncated.

See the detailed [architecture guide](docs/architecture.md).

## Installation

### Codex

Install the skill with Codex's skill installer:

```text
python ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo DJay-TW/djay-onepath \
  --path skills/djay-onepath
```

On Windows, use the equivalent Python path under `%USERPROFILE%\\.codex\\skills\\.system\\skill-installer\\scripts`.

Alternatively, copy `skills/djay-onepath` into `~/.codex/skills/djay-onepath` and start a new Codex task.

### Claude Code

Copy `skills/djay-onepath` into one of Claude Code's skill locations:

```text
~/.claude/skills/djay-onepath/       Personal skill, available in every project
.claude/skills/djay-onepath/         Project skill, committed with one repository
```

Start a new Claude Code session, then invoke `/djay-onepath`. Claude Code reads the same standard `SKILL.md`; no separate plugin or duplicated skill definition is required.

## Usage

```text
Codex:       $djay-onepath design this dashboard refresh flow
Claude Code: /djay-onepath design this dashboard refresh flow
```

## Repository layout

```text
skills/djay-onepath/    Installable Codex and Claude Code skill
docs/architecture.md    English architecture guide
README.md               Project overview and installation instructions
LICENSE                 MIT license
```

## Version

Current skill version: **v0.5**.

## License

This project is released under the [MIT License](LICENSE).
