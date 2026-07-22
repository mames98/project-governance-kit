# Project Governance Kit (PGK)

A portable, lightweight governance template for AI-built projects. Copy it into a new project and you get — from day one — a single place that tracks where things stand, a decision log that preserves the *why* behind every choice, and a session-start discipline so no context is ever lost between sessions or when a session dies mid-work.

PGK is deliberately small. It is records-and-discipline, not automation. The whole point is that it's cheap to instantiate and impossible to outgrow — you add heavier machinery (schedulers, hooks, dashboards) only if a specific project earns it.

## Why this exists

AI-built projects accumulate decisions fast, across many sessions, with an assistant that forgets everything between sessions. Without a governance layer, three things go wrong:
- **Lost context** — a session ends or crashes and the next one can't tell where things stood.
- **Un-provenanced decisions** — choices get made, then re-litigated months later because nobody recorded *why*.
- **Drift** — the project's actual state and what any document claims about it silently diverge.

PGK fixes all three with three files and one rule.

## What's in the kit

| File | What it is |
|---|---|
| `templates/state-board.md` | The single-surface "where things stand right now" — read first, every session. |
| `templates/decision-log.md` | The append-only record of decisions (ADRs). The *why* trail. |
| `templates/session-start.md` | The discipline every session follows: read the board first, and how to hand off. |
| `INSTANTIATE.md` | Greenfield setup — a procedure a Claude session executes in a new repo (manual checklist included as an appendix). |
| `ADOPT.md` | Brownfield adoption — a Claude session scans an existing repo read-only, proposes governance reconstructed from evidence, and writes only what you confirm. |
| `bootstrap/` | Fill-in skeletons that wire in the one rule: a two-part `CLAUDE.md` skeleton and paste-ready assistant instructions for advisor sessions. |

PGK has two layers: **Core** (the files above — instantiated day one, every project) and an **Automation Tier** (`automation-tier/` — deferred, self-contained reference specs for the scheduled traceability audit, auto-remediation, and the governance orchestrator/hot cache; activated per `automation-tier/README.md` only when a project earns it).

## The one rule

**Every working session reads the state board before doing anything else, and updates it before ending.** Everything else in PGK supports that one rule.

## How to use it

Two paths, both run by pointing a Claude session at a single file:

- **New project** → `INSTANTIATE.md`. The session places the templates, fills what it can verify, interviews you for the rest (core premise → ADR-001, decision owners, session roles, authoritative docs), wires in the one rule via `bootstrap/`, and hands you a ready-to-ratify commit. A manual checklist survives as an appendix if you'd rather do it by hand.
- **Existing repo** → `ADOPT.md`. The session scans read-only, proposes an authority table, a state board, and ADR candidates reconstructed from evidence — the *why* of every candidate stays blank until you supply it — then writes only what you confirm, anchored by ADR-000 marking the adoption line.

Multi-session governance also works across multiple human operators: the templates parameterize the decision-gating authority as `[DECISION_OWNER]` (defined in each template; single-operator projects fill one value everywhere).

## Design principles

- **Records over automation.** The kit is markdown. Add automation only when a project's size justifies it.
- **One instance per project.** Each project gets its own filled-in copy — never a shared store. (Governance is per-project, like a project's own config.)
- **Instance first teaches the template.** Each new project you instantiate PGK into reveals what's truly generic vs. what was project-specific. Fold generic improvements back into PGK; leave the specific stuff in the instance.
- **Honest state.** The board reflects what's *actually* true, verified — not what a document hopes is true. Flag anything unverified as unverified.
