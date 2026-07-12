# PGK Automation Tier

The heavier, autonomous layer of project governance — **deferred by default, self-contained here**.

PGK Core (the state board, the decision log, the session discipline) is instantiated on day one of every project. This tier is NOT. It is the machinery a project adds only when it has earned it — when manual governance maintenance has become a real, recurring cost. Until then, records-and-discipline is the whole kit.

This folder is self-contained: the real specifications live in `specs/`, copied verbatim from the working system they were built and proven in (GEO Machine). They are **reference implementations**, not fill-in templates — activating this tier means generalizing them for your project per the per-spec map in `generalize/`. Nothing here requires the source project to exist.

## The three components and how they relate

The tier is a loop:

1. **Traceability audit** (`specs/traceability-check-skill-spec.md`) finds drift — a scheduled, read-only audit that checks every governance claim traces to reality, classifies findings, and *drafts* exact-text remediation for mechanical cases.
2. **Auto-remediation** (`specs/auto-remediation-spec.md`) applies the drafted fixes — a three-tier execution cycle (auto-execute / execute-and-flag / halt-and-interrupt) constrained to a pre-ratified pattern allowlist, with a quarantine branch and a 30-day graduation gate.
3. **The orchestrator + hot cache** (`specs/agentic-os-dispatch.md`) keeps orientation current and watches the loop — a marker-based generated "hot cache" for sub-30-second session orientation, refresh triggers, and a governance monitor that alerts on staleness, aging decisions, and health thresholds.

A fourth piece, `specs/state-board-audit-section.md`, is the board section that surfaces the audit loop's state on the PGK Core state board — Core's board template deliberately excludes it; it joins the board when this tier activates.

## Earned-when triggers

| Component | One line | Earned when |
|---|---|---|
| Traceability audit | Scheduled spec-vs-reality drift audit with classified findings + drafted fixes | The project has enough spec/governance surface that drift can hide — claims about what's shipped, current, or where things live start going stale without anyone noticing. |
| Auto-remediation | Autonomous application of audit-drafted mechanical fixes, quarantined then graduated | Audit findings recur mechanically and carry forward run after run because each individual fix is too small to justify operator attention. |
| Orchestrator + hot cache | Self-maintaining orientation cache + governance-loop monitor | The project runs always-on (crons, multiple concurrent sessions) and manual board/orientation maintenance is a real cost — or sessions repeatedly start from stale context despite Core discipline being followed. |
| State-board audit section | Board section synthesizing the latest audit run | Automatic — add it the moment the traceability audit is active. |

Activate in that order. The audit is useful alone; auto-remediation is meaningless without it; the orchestrator assumes both exist (its "Loop" is audit → remediation → refresh).

## Status of the copies

Each file in `specs/` opens with a provenance block: source path, source version, and the source-repo commit it was copied at. Below that block the content is byte-faithful to the source. The copies are GEO-Machine-shaped — they reference that project's authority documents (Codex, roadmap, backlog), paths, hosts, Discord wiring, and role names (Cowork/CC). The `generalize/` notes map, per spec, what is the **portable pattern** versus what to **strip or replace** on activation. The notes are summaries *about* the specs, written at codification time — they are not part of the specs.

Full generalization happens at activation, not before. That is deliberate: generalizing a spec you aren't using yet is speculative work, and the verbatim copy preserves details a premature generalization would lose.
