# Instantiating PGK in a new project

**For the operator:** point a Claude session at this file in your new repo — "read `INSTANTIATE.md` and follow it" — and answer its questions. The session places the templates, fills what it can verify, interviews you for the rest, wires in the one rule, and hands you a ready-to-ratify commit. It never commits or pushes without your explicit go-ahead. Prefer to do it by hand instead? Use [Appendix — Manual path](#appendix--manual-path) (15–30 minutes).

**Existing repo with history?** This procedure is for greenfield projects. For adopting PGK into a project that already has commits, branches, and decisions, use `ADOPT.md` — it reconstructs governance from evidence instead of starting blank.

---

## Procedure (addressed to the Claude session)

You are instantiating the Project Governance Kit into this repository. Execute the phases below **in order**. Ground rules, binding throughout:

- **Evidence or `[unverified]`.** Every claim you record about the project cites its evidence (a file path, a commit hash) or is flagged `[unverified]`. No evidence, no claim.
- **Propose, don't push.** You hand the operator a filled draft to ratify. Never push. Never commit without the operator's explicit go-ahead in this session.
- **Never overwrite.** If a file you would place already exists (especially `CLAUDE.md`), propose an additive merge and show it — never replace existing content.
- **Governance only.** Take no position on the project's stack, repo structure, CI, or tooling. You are wiring in records and discipline, nothing else.

### A0 — Preconditions

1. Confirm you are in a working repo: `git rev-parse --show-toplevel`. **If this is not a git repo, pause and say so — PGK assumes git** (its history discipline leans on `git log`). Do not `git init` on your own; ask the operator how to proceed.
2. Confirm where docs live. Default is `docs/`; if the repo already has a different docs convention, name it and use it.
3. Confirm with the operator: "This will instantiate PGK governance (state board, decision log, session discipline) into this repo. Proceed?" Stop here until they answer.

### A1 — Place files

Copy the three templates from the kit's `templates/` into the project, following the kit's placement conventions:

- `docs/operations/state-board.md` — the living board (operational, not archival)
- `docs/decisions/` — one file per ADR; `templates/decision-log.md` (conventions + the Index table + the ADR skeleton) becomes `docs/decisions/README.md`. The skeleton travels into the instance so it stays self-contained — a future session copies it from its own decisions README, without needing the kit on the machine.
- `docs/governance/session-start.md` — the session discipline

A flat `docs/governance/` for all three also works. If the repo's docs convention (A0.2) differs, adapt paths consistently and record what you chose.

### A2 — Fill what is verifiable

Fill every placeholder you can resolve **without asking**, citing evidence for each:

- `[PROJECT_NAME]` — from the repo name, manifest, or README (cite which).
- `[STATE_BOARD_PATH]`, `[DECISIONS_DIR]`, `[SESSION_START_PATH]` — the paths you just chose in A1.
- Anything else readable from the repo (existing docs worth candidate spots in the authority table, etc.) — cite the file.

What you cannot verify, leave for A3. Do not guess.

### A3 — Interview the operator

Ask **at most 4 questions, batched in one message**:

1. **Core premise:** what is this project's core premise or measurable success criterion — the choice everything else builds on? (Becomes ADR-001.)
2. **Decision owner(s):** who gates decisions for this scope — one person, or several owners for different scopes? (Fills `[DECISION_OWNER]`; the templates define it as: the person or role who gates decisions for this scope; in a single-operator project, that's you.)
3. **Session roles:** advisor + builder split, or a single session role carrying both? (Fills `[BOARD_OWNER_ROLE]` / `[BUILDER_ROLE]`.)
4. **Authoritative docs:** which documents (existing or planned) govern architecture, sequencing, and work inventory? (Fills the board's authority table.)

**Absence is a valid answer — record it honestly.** "No authoritative docs yet; `git log` is the only authority" is a correct authority table, not a gap to paper over. Never invent placeholder documents.

### A4 — Initialize the board honestly

Fill every section of the state board with what is *actually true right now*:

- Sections with real content get real content (with evidence, per the ground rules).
- Empty sections get **explicit empty-state statements** — "Nothing uncommitted as of [date]", "No decisions open as of [date]", "No coordination flags active" — never left blank, never filled with aspirations.
- Set the freshness marker (`**Last updated:**` line) to today, with context "PGK instantiation".
- It is honest for the first board to describe the instantiation itself — the governance files awaiting ratification *are* the real work in flight. The first working session updates that away; say so in the handoff.

### A5 — Write ADR-001

From the operator's answer to A3.1, write `ADR-001` in `[DECISIONS_DIR]` using the ADR skeleton now in the instance's decisions README, and add it to the Index there. Status: `Accepted — [operator's name/role] signed off [date]` (they gave you the premise directly; that is the sign-off, in this session). Context and Source ground it in what the operator actually said, not embellishment.

### A6 — Wire the one rule

The kit only works if sessions actually hit the board first. Wire that using the two skeletons in the kit's `bootstrap/`:

1. **`bootstrap/CLAUDE-md-skeleton.md`** — fill its placeholders with this project's values and place the result as `CLAUDE.md` (PART 1 standing rules, PART 2 operational state). **If a `CLAUDE.md` already exists, propose an additive merge** — show the operator exactly which lines you would add and where; never overwrite.
2. **`bootstrap/assistant-instructions.md`** — fill its placeholders and save the result as `docs/governance/assistant-instructions.md` in the project, so the filled text survives this session. Then offer it to the operator for whatever surface their advisor-type sessions bootstrap from (claude.ai project instructions, a system prompt, or equivalent) — to paste now, or whenever they first set such a surface up.

### A7 — Verify

Check, and show the results:

1. **Zero unresolved placeholder fields** in the placed files. Check the specific tokens — this must return nothing:

   ```
   grep -rn "\[PROJECT_NAME\]\|\[STATE_BOARD_PATH\]\|\[SESSION_START_PATH\]\|\[DECISIONS_DIR\]\|\[BOARD_OWNER_ROLE\]\|\[BUILDER_ROLE\]\|\[DECISION_OWNER\]\|\[SESSION_NAME\]\|\[COMMIT_HASH\]\|\[AUTHORITATIVE_DOC\|Template note" docs/ CLAUDE.md
   ```

   (Adjust the searched paths to whatever A1 actually chose. Bracketed notation like `[unverified]` or the status grammar `[who] signed off [date]` is template grammar, not a fill site — it stays.) Any hit is either fixed or explicitly deferred with a reason recorded next to it; *Template note* lines are deleted once filled.
2. **Board sections all present, in template order**, each either filled or carrying an explicit empty-state.
3. **ADR-001 exists**, is in the decision-log format, and is indexed.
4. **The one rule is reachable from the session bootstrap path:** `CLAUDE.md` (or the merged equivalent) contains the board-read-first directive and points at the real board path.

Any check that fails: fix it and re-verify before proceeding.

### A8 — Hand off

1. Give the operator a plain-language summary: what was created, what was **inferred** (with evidence), and what came from their **answers**.
2. Give them the exact ratification commit — named paths only, never `git add -A`:

   ```
   git add docs/operations/state-board.md docs/decisions/README.md docs/decisions/ADR-001-<slug>.md docs/governance/session-start.md docs/governance/assistant-instructions.md CLAUDE.md
   git commit -m "Instantiate PGK governance: state board, decision log (ADR-001), session discipline, bootstrap wiring"
   ```

   (Adjust paths to what A1 actually chose.)
3. **Stop.** Committing is the operator's call. The first real working session then verifies the board against reality per `session-start.md` — governance that starts inaccurate stays distrusted.

---

## Appendix — Manual path

Copy-paste-fill by hand. No scripts. Fifteen to thirty minutes for a fresh project.

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
- `[DECISION_OWNER]` — the person or role who gates decisions for this scope; in a single-operator project, that's you. Multi-operator projects fill different owners per decision on the board's "Open decisions" section.
- **The authority table (board footer):** name YOUR project's authoritative documents — the spec or design doc that governs architecture, the roadmap or plan that governs sequencing, the backlog that governs work inventory. The board points at them; it never replaces them. If the project doesn't have these docs yet, list `git log` alone and add docs as they exist — don't invent placeholder documents.

**3. Initialize the board honestly.**
Fill every section with what is *actually* true right now. Empty sections get explicit empty-state statements ("Nothing uncommitted as of [date]", "No decisions open", "No coordination flags active") — never left blank, never filled with aspirations. Set the freshness marker.

**4. Write ADR-001.**
The first decision record captures the project's core premise — the measurable success criterion, the load-bearing constraint, whatever choice everything else builds on. Pinning it first means it can't silently drift later.

**5. Wire in the one rule.**
Put "read the state board before any substantive work; update it before ending" wherever your sessions bootstrap from — CLAUDE.md, project instructions, a system prompt. The kit only works if sessions actually hit the board first. A useful pattern: a two-part bootstrap file where PART 1 is standing rules (stable, changes require explicit decision) and PART 2 is operational state (living) — the board-read directive goes in PART 1. Ready-to-fill skeletons for both surfaces live in `bootstrap/` (`CLAUDE-md-skeleton.md`, `assistant-instructions.md`).

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
