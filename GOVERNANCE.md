# Governance

`.mind` is an open specification for markdown-native agent memory. This file describes how decisions are made.

## Steward

ModelBound (https://modelbound.co) is the current steward of the `.mind` spec. Stewardship covers:

- Maintaining `mind-spec`, `mind-cli`, `mind-mcp`, and `mind-examples` under the `ModelBound` GitHub org
- Publishing the spec at [mind.modelbound.co/spec](https://mind.modelbound.co/spec)
- Running the reference registry at [mind.modelbound.co/registry](https://mind.modelbound.co/registry)
- Merging accepted RFCs and cutting spec releases

Stewardship is **not** ownership. The spec is Apache 2.0 licensed; anyone may fork.

## Changes to the spec

All breaking or semantic changes to `SPEC.md` go through an RFC:

1. Open an issue in `mind-spec` using the **RFC** template.
2. Discuss for a minimum of 7 days.
3. A steward summarizes consensus and either merges, requests changes, or rejects with rationale.
4. Accepted RFCs land as a PR bumping `SPEC.md` and the `version:` field in the frontmatter.

Editorial fixes (typos, clarifications with no semantic change) may be merged directly by a maintainer.

## Versioning

The spec uses `MAJOR.MINOR` numbers stored in the `version:` frontmatter of every `.mind/INDEX.md`.

- **MAJOR** — breaking changes to required file layout or frontmatter keys.
- **MINOR** — additive, backwards-compatible extensions.

Reference implementations (`mind-cli`, `mind-mcp`) MUST accept every MINOR release within a MAJOR line.

## Handoff

If a neutral foundation (e.g. OpenJS, LF AI & Data) or a coalition of adopters requests co-stewardship, the current steward will:

- Publish a transition proposal as an RFC
- Transfer the `ModelBound/mind-*` repos or mirror them under the new home
- Continue to ship a compliant reference implementation

The goal is to keep `.mind` neutral, forkable, and boring.
