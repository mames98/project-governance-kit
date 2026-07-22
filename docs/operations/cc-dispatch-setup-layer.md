# CC Dispatch — PGK Setup Layer (Goal + Loop, autonomous-to-done)

**Date:** 2026-07-22  **Author:** Cowork (facilitator), operator-ratified
**Type:** Build against a ratified spec. Commits land on `main` of `~/project-governance-kit` (public repo); push only after all success criteria pass the exit gate.
**Method:** Loop method — plan first → /goal with success criteria → loop until all criteria met. Auto mode, high effort. Run until all SC ✅ or a cap is hit.
**State file:** `docs/operations/cc-dispatch-setup-layer.state.md` — update after EVERY iteration.
    FORMAT (compact board, not narrative): SC status table · attempt counter · failed-approaches
    list (one line each: approach → why it failed) · next step. On session start, read it first
    and resume from recorded state; never restart from zero.

**Spec authority:** `docs/specs/setup-layer-spec-v1.0.md` (placed by Cowork, uncommitted — commit it as part of this dispatch's work). The spec is the content authority; this dispatch is the execution contract. On any conflict, PAUSE and surface.

**Trigger:** Operator decision (Cowork session 2026-07-22): PGK needs a setup layer — greenfield instantiation is human-addressed and under-wires "the one rule"; brownfield adoption has no path at all; Core's single-operator language blocks the enterprise pitch.
**The real ask:** After this dispatch, pointing a Claude session at INSTANTIATE.md (new repo) or ADOPT.md (existing repo) produces a correctly-wired, honestly-initialized PGK instance — with no fabricated provenance possible by construction — and the Core templates read multi-operator-neutral.

## Pre-execution requirements

(a) Confirm HEAD is `da8c53f` ("Add self-contained Automation Tier…"). If HEAD moved, verify the moves are operator commits and proceed only if the working files below are untouched; else halt and surface.
(b) Working tree shows exactly 2 untracked files: `docs/specs/setup-layer-spec-v1.0.md`, `docs/operations/cc-dispatch-setup-layer.md` (this file). Any other modification → halt and surface.
(c) `~/geo-machine-jancal` is used READ-ONLY as the brownfield scan fixture (SC-6). Never write there.

## STAGE 1 — PLAN FIRST (BEFORE any file changes)

Read the spec end-to-end, then plan using the strongest planning/execution skill installed in this project; fall back to native plan mode if none respond. Form your OWN reading of the spec; the component descriptions in it are ratified requirements, but the *decomposition into edits* is yours. Exit Stage 1 only when the plan is written with file paths and every SC below is addressable.
If forked subagents are available, fork the planning helper (and later any parallel worker sub-tasks, e.g. Component A vs Component C drafting); otherwise use regular subagents. The exit-gate critic is NEVER forked.

## STAGE 2 — THE GOAL (run /goal against this)

> **Goal:** Implement PGK Setup Layer Spec v1.0 in full — INSTANTIATE.md v2, `bootstrap/` skeletons, ADOPT.md, Core neutralization, and README/doc updates — verified by live execution of both procedures. Loop until every success criterion below passes adversarial verification. Do not stop early. Do not touch the criteria.

## SUCCESS CRITERIA (loop exit condition — ALL must be ✅; FROZEN once the loop starts)

- **SC-1 — INSTANTIATE.md v2 conforms to spec §3:** addressed to a Claude session with operator preamble; all phases A0–A8 present in order; manual-path appendix preserves the current checklist content including the (updated) "What NOT to add yet" section.
- **SC-2 — `bootstrap/` conforms to spec §4:** both skeleton files exist, placeholder-driven, referenced from INSTANTIATE.md A6; CLAUDE-md skeleton uses the PART 1 / PART 2 pattern with the board-read directive in PART 1.
- **SC-3 — ADOPT.md conforms to spec §5:** phases C1–C4 in order; hard rules (read-only scan, no invented whys, evidence citations, never pushes) stated inside the file; ADR-000 adoption-line requirement and reconstructed-status format present verbatim in intent.
- **SC-4 — Neutralization conforms to spec §6:** in the three Core templates, `grep -n "operator"` output shows only generic-session usage or the `[DECISION_OWNER]` definition; Open-decisions section carries owner; session-start has the multi-operator note; a diff review (logged in state file) confirms zero semantic changes to discipline rules.
- **SC-5 — LIVE greenfield run passes (spec §8.1):** execute INSTANTIATE.md v2 yourself against a fresh `/tmp` scratch git repo using Appendix A fixture answers; the produced instance meets every §8.1 condition; transcript/evidence logged; scratch repo deleted after.
- **SC-6 — LIVE brownfield scan passes (spec §8.2):** execute ADOPT.md C1–C2 against `~/geo-machine-jancal` (read-only); the proposal artifact (kept under `/tmp`, not committed) contains zero uncited claims and zero filled-in whys; `git -C ~/geo-machine-jancal status --porcelain` is unchanged before/after.
- **SC-7 — Documentation updates conform to spec §7:** README kit table + two-path usage; mutual INSTANTIATE↔ADOPT cross-references; every cross-referenced path in the repo resolves (`grep`-check links against `ls`).
- **SC-8 — Repo integrity:** markdown-only (no scripts/binaries added anywhere, spec §8.4); `automation-tier/` byte-identical to HEAD (`git diff --stat da8c53f -- automation-tier/` empty); spec file committed; explicit named-path staging (no `-A`); why-rich commit message; pushed; `git status -sb` shows sync.

SCs are immutable mid-run. If one proves wrong or untestable, PAUSE action-oriented: evidence + recommended amendment + rationale, then wait for operator ratification. Never weaken, reinterpret, or delete a criterion yourself.

## VERIFICATION PROTOCOL (adversarial — self-grading does not count)

Each iteration: self-check SCs cheaply (greps, file reads, the live runs for SC-5/6 once components stabilize). When you believe ALL SCs pass, invoke the EXIT GATE: a fresh-context critic subagent (different model if available), given ONLY this dispatch + the spec + the changed files + the SC-5/6 evidence artifacts — not your reasoning — instructed: "Assume this work is broken. Try to make each SC fail. Hunt specifically for: an uncited claim in the SC-6 proposal; a fabricated why; a semantic change hidden in the neutralization diff; a phase missing or reordered; a cross-reference that doesn't resolve." Any SC the critic breaks returns to the loop with the critic's evidence logged in the state file. No exit without surviving the critic. A critic that approves everything first pass is broken — tighten it and re-run.
The critic MUST be a regular (fresh-context) subagent — NEVER a forked subagent.

## SUBAGENT MODE (forked vs fresh)

If forked subagents are available, FORK worker-side helpers: the Stage-1 planning helper and parallel component drafting (A/B vs C). If unavailable, use regular subagents — do not block. The EXIT-GATE CRITIC and every SC verifier are ALWAYS fresh-context regular subagents, never forks.

## CAPS (circuit breakers — hitting one = write state, PAUSE, surface to operator)

- Max loop iterations: 10
- Max consecutive failures of same SC: 3 (same failure 3× = wrong diagnosis; return to Stage 1 or pause). On any re-plan: the new approach must be from a CATEGORICALLY different family than everything on the state file's tabu list.
- Spend: $15 total for this dispatch.
A cap hit is a mandatory stop with the state file as handoff + a recommended next step (continue / re-plan / abandon, with rationale).

## AUTONOMY CONTRACT

You may freely: decompose the spec into edits, draft and iterate all files, run the live verifications, commit when verified, push after the exit gate passes.
**NEVER (prohibited outright):** modify anything under `automation-tier/`; write to `~/geo-machine-jancal`; change the meaning of any Core discipline rule (spec §6 semantics-frozen clause); add scripts, code, or tooling to the PGK repo; edit the spec (conflicts → PAUSE); `git add -A` / `git add .`; force-push; weaken an SC; modify the checks that verify SCs; re-propose a tabu-listed approach; modify this dispatch's protocol/caps mid-run.
**PAUSE-AND-ASK (surface to operator):** any spec ambiguity or internal conflict; `~/geo-machine-jancal` unavailable as the SC-6 fixture (propose an alternative, wait); any SC amendment (with recommendation attached); anything not covered by the spec that would change what a PGK user sees.

## SCOPE GUARD

- IN: spec §§3–7 deliverables, §8 verifications, the commit(s) + push.
- OUT: automation-tier changes, skill packaging, template redesign beyond §6, STE or any instantiation into a real project, dispatch-templating codification. Satisfy the criteria and stop.

## COMPREHENSION HANDOFF (before declaring done)

Short plain-English summary: what changed and why, what the critic caught across iterations, what remains fragile (e.g., untested surfaces for the procedures), and the commit hash(es). The operator must be able to audit the result, not rubber-stamp it.

## Principles Alignment

Records over automation (markdown-only, §2.4) · honest state / never fabricate provenance (§2.1–2.2 — enforced by SC-6's zero-uncited-claims gate) · operator gates (procedures propose, never push, §2.3) · instance-first (§2.6 — minimum shippable, elaborations wait for real use). No conflicts identified.
