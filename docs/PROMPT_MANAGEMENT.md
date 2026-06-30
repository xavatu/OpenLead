# Runtime Prompt Management

**Status:** Accepted for MVP

---

## 1. Requirement

The contributor must be able to slightly adjust prompts about themselves, their goals, and task preferences without restarting OpenLead.

The solution must also preserve reproducibility: every assessment and research report must identify the exact prompt content used.

---

## 2. Prompt separation

### 2.1 Non-editable system policy

Stored in code and deployment configuration:

- GitHub read-only boundary;
- Linear-only write boundary;
- free-only inference;
- tool allowlist;
- structured-output requirement;
- evidence requirement;
- prompt-injection handling;
- no shell/code execution;
- secret protection.

These rules cannot be changed from the database prompt editor.

### 2.2 Versioned agent templates

Stored in the codebase and versioned with releases:

- evaluation agent contract;
- research agent contract;
- output interpretation rules;
- evidence formatting;
- validation instructions.

Agent-template version is persisted with every run.

### 2.3 Runtime-editable user prompts

Stored in PostgreSQL:

| Key | Purpose |
|---|---|
| `contributor_profile` | Current skills, experience, strengths, limitations |
| `contribution_goals` | Learning and contribution goals |
| `task_preferences` | Preferred/excluded task types and size |
| `evaluation_guidance` | Personal weighting and interpretation notes |
| `research_guidance` | Research depth or special areas to inspect |
| `linear_description_preferences` | Presentation preferences for created tasks |
| `repository_override` | Optional repository-specific rules |

---

## 3. Data model

```text
PromptDefinition
- id
- key
- scope_type
- scope_id
- description

PromptRevision
- id
- prompt_definition_id
- revision_number
- content_markdown
- content_hash
- status
- validation_result
- created_by
- created_at

PromptActivation
- id
- prompt_definition_id
- prompt_revision_id
- activated_by
- activated_at
- deactivated_at

PromptBundle
- id
- bundle_hash
- revision IDs used by the run
- created_at
```

Revision content is immutable.

---

## 4. Activation semantics

1. User creates a draft revision.
2. OpenLead validates size, encoding, forbidden control constructs, and required headings when applicable.
3. Optional dry-run renders the final assembled prompt without making an external model call.
4. User activates the revision.
5. Activation atomically deactivates the previous revision.
6. New workflow runs use the new active revision.
7. Existing runs continue with their pinned `PromptBundle`.
8. Rollback activates an earlier revision without deleting history.

---

## 5. Hot reload behavior

No restart is required.

At the beginning of evaluation or research:

1. Load current active revision IDs.
2. Build or reuse a `PromptBundle` identified by its content hash.
3. Persist the bundle ID on the job/run.
4. Use the bundle for the entire operation.

Caching is optional:

- cache key: active revision IDs;
- cache invalidation: activation transaction;
- initial MVP may read from PostgreSQL on every run;
- PostgreSQL `LISTEN/NOTIFY` can be added later.

---

## 6. CLI

```bash
openlead prompts list
openlead prompts show contributor_profile
openlead prompts draft contributor_profile --editor
openlead prompts validate <revision-id>
openlead prompts render-bundle --repository owner/repo
openlead prompts activate <revision-id>
openlead prompts rollback contributor_profile <revision-id>
openlead prompts history contributor_profile
openlead prompts diff <revision-a> <revision-b>
```

`--editor` uses `$EDITOR` and stores the result as a new draft.

---

## 7. Concurrency

Prompt activation uses optimistic concurrency.

An activation request includes the currently known active revision ID. If it changed since the user began editing, activation fails with a conflict and requires review.

This prevents silently replacing another edit.

---

## 8. Prompt assembly order

```text
system safety policy
agent template
active contributor profile
active contribution goals
active task preferences
active evaluation/research guidance
optional repository override
untrusted GitHub evidence
```

Every block is delimited and labelled.

Untrusted GitHub content must never be interpolated into a system instruction.

---

## 9. Size and validation

Configurable limits:

- maximum characters per revision;
- maximum total active prompt size;
- maximum repository override size;
- UTF-8 text only;
- Markdown allowed;
- no secrets;
- no API keys;
- no arbitrary provider/model selection;
- no tool-enable directives.

Validation warnings may include:

- contradictory preferences;
- missing time budget;
- overly broad goals;
- overlap between preferred and excluded task types.

Warnings do not automatically modify content.

---

## 10. Prompt analytics

Every assessment and research event stores:

- prompt-bundle ID;
- individual revision IDs;
- agent-template version;
- model/provider;
- score/confidence/outcome.

This enables comparison between prompt versions without automatic online learning.

A prompt revision should not be judged until it has a meaningful sample size.

---

## 11. Bootstrap and backup

At first startup, default prompt revisions may be seeded from files:

```text
config/prompts/defaults/*.md
```

After seeding, PostgreSQL is authoritative.

Backup must include prompt tables together with all analysis tables to preserve reproducibility.
