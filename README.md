# OpenLead AI

OpenLead AI is an autonomous AI lead that discovers, researches, evaluates, and prepares open-source contribution tasks for a team of one.

It monitors an explicit allowlist of public GitHub repositories, evaluates new and changed issues against a configurable contributor profile, researches promising candidates through the read-only GitHub MCP Server, and maintains an actionable backlog in Linear.

## Core decisions

- Python modular monolith.
- PostgreSQL is the source of truth and durable job queue.
- PydanticAI is the agent framework.
- GitHub is read-only in the MVP.
- Linear is the only external write target.
- Hosted inference must remain free-only and fail closed.
- User/profile prompts are versioned in PostgreSQL and can be changed without restarting the service.
- No Git submodules are introduced until a genuinely reusable component with a second consumer appears.

## Documentation

- [Architecture](docs/ARCHITECTURE.md)
- [Domain model](docs/DOMAIN_MODEL.md)
- [Prompt management](docs/PROMPT_MANAGEMENT.md)
- [Analytics](docs/ANALYTICS.md)
- [Edge-case policies](docs/EDGE_CASES.md)
- [Implementation plan](docs/IMPLEMENTATION_PLAN.md)
- [Foundational ADR](docs/adr/0001-foundational-decisions.md)

## MVP deployment

The primary runtime is Docker Compose on a VPS:

```text
OpenLead worker
PostgreSQL
GitHub MCP Server
```

GitHub Actions is used for CI and may also run one-shot scheduled workflows against an external PostgreSQL database.

## Status

Architecture specification. Implementation has not started.
