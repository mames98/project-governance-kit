<!--
PGK AUTOMATION TIER — VERBATIM EXTRACTED SECTION (GEO Machine implementation)
Source file:    docs/governance/state-board-spec.md (Operational State Board — Specification, v1.1, 2026-05-19)
Extracted:      §3.9 "Traceability audit state" — the required board section PGK Core's
                state-board template deliberately excludes; it belongs to this tier.
Source repo:    mames98/GEO-Machine, HEAD 4579313 (45793134dcaf1d1e8e243dcdab0f9ae3cc7a92c4, 2026-06-12)
Copied:         2026-07-11 (PGK automation-tier codification)
Everything below this comment block is byte-faithful to the source section.
Generalization map: ../generalize/state-board-audit-section-notes.md
-->

### 3.9 Traceability audit state
Synthesis of the most recent traceability audit run's outcome and the pending remediation pipeline. Always present (even when the last audit produced zero findings — "Last audit: [date]: zero violations" is valuable orientation).

Required content:

- **Most recent audit run** — date, run number, and outcome summary (zero violations OR N findings categorized).
- **Findings count by category** — when non-zero, break out: violation / drift / unsupported-claim / verify-only.
- **Pending remediation** — drift findings drafted per Skill v1.1 §6, with status (drafted / under operator review / committed). Empty when no remediation pipeline is active.
- **Next scheduled audit run** — date and time of the next scheduled execution.
- **Audit output reference** — pointer to the full audit document at `docs/audits/[date]-traceability-audit-runN.md`.
- **Carry-forward to next run** — items surfaced by the most recent audit that were not closed in its corresponding closeout commit (e.g., findings deferred to a future content-alignment pass per Skill v1.1 §6.3 stop conditions).

Updated by Cowork at next session start after an audit run completes, per Cowork operating discipline §9.4 trigger-based updates. The full audit report stays at `docs/audits/`; the state board surfaces the actionable summary so fresh sessions orient without opening the audit document.

