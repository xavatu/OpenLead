# ADR-0001: Foundational Architecture Decisions

**Status:** Accepted  
**Date:** 2026-06-30

## Context

OpenLead AI must autonomously build a personal open-source contribution backlog while remaining inexpensive, auditable, safe against untrusted repository content, and easy to operate.

## Decisions

### Modular monolith

Use one Python deployable with explicit internal modules.

Reason: the MVP needs strong boundaries but not distributed deployment or service-to-service complexity.

### PostgreSQL-backed durable workflows

Use PostgreSQL for product data, prompt versions, analytics, and a `FOR UPDATE SKIP LOCKED` job queue.

Reason: one durable dependency is sufficient and restart-safe.

### PydanticAI

Use PydanticAI for typed agent output, validation, model fallback, usage limits, and GitHub MCP integration.

Do not build a custom LLM router and do not add LiteLLM in MVP.

### Free-only inference

Allow only explicitly configured free models. When all free quotas are exhausted, wait for quota reset.

No paid fallback is permitted.

### GitHub read-only

Use GitHub REST/GraphQL for polling and GitHub MCP for read-only research.

No GitHub write operation is allowed in MVP.

### Linear as task sink

Linear is the human backlog and the only external write target.

Only an OpenLead-managed section of the description may be replaced.

### Runtime-editable prompts in PostgreSQL

Contributor/project prompts are immutable, versioned database revisions with activation and rollback.

New workflow runs hot-load the active revisions. In-flight runs keep a pinned prompt bundle.

System safety and agent contracts are not runtime editable.

### No web UI in MVP

Use Linear and CLI.

### No submodules initially

Do not extract modules until a real second consumer and stable generic API exist.

## Consequences

Positive:

- low operational complexity;
- reproducible analysis;
- no restart for profile changes;
- clear write and trust boundaries;
- straightforward deployment;
- testable workflows.

Negative:

- PostgreSQL is mandatory;
- CLI prompt editing is less friendly than a web UI;
- provider quotas still require product-specific state;
- GitHub polling is not real-time;
- one deployable may require later decomposition if scale grows.

## Revisit triggers

Reconsider the architecture when:

- multiple users are required;
- several workers need centralized inference routing;
- more than one product consumes a stable internal component;
- GitHub write automation becomes a product requirement;
- custom UI becomes necessary;
- PostgreSQL job semantics are no longer sufficient.
