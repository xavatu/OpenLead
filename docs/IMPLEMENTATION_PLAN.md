# MVP Implementation Plan

The implementation is intentionally vertical. Each milestone ends with a runnable, testable system slice.

---

## Milestone 0 — Repository and engineering baseline

Deliver:

- `uv` Python project;
- Python 3.12;
- Ruff, mypy/pyright, pytest;
- Dockerfile;
- Docker Compose with PostgreSQL;
- Alembic;
- structured logging;
- settings and `.env.example`;
- CI.

Exit criteria:

- `docker compose up -d` starts PostgreSQL and worker container;
- migrations run;
- unit tests and lint pass.

---

## Milestone 1 — Runtime prompt management

Deliver:

- prompt tables;
- immutable revisions;
- activation history;
- prompt bundles;
- seed defaults;
- CLI list/show/draft/validate/activate/rollback/diff;
- runtime loading without restart.

Exit criteria:

- activate a revised contributor profile;
- next workflow sees new version;
- in-flight workflow retains old bundle;
- rollback works;
- all runs record prompt-bundle ID.

---

## Milestone 2 — GitHub discovery

Deliver:

- repository registry;
- GitHub read-only client;
- issue/comment normalization;
- incremental polling;
- cursors, overlap, ETag;
- snapshots;
- material-change detection;
- durable discovery jobs.

Exit criteria:

- repeated scans are idempotent;
- new comment creates one new snapshot/job;
- cursor survives restart;
- pull-request objects are excluded.

---

## Milestone 3 — Deterministic eligibility

Deliver:

- assignee rule;
- excluded labels;
- closed/archive rules;
- claim detection;
- configurable claim expiry;
- competing-PR detection;
- reason codes and tests.

Exit criteria:

- default cases match edge-case policy;
- no LLM call occurs for hard rejections;
- uncertain claim routes to review.

---

## Milestone 4 — PydanticAI evaluation

Deliver:

- inference configuration;
- free-only guard;
- PydanticAI model adapters;
- `FallbackModel`;
- `UsageLimits`;
- `EvaluationAgent`;
- typed output and validators;
- quota/cooldown persistence.

Exit criteria:

- assessment is immutable and references snapshot/prompt bundle;
- malformed output is handled;
- all-free-provider exhaustion produces `WAITING_FOR_QUOTA`;
- no unknown/paid model is callable.

---

## Milestone 5 — Repository profiling and MCP research

Deliver:

- GitHub MCP process;
- read-only tool allowlist;
- repository profile;
- evidence model;
- `ResearchAgent`;
- file/evidence validation;
- research limits.

Exit criteria:

- report cites real files/evidence;
- invented file fails validation;
- prompt injection fixture is ignored;
- MCP failure is retryable and visible.

---

## Milestone 6 — Linear publication

Deliver:

- Linear GraphQL client;
- configured state validation;
- managed Markdown block;
- labels;
- source marker;
- idempotency;
- deletion tombstones.

Exit criteria:

- high-score task is created once;
- retry after lost response does not duplicate;
- user text is preserved;
- deleted task is not recreated.

---

## Milestone 7 — Reconciliation and contribution outcomes

Deliver:

- upstream issue reconciliation;
- `Upstream Closed`;
- warning updates;
- user PR detection by GitHub username;
- automatic `In Progress`;
- merge outcome tracking;
- assessment freeze after start.

Exit criteria:

- closure does not delete task;
- PR detection updates status;
- post-start comments create warnings, not rewritten assessment.

---

## Milestone 8 — Analytics and reports

Deliver:

- event store;
- funnel SQL views;
- recommendation metrics;
- estimate calibration;
- learning metrics;
- prompt-version metrics;
- weekly CLI/Markdown report.

Exit criteria:

- actual hours can be recorded;
- P50/P90 coverage is calculable;
- prompt revisions can be compared;
- operational failures are visible.

---

## Milestone 9 — Deployment hardening

Deliver:

- VPS runbook;
- backups;
- health checks;
- job recovery;
- log rotation;
- GitHub Actions CI and optional one-shot workflow;
- secrets documentation.

Exit criteria:

- one-command deployment;
- restart loses no durable work;
- database backup/restore is tested;
- no GitHub write permissions are required.
