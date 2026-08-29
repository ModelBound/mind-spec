# .mind/ Specification

**Version:** 0.1
**Status:** Draft
**License:** Apache 2.0

> Markdown is the runtime.

## 1. What is `.mind/`?

`.mind/` is a version-controlled folder where an agent keeps everything it knows.

No databases. No vector stores. No custom SDKs. Just plain markdown files in a directory. The LLM reads them as instructions, writes them as memory, and uses git for history, provenance, and rollback.

## 2. Folder layout

A minimal `.mind/` directory looks like this:

```
.mind/
  INDEX.md          # routing table: what the agent should read and when
  self.md           # identity, role, tone, constraints
  memory/           # learned facts, observations, decisions
    users/
      alice.md
      bob.md
    projects/
      website-redesign.md
  skills/           # reusable capabilities (often mirrors SKILL.md)
    write-tests.md
    review-code.md
  context/          # documents, APIs, references the agent may need
    api-reference.md
    runbook.md
  diff/             # proposed writes awaiting review (optional)
```

Only `INDEX.md` is required. All other directories are conventions — an agent MUST NOT fail if a directory is missing.

## 3. `INDEX.md`

`INDEX.md` is the agent's entry point. It tells the agent which files matter, in what order, and how to route new inputs.

```markdown
---
version: 0.1
agent: coding-assistant
---

# Index

## Always read first
- [self.md](self.md)
- [context/api-reference.md](context/api-reference.md)

## Route by intent
- ask: "help me write code" → [skills/write-tests.md](skills/write-tests.md)
- ask: "review this PR" → [skills/review-code.md](skills/review-code.md)
- ask: "who is Alice?" → [memory/users/alice.md](memory/users/alice.md)
```

An implementation MAY parse the "Route by intent" section as a lookup table; it MUST also treat the whole file as prose that an LLM can reason about.

## 4. File format

Every file in `.mind/` is markdown. Files MAY include YAML frontmatter for metadata:

```markdown
---
type: memory
confidence: 0.92
source: user-statement
created_at: 2026-07-07
---

# Alice prefers TypeScript

Alice writes TypeScript for frontend work and prefers explicit return types on public functions.
```

### 4.1 Reserved frontmatter keys

| Key | Meaning |
|-----|---------|
| `type` | One of `skill`, `rule`, `memory`, `context`, `system-prompt`, `corpus-document` |
| `confidence` | Float 0.0–1.0 trust score for memories |
| `source` | How this knowledge was acquired (`user-statement`, `observation`, `import:<url>`) |
| `created_at` | ISO 8601 date |
| `reviewed_at` | Last human review date |
| `replaces` | Path of a previous file this supersedes |
| `trust` | `human-reviewed` freezes confidence at 1.0 until explicitly changed |
| `review` | Skill review lifecycle metadata (see §12) |

Unknown frontmatter keys MUST be preserved and passed through by tooling.

## 5. The write protocol: mind-diff

An agent does not silently overwrite memory. It proposes a change as a markdown diff file under `.mind/diff/`:

```
.mind/diff/2026-07-07-001-add-alice-preference.md
```

Diff filename convention: `YYYY-MM-DD-NNN-<slug>.md`.

A diff file contains:

```markdown
---
type: proposed-write
target: memory/users/alice.md
confidence: 0.91
reason: Alice said this twice in the same thread.
---

# Proposed change

Add to memory/users/alice.md:

> Alice prefers TypeScript and explicit return types on public functions.

## Alternative

Keep the existing file unchanged if this conflicts with prior knowledge.
```

A human or trusted CI step reviews the diff and either:

- **Accepts** it — apply to `target`, delete the diff, commit both changes together.
- **Rejects** it — delete the diff, commit the rejection.

Git history is the audit log.

## 6. Trust and garbage collection

- Every memory has a `confidence` score. Values below 0.5 SHOULD be tagged for review.
- `mind gc` removes unreferenced diff files older than 30 days and merges duplicate memories.
- `mind trust <path>` sets `trust: human-reviewed` and freezes `confidence` at 1.0.

## 7. Compatibility

`.mind/` composes with existing formats:

