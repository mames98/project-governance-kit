# Generalization notes — traceability-check-skill-spec.md

> Notes ABOUT the spec, written at PGK codification time (2026-07-11). Not part of the spec. A map for activation, not a rewrite.

## The portable pattern

- **A scheduled, read-only audit** that verifies every governance claim traces to reality. It surfaces and drafts; it never fixes directly, never writes the state board, and commits exactly one file per run (its own report).
- **The check set** generalizes as: (1) plan stages reference real backlog entries; (2) backlog entries reference the authorizing spec section; (3) declared spec statuses match implementation reality (sampled, so runtime stays bounded); (4) shipped artifacts trace upward; (5) cross-branch divergence — governance on main describing work that lives only on an unmerged branch (run on a separate, heavier cadence); (6) ratified docs on disk are actually git-tracked (the durability check).
- **Findings classification:** violation (broken trace) / drift (stale trace) / branch-divergence (recoverable via merge — distinct from violation) / ambiguity (needs judgment).
- **Two independent 1/2/3 tier scales** — phase-closure tier (§5.5: does this finding block closing a phase? solves the "runaway gate" problem where endless minor drift re-opens closure forever) and auto-remediation tier (§5.8: may a machine apply the fix?). Keeping them orthogonal is load-bearing.
- **Remediation drafting, not fixing** (§6): exact Before:/After: text + verification check, only for findings matching a **pre-ratified pattern allowlist** — new patterns require spec amendment, never auditor improvisation. Stop conditions (no pattern match, ambiguous targets, cascade, substantive content) route to the operator.
- **Carry-forward tracking** across runs — is the trace pattern stable, improving, or degrading?
- **Scheduled runs auto-commit + notify; manual runs don't** — the operator-review gate on scheduled placement was measured to be not load-bearing.

## GEO-specific — strip or replace on activation

- **Input doc table (§4):** Codex / realization plan / roadmap / backlog / CLAUDE.md PART 2 paths → your project's authoritative docs. The *shape* (authority doc, sequencing doc, inventory doc, operational-state doc) is the portable part.
- **The six §6.1 patterns** are all shaped around GEO's Codex §4.1/§4.2 registry rows. Re-derive your own allowlist from your own doc set; start empty and ratify patterns as real findings recur.
- **Discord wiring** (`DISCORD_WEBHOOK_COMMANDS`, `audit_notify.py`, emoji contract) → any notification channel; keep the post-then-exit-cleanly contract (notification is glue, not a gate).
- **Cadence** (Mac Mini cron, 06:00/12:00) → whatever runner you have. The `docs/audits/<date>-...-run<N>.md` naming + max(N)+1 numbering is portable as-is.
- **Role names:** Cowork/CC → advisor/builder. §6.7's rule ports directly: the audit emits signals; only the advisor writes the board.
- **Run-8/Run-5 examples, version history, Codex § references** — historical context; keep for understanding, drop from the generalized spec.
