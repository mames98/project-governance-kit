# Generalization notes — state-board-audit-section.md

> Notes ABOUT the extracted section, written at PGK codification time (2026-07-11). Not part of the spec.

## The portable pattern

- When the traceability audit activates, the state board gains a **required, always-present section** synthesizing the latest run: most recent run (date, number, outcome), findings count by category, pending remediation with status, next scheduled run, a pointer to the full report, and carry-forward items not closed at the run's closeout.
- **"Always present" is deliberate** — "Last audit: [date]: zero violations" is valuable orientation, not noise. Empty ≠ omitted.
- **Division of labor ports as-is:** the full report stays in the audit directory; the board surfaces only the actionable summary; the **advisor** updates this section at next session start after a run completes (the audit itself never writes the board).
- PGK Core's board template excludes this section on purpose: it presumes the audit exists. Add it (in the board's section order, after "Don't touch right now") the moment the audit is active.

## GEO-specific — strip or replace on activation

- "Skill v1.1 §6" / "Cowork operating discipline §9.4" references → your audit spec's remediation-drafting section and your session-discipline doc's trigger-update rule.
- `docs/audits/[date]-traceability-audit-runN.md` path → your audit output convention (the naming pattern itself is portable).
- Friday-run scheduling references → your cadence.

## Evolution note (observed in the live system, 2026-06)

GEO's live board later evolved this section into a **machine-managed factual zone** — `<!-- BEGIN:sb-traceability -->` markers with a small field/value table, self-healed by the automation loop, plus a sibling auto-remediation zone. That is an orchestrator-tier refinement (generator-owned sections inside an advisor-owned document): adopt it only after the orchestrator/hot-cache layer is running; before that, the advisor maintains this section by hand per the spec text.