- **`SKILL.md`** files can live in `.mind/skills/`. Frontmatter is compatible.
- **Cursor rules** can be exported as `.mind/rules/*.md`.
- **`AGENTS.md`** content maps to `.mind/self.md`.
- **System prompts** can be stored as `.mind/self.md` or a dedicated file.
- **Corpus documents** and RAG sources MAY be symlinked or copied into `.mind/context/`.

## 8. Versioning

The spec version lives in `INDEX.md` frontmatter (`version: 0.2`). Minor versions are additive. Major versions require an RFC filed as an issue on `ModelBound/mind-spec`.

## 9. Security considerations

`.mind/` files are executable-adjacent: an LLM will treat their contents as instructions. Implementations SHOULD:

- Refuse to load `.mind/` folders outside the current project root.
- Treat content fetched from URLs (`source: import:<url>`) as untrusted until reviewed.
- Never commit secrets to `.mind/` — use `.env` and reference by name only.

## 10. Scope constraints (anti-slop)

Every skill in `.mind/skills/` SHOULD include explicit task-scope limits so agents stop before exceeding a single context window of work.

### 10.1 Default limits

Override repo-wide defaults in `.modelbound/task-budgets.json`:

```json
{
  "files": { "max": 5 },
  "loc": { "max": 250 },
  "features": { "max": 1 }
}
```

### 10.2 Required block

New skills created by reference tooling include this block by default:

```markdown
## Scope Constraints (hard limits — split task if exceeded)

- **Max files per task:** 5
- **Max lines of code changed per task:** 250
- **Max features per task:** 1

If any of these would be exceeded, STOP and produce a split plan instead of writing code:

<task-split>
  <reason>Why this exceeds scope</reason>
  <subtasks>
    <task name="..." files="..." exit-criteria="..." />
  </subtasks>
</task-split>
```

When an agent would exceed any limit, it MUST emit `<task-split>` and MUST NOT write code until the human approves a smaller subtask.

### 10.3 Trust scanner

The offline trust scanner (version **h5**) penalizes skills missing scope limits, unbounded wording, unapproved dependency installs, and unbounded refactor instructions. See `ModelBound/mind-cli` and `ModelBound/modelbound-mcp-server`.

## 11. Skill review lifecycle

Skills support a local-first review state stored in YAML frontmatter:

```yaml
review:
  state: draft | pending_review | approved | rejected
  reviewed_by: alice
  reviewed_at: 2026-08-29T12:00:00.000Z
  approved_hash: <sha256 of body at approval>
  approved_trust: 92
  scanner_version: h5
  notes: optional reviewer notes
```

Rules:

- **draft** — default for new or edited skills.
- **pending_review** — author requested review (`mind review request <path>`).
- **approved** — reviewer signed off; `approved_hash` captures the body snapshot.
- **rejected** — reviewer declined with optional `notes`.

If an approved skill's body changes, effective state becomes **draft** (hash mismatch). CI SHOULD run `mind review gate <path>` to block merges of unapproved skills.

### 11.1 Served payloads

When a skill is loaded via MCP or CLI, implementations MUST include:

| Field | Meaning |
|-------|---------|
| `version` | Skill version from frontmatter |
| `trust_score` | 0–100 heuristic score |
| `scanner_version` | Trust scanner version (e.g. `h5`) |
| `review_state` | Effective review state |
| `review_meta` | Reviewer, timestamps, approved hash/trust |

### 11.2 Confidence history

Each trust/pipeline run MAY append to `.modelbound/skill-history/<skill-slug>.jsonl`:

```json
{"ts":"2026-08-29T12:00:00.000Z","trust":88,"tests_passed":3,"tests_total":3,"scanner_version":"h5"}
```

`mind status` and `mind skill-trust` report pass rate and trust trend (↑/↓/→) versus the previous run.

## 12. Reference implementations

- [`ModelBound/mind-cli`](https://github.com/ModelBound/mind-cli) — command-line tools.
- [`ModelBound/mind-mcp`](https://github.com/ModelBound/mind-mcp) — MCP server exposing `mind_read`, `mind_route`, `mind_propose_write`, `mind_recall`.
- [`ModelBound/mind-examples`](https://github.com/ModelBound/mind-examples) — starter `.mind/` folders.

## 13. Changelog

- **0.2** (2026-08-29) — Scope constraints, `<task-split>` stop condition, skill review lifecycle, trust scanner h5, confidence history.
- **0.1** (2026-07-07) — Initial public draft.
