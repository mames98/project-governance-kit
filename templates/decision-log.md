# Decision Log — [PROJECT_NAME]

The append-only record of decisions (ADRs — Architecture Decision Records). This is the *why* trail: choices get made, and without a record of the reasoning they get re-litigated months later. One file per decision, numbered sequentially, stored at `[DECISIONS_DIR]` (e.g. `docs/decisions/`).

**Conventions:**
- **Numbering:** `ADR-NNN-short-slug.md`, sequential, never reused. ADR-001 should pin the project's core premise or most load-bearing early decision.
- **Append-only:** a superseded decision gets a new ADR that names what it supersedes; the old ADR's Status line is updated to point forward. Never rewrite history.
- **Status values:** `Proposed` / `Accepted — [who] signed off [date]` / `Superseded by ADR-NNN`.
- **Cross-reference:** ADRs name the commits, documents, and other ADRs they relate to. A decision without provenance is half a decision.

## Index

| ADR | Title | Status | Date |
|---|---|---|---|
| ADR-001 | [title] | [status] | [date] |

---

## ADR template

Copy the skeleton below into a new `ADR-NNN-short-slug.md` for each decision.

```markdown
# ADR-NNN: [Decision title — what was decided, in one line]

**Status:** [Proposed | Accepted — [who] signed off [date] | Superseded by ADR-NNN]
**Relates to:** [ADR-NNN, relevant docs, open items — or "—"]

## Context

[What situation forced this decision. What was observed, what broke, what
gap existed. Ground it in verified facts — name the evidence (commits,
audits, incidents) rather than asserting from memory.]

## Decision

[What was decided, stated concretely enough that a future session can tell
whether the project still conforms to it. Include the load-bearing
specifics: names, thresholds, mechanisms, boundaries of scope.]

## Rejected alternatives

| Option | Why not |
|---|---|
| [alternative considered] | [the reason it lost — one or two sentences, honest about trade-offs] |

## Consequences

[What follows from this decision: what it enables, what it constrains,
what debt or open items it knowingly creates, what it defers.]

## Source

[Provenance: the documents, commits, audits, or conversations this
decision is grounded in. Links or paths a future session can verify.]
```
