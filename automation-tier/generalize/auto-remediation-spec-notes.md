# Generalization notes — auto-remediation-spec.md

> Notes ABOUT the spec, written at PGK codification time (2026-07-11). Not part of the spec. A map for activation, not a rewrite.

## The portable pattern

- **An autonomous consumer of audit output** that closes the gap between "audit found it" and "finding addressed" for mechanical fixes too small to justify operator attention — while preserving operator authority over everything substantive.
- **Three-tier execution:** Tier 1 auto-execute-and-commit; Tier 2 auto-execute-and-flag ("execute then flag" because awareness conditions describe blast-radius-if-wrong, not standalone risk); Tier 3 halt the ENTIRE cycle and interrupt (one Tier 3 halts everything — preserves its visibility against digest noise).
- **The allowlist as ratification gate:** only findings matching a pre-ratified mechanical-fix pattern with explicit Before:/After: text are eligible. Pattern-match failure = halt, never guess-and-commit. Never fabricate a fix for an ambiguous finding — producing nothing beats producing something invented.
- **All-or-nothing cycles:** verification failure on any edit reverts every edit applied that cycle; no partial commits. Idempotent against the same audit file.
- **Quarantine + graduation:** first 30 days commit to a date-stamped quarantine branch the operator merges daily; graduation to direct-to-main is automatic when zero reverts AND zero traceable new violations hold over a trailing 30-day window with ≥10 cycles — and re-quarantine is automatic and bidirectional (one bad cycle can't be papered over by 29 clean ones).
- **Plain-English operator digests** with a severity contract (clean / glance-recommended / operator-decision-needed / environmental-skip), and every halt carrying a recommended action — the operator ratifies or overrides, never thinks from scratch.
- **The commit message is the audit trail** — findings, patterns, files, verification, cycle metadata, quarantine status. Jargon belongs in the commit (developer surface); plain English in the digest (operator surface).
- **A JSON state file** for quarantine status / window counters; **the cycle never writes the state board** (the advisor synthesizes its commits at next session start).
- The **v1.0.1 lesson** is itself portable: a defensive pre-check (head-advance halt) that duplicated what atomic str_replace failure already detects false-halted every cycle — strip defensive armor a lower mechanism already provides.

## GEO-specific — strip or replace on activation

- **The §3.1 pattern table** mirrors GEO's skill-spec §6.1 allowlist (Codex registry-row edits). Your allowlist comes from your own audit spec; keep the mirror-table sync discipline (§10.2) if you mirror at all.
- **Discord wiring** (`discord_notify`, `DISCORD_WEBHOOK_COMMANDS`, `#commands`) and the **§8.5 reuse table** of `pipelines/shared/` helpers → your project's notification + shared-infra equivalents.
- **Cron specifics** (Mac Mini, `~/geo-machine`, `15 6,12 * * *`, 5–10 min after the audit) → your runner; keep the sequencing principle (remediation fires after the audit's push has settled; separate cron entry, not chained).
- **Paths:** `pipelines/auto_remediation/cycle.py`, `data/auto_remediation_state.json`, `data/logs/auto-remediation.log` → your layout; the thin-orchestrator-invokes-agent-only-when-judgment-needed note (§8.2) ports as-is.
- **Role naming:** "CC sub-mode" → builder sub-mode; Codex principle citations (§1.2 effectiveness, Principle 0) → your project's equivalent principles or PGK's honest-state principle.
- **GitHub URL construction, Codex §4.1 registration text, revision history** — GEO housekeeping; drop.
