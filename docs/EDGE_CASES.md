# Edge-Case and Failure Policies

Defaults are configurable unless they enforce safety, write boundaries, or free-only inference.

---

## 1. GitHub issue lifecycle

| Situation | Default behavior |
|---|---|
| Issue has an assignee | Reject automatically. |
| Comment says someone is working on it | Do not auto-publish; send to review. |
| Claim is older than configured expiry | Ignore only when expiry is enabled. |
| Active competing PR exists | Reject. |
| Related PR lacks closing keywords | Treat as competing when evidence links it to the same work. |
| Old issue has recent activity | Eligible. |
| Issue has no body | Evaluate only when comments provide sufficient context; otherwise review/reject. |
| Issue has very many comments | Select bounded relevant comments and retain provenance. |
| Issue is a tracking/umbrella issue | Reject or review unless a concrete subtask is identifiable. |
| Issue is created by a bot | Apply normal rules; bot authorship alone is not rejection. |
| Issue closes during research | Stop publication and record `Upstream Closed`. |
| Issue closes after publication | Move Linear task to `Upstream Closed`; never delete it. |
| Issue reopens | Create warning and optionally re-evaluate. |
| Issue is deleted or inaccessible | Mark upstream unavailable and preserve history. |
| Repository-specific claiming process exists | Repository policy overrides generic claim expiry. |

---

## 2. Repository lifecycle

| Situation | Default behavior |
|---|---|
| Repository renamed/transferred | Resolve by stable GitHub repository ID and update display name. |
| Repository archived | Stop discovery; mark active tasks with warning. |
| Repository becomes private | Stop processing in public-only MVP. |
| Repository deleted | Preserve history and mark unavailable. |
| Default branch changes | Refresh repository profile. |
| History is force-pushed | Pin existing research to its old SHA; refresh for new work. |
| Monorepo | Research only relevant bounded paths. |
| Submodules/LFS | Record limitations; do not fetch unsupported content automatically. |
| Generated/vendor code dominates | Avoid using it as primary implementation evidence. |
| Tests require paid/proprietary infrastructure | Penalize feasibility or reject based on project policy. |
| Contributions are not accepted | Reject repository candidates. |
| Maintainer inactivity | Reduce acceptance-probability score; configurable hard gate. |

---

## 3. Re-evaluation

Default trigger: any new issue comment or issue-field change.

Configurable alternatives:

- maintainer comments only;
- body/label/assignee/PR changes only;
- minimum debounce interval;
- no re-evaluation after publication;
- repository-specific trigger.

After task starts:

- assessment remains frozen;
- new upstream changes create warnings;
- research history is retained;
- user decides whether to continue.

---

## 4. AI and MCP

| Situation | Default behavior |
|---|---|
| Malformed structured output | PydanticAI validation retry; then fail visibly. |
| Valid but contradictory output | Application validator rejects and retries/fails. |
| Agent invents a file | Evidence validator rejects report. |
| File changes after research | Existing report remains pinned; create stale warning. |
| MCP unavailable | Retry, then `RESEARCH_FAILED`. |
| MCP response truncated | Mark evidence incomplete and lower confidence. |
| Repeated tool loop | Stop at tool-call limit. |
| Context too large | Deterministically reduce evidence; never silently truncate critical fields. |
| Prompt injection in repository content | Treat as untrusted data; do not follow instructions. |
| Providers disagree | Use configured fallback only for failures, not majority voting; route uncertainty to review. |
| Model removed | Configuration becomes invalid until another allowed model is activated. |
| All free quotas exhausted | `WAITING_FOR_QUOTA`; no paid fallback. |
| Evidence remains insufficient | `INSUFFICIENT_EVIDENCE`; no automatic publication. |

---

## 5. Prompt management

| Situation | Default behavior |
|---|---|
| Prompt edited during active analysis | Active analysis keeps its pinned bundle. |
| Two users/processes activate concurrently | Optimistic-lock conflict; one activation fails. |
| New revision is invalid | Keep as draft; active revision is unchanged. |
| User wants previous behavior | Roll back by reactivating an old revision. |
| Prompt contains a secret | Validation warning/error; do not activate. |
| Prompt tries to enable writes or paid models | Ignore and reject activation where detectable. |
| Prompt contradicts another prompt | Warn; allow activation only with explicit confirmation. |
| Database is unavailable | Continue no analysis; do not fall back to stale unknown prompt state. |
| Active revision is deleted | Deletion is prohibited; revisions are archived only. |

---

## 6. Linear

| Situation | Default behavior |
|---|---|
| Task already exists | Reuse it via source marker/idempotency record. |
| API creation succeeded but response was lost | Search by source marker before retrying creation. |
| User edits title/description | Preserve user content; update managed block only. |
| User deletes task | Create tombstone; never recreate automatically. |
| User moves task/project | Preserve movement unless configuration requires a warning. |
| Token is revoked | Stop writes and surface configuration failure. |
| Rate limit | Retry with backoff. |
| Status mapping missing | Fail publication/config validation. |
| User PR is detected | Request `In Progress`. |
| PR closes without merge | Add warning; do not automatically drop task. |
| PR merges | Record outcome; status transition to `Done` is configurable/manual-safe. |

---

## 7. Durable jobs

| Situation | Default behavior |
|---|---|
| Worker crashes | Lease expiry returns job to pending. |
| Duplicate job is enqueued | Idempotency key prevents duplicate active work. |
| Job repeatedly fails | Move to `DEAD` after configured attempts. |
| Quota wait spans restart | Persist `run_after` and resume later. |
| Cursor update fails | Entire scan transaction rolls back. |
| Partial repository scan | Do not advance cursor beyond successfully persisted data. |

---

## 8. Planning and backlog

OpenLead builds the backlog but does not enforce concurrency.

Labels may identify:

- `plan:main`;
- `plan:fallback`;
- `plan:research`.

The user controls:

- concurrent active tasks;
- repository mix;
- final priorities;
- whether a low-score task is started.

Research time counts toward actual work hours.
