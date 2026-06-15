# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A demo set of Architecture Decision Records (ADRs). There is no application code, no build, no
tests, no linter — every artifact is a Markdown ADR under `docs/ADR/`. Work in this repo means
authoring or editing ADR documents.

## File naming

`docs/ADR/ADR-NNNN-kebab-case-short-title.md`. `NNNN` is a zero-padded sequence number; the
existing files show numbers are sparse (there are gaps in the sequence) — do not assume the
next ADR is `last + 1`. When adding a new ADR, pick a number higher than every existing one
unless the user specifies otherwise.

## ADR document structure

Each ADR follows the same shape. Match it when adding or editing files:

1. **H1 title:** `# ADR-NNNN: <one-line decision statement>` (the title states the decision
   itself, not the topic — e.g. "Use IAP for…", "Rotate JWT signing keys…").
2. **Metadata block** as a bullet list directly under the title:
   - `**Status:**` — one of `Draft`, `Accepted`, `Superseded by ADR-XXXX`. Drafts also include a
     blockquote note immediately after the metadata block flagging that it is a proposal and
     listing what must be resolved before it is Accepted.
   - `**Date:**` — ISO `YYYY-MM-DD`.
   - `**Deciders:**` — named people grouped by team/role, e.g. `Platform team (A. Lindqvist,
     R. Okafor), Security review (M. Sato)`.
   - `**Related:**` (optional) — `ADR-XXXX (short topic)` references to sibling ADRs.
3. **Sections**, in this order:
   - `## Context` — why the decision is needed; constraints; what is wrong with the status quo.
   - `## Decision` (or `## Decision (proposed)` for Drafts) — what we are doing.
   - `## Options considered` — numbered list; mark the chosen one `(chosen)` and rejected ones
     with a brief reason. Drafts may omit this if alternatives are still open.
   - `## Consequences` — split into `**Positive**` and `**Negative / trade-offs**` bullet
     sub-sections. Be honest about the downsides; ADRs in this repo do not hide them.
   - `## Open questions (to resolve before Accepted)` — Drafts only.
   - `## Runbook` / `## Follow-ups` — optional, used when the decision implies concrete
     operational steps or downstream work (see ADR-0009 for an example of a full runbook with
     gcloud commands).

## Cross-referencing and lifecycle

- ADRs are immutable once Accepted: do **not** rewrite an Accepted ADR to reflect a new
  direction. Instead, write a new ADR that supersedes it, and change the old one's status to
  `Superseded by ADR-XXXX` (see ADR-0003 → ADR-0007 for the pattern).
- When an ADR depends on or scopes another, link it from `**Related:**` in the metadata block
  *and* mention the relationship inline in `## Context` or `## Decision` so a reader can see
  the boundary (see ADR-0009 deferring scoping to ADR-0010, and ADR-0011 deferring non-HTTP
  access to ADR-0009).
- Some `**Related:**` numbers may point to ADRs not present in `docs/ADR/`. That is expected
  for the demo; do not invent their contents — refer to them by number only.
- **Every ADR reference is a relative markdown link**, including in metadata (`Status:`,
  `Supersedes:`, `Related:`) and inline prose: `[ADR-NNNN](ADR-NNNN-slug.md)`. When you add or
  edit an ADR, link every mention so navigation works in both file viewers and rendered docs.
  If you reference an ADR that does not exist in `docs/ADR/`, write the bare `ADR-NNNN` without
  a link rather than fabricating a target path.

## Editing conventions

- Hard-wrap prose at roughly the column width used by the existing ADRs (~95 cols). Match the
  surrounding file rather than reformatting.
- Use fenced code blocks with a language tag (` ```bash `) for commands, as in ADR-0009's
  runbook.
- Keep the decision statement in the H1 and `## Decision` aligned; if you change one, change
  the other.
