# Instantiating PGK in a new project

Copy-paste-fill. No scripts. Fifteen to thirty minutes for a fresh project.

## The checklist

**1. Copy the templates in.**
Copy the three files from `templates/` into your project — `docs/governance/` is the default home, but put them wherever your project keeps docs. Conventional layout:

- `docs/operations/state-board.md` (the living board — it's operational, not archival)
- `docs/decisions/` (one file per ADR) with `decision-log.md`'s conventions section as `docs/decisions/README.md`
- `docs/governance/session-start.md`

A flat `docs/governance/` for all three also works. Pick one and be consistent.

**2. Fill the placeholders.**
Every `[PLACEHOLDER]` in the templates needs a real value:

- `[PROJECT_NAME]` — the project.
- `[STATE_BOARD_PATH]` — wherever you put the board in step 1.
- `[BOARD_OWNER_ROLE]` / `[BUILDER_ROLE]` — your session roles (e.g. "Cowork" / "Claude Code", or "advisor session" / "build session"). Single-session project: one role carries both; fill accordingly.
- **The authority table (board footer):** name YOUR project's authoritative documents — the spec or design doc that governs architecture, the roadmap or plan that governs sequencing, the backlog that governs work inventory. The board points at them; it never replaces them. If the project doesn't have these docs yet, list `git log` alone and add docs as they exist — don't invent placeholder documents.

**3. Initialize the board honestly.**
Fill every section with what is *actually* true right now. Empty sections get explicit empty-state statements ("Nothing uncommitted as of [date]", "No decisions open", "No coordination flags active") — never left blank, never filled with aspirations. Set the freshness marker.

**4. Write ADR-001.**
The first decision record captures the project's core premise — the measurable success criterion, the load-bearing constraint, whatever choice everything else builds on. Pinning it first means it can't silently drift later.

**5. Wire in the one rule.**
Put "read the state board before any substantive work; update it before ending" wherever your sessions bootstrap from — CLAUDE.md, project instructions, a system prompt. The kit only works if sessions actually hit the board first. A useful pattern: a two-part bootstrap file where PART 1 is standing rules (stable, changes require explicit decision) and PART 2 is operational state (living) — the board-read directive goes in PART 1.

**6. First real session: verify.**
The first working session after instantiation reads the board, verifies every claim against reality (git log, actual files, actual running state), and fixes anything the initialization got wrong. Governance that starts inaccurate stays distrusted.

## What NOT to add yet

PGK baseline is records-and-discipline. The heavier capability is **deferred but codified**: the real reference specs live self-contained in `automation-tier/` (copied verbatim from the working system PGK was extracted from, with per-spec generalization notes). When a project earns a component, activate it per `automation-tier/README.md` — don't re-derive it:

- **Traceability audit** (scheduled spec-vs-reality drift audit with classified findings + drafted fixes). Earned when: the project has enough spec surface that drift can hide.
- **Auto-remediation** (autonomous application of audit-drafted mechanical fixes, quarantined then graduated). Earned when: audit findings recur mechanically and carry forward unaddressed run after run.
- **Orchestrator + hot cache** (self-maintaining orientation cache, session hooks, governance-loop monitor, machine-managed board sections, session locks). Earned when: the project runs always-on and manual governance maintenance is a real cost, or the board goes stale repeatedly *despite* the discipline being followed.
- **Dispatch templating** (structured, copy-paste task briefs from advisor session to build session, with scope / stop conditions / verification). Earned when: an advisor/builder split is actively running and briefs are being rewritten from scratch each time. (Not yet codified in the tier; the pattern lives in the source system's operating discipline.)

Adding these speculatively is over-engineering. The board plus the decision log plus the one rule is the whole baseline.

## Feeding improvements back

Instance-first teaches the template: when using PGK in a project reveals that something in these templates is actually project-specific — or that something generic is missing — fix the instance first, then fold the generic part back into this kit.
