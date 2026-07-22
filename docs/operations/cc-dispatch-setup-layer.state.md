# State — CC Dispatch: PGK Setup Layer

Compact board for `docs/operations/cc-dispatch-setup-layer.md`. Read first on session start; resume from here.

**Iteration:** 4 (of max 10) · **Phase:** EXIT GATE SURVIVED — finalizing (state commit + push) · **Spend cap:** $15

## Exit gate log

**Critic #1** (fresh-context, opus, given dispatch + spec + changed files + SC-5/6 evidence only): attacked all 8 SCs — re-derived checks itself (10+ hash spot-checks against the brownfield fixture, independent placeholder sweeps, line-by-line semantic diff, full cross-ref extraction). Verdict: 8× SURVIVED, 0 broken. Near-misses: (1) session-start L17 [DECISION_OWNER] vs neutral phrasing inconsistency claim — DISPOSITION: keep; the original assigned the redirect duty to "the operator" (the gating human), so [DECISION_OWNER] is the faithful rename, while "operator vigilance" was generic prose → neutral, per spec §6's two-pronged rule. (2) SC-6 proposal "Honest note" summary sentence — clears the bar (summary of cited rows directly above; no new fact). (3) SC-5 evidence was for run-2 text + one wording-only edit, not the final committed text executed verbatim — ACTION: SC-5 run 3 launched against committed HEAD text, fresh scratch (`0a6dbad` stub), evidence to be regenerated.

**Per dispatch rule** ("a critic that approves everything first pass is broken — tighten it and re-run"): critic #2 will run after SC-5 run 3, tightened — different attack set: spec §3 phase-content conformance (per-phase requirements, not just headers/order), §2 binding-principles embedding in both procedures, §4 skeleton content requirements, §6 README sentence, appendix "(updated)" reading, and the three near-miss surfaces above.

**Critic #2** (fresh-context, opus, content-conformance surface: per-phase §3/§5 requirement extraction, §2 principle operationalization, §4 skeleton elements, §6 definition exactness, cross-file contradiction hunt, evidence-chain pinning, state-file honesty): 8× SURVIVED, zero SC-breaking defects on deliverables. Found: **Defect 1 (real, fix-before-commit)** — this state file's SC-5 row still cited run 2/`b12a951` after run 3 superseded it → CORRECTED above. Defect 2 (cosmetic) — session-start [DECISION_OWNER] Template note sits one line after first use; disposition: keep — adjacent placement is a fair reading of "defined on first use" and the note must follow the sentence it explains. Defect 3 (cosmetic) — manual-path step 1 wording ("conventions section as README") narrower than procedure A1 (whole file travels); disposition: keep — the appendix is preservation-mandated (§3: content preserved, updates limited to §6/§7), and the manual path works standalone as written. Critic #2 also independently confirmed this file's SC-4 diff-review claim accurate.

**Gate verdict:** two independent adversarial passes, distinct attack surfaces, 16/16 SURVIVED; critic #2 produced a substantiated defect (state-file staleness), demonstrating the gate had teeth. Exit gate passed.

## Untested surfaces (from critic #2 — honest fragility list for the operator)

- A6's additive-merge branch (pre-existing CLAUDE.md) never executed live — both fixtures were clean.
- ADOPT C3/C4 (CONFIRM/WRITE) never executed live — SC-6 mandates C1–C2 only; ADR-000 generation and reconstructed-ADR writing are read-verified, not run-verified. First real adoption should be watched.
- Multi-operator routing ([DECISION_OWNER] with distinct owners) never exercised — all runs single-operator.
- assistant-instructions.md never pasted into a real advisor surface.
- A7 grep pattern portability (BSD vs GNU grep alternation) unverified.

## SC status

| SC | What | Status | Evidence |
|---|---|---|---|
| SC-1 | INSTANTIATE.md v2 (spec §3) | 🟡 drafted, self-check passed | A0–A8 headers in order (grep); preamble + ADOPT cross-ref; manual-path appendix preserved, edits limited to §6 ([DECISION_OWNER] in step-2 list) + §7 (bootstrap/ pointer in step 5); WNTAY verbatim |
| SC-2 | bootstrap/ skeletons (§4) | 🟡 drafted | Both files exist; PART 1/PART 2 pattern with board-read directive in PART 1; referenced from A6 |
| SC-3 | ADOPT.md (§5) | 🟡 drafted, self-check passed | C1–C4 in order (grep); 4 hard rules in file; ADR-000 adoption line + reconstructed-status format verbatim (grep = 1 each) |
| SC-4 | Neutralization (§6) | 🟡 diff-reviewed | grep: remaining "operator" only inside [DECISION_OWNER] definitions + spec-mandated multi-/single-operator phrasing; decision-log.md had zero occurrences → untouched; diff review: renames/parameterization + additive owner column + additive multi-operator note, zero semantic changes to discipline rules |
| SC-5 | Live greenfield run (§8.1) | ✅ passed (run 3, executing the exact committed text at kit HEAD `0b2c2f3`) | All 6 §8.1 conditions verified by dispatcher directly on the run-3 instance: token-grep empty (exit 1), 8 sections in template order, honest board (real open-decision + coordination-flag entries; empty sections dated empty-states), ADR-001 in format + indexed, freshness set, rule reachable, nothing committed (fixture sole commit `0a6dbad`). Runs 1–2 passed §8.1 on earlier text and drove 5+1 procedure improvements. Evidence: /tmp/pgk-sc5-evidence.md; transcript /tmp/pgk-sc5-transcript.md (pins executed HEAD). [Row corrected per critic #2 Defect 1 — previous row cited run 2/`b12a951`, superseded by run 3] |
| SC-6 | Live brownfield scan (§8.2) | ✅ passed | Porcelain before/after identical (clean); 35 observations + 4 proposals all evidence-cited; 3 [unverified] flags; zero filled whys (all 6 candidates literal "Why: (left blank for the operator)"); evidence: /tmp/pgk-sc6-evidence.md, artifact /tmp/pgk-sc6-proposal.md (196 lines, uncommitted) |
| SC-7 | Doc updates + cross-refs (§7) | 🟡 drafted, self-check passed | README table rows + two-path usage + multi-operator sentence; all 11 cross-referenced paths resolve (ls check) |
| SC-8 | Repo integrity + push | ✅ (push in progress at final state write) | Markdown-only (both critics confirmed: every added file .md; only non-md are pre-existing LICENSE + git-ignored harness settings file); `git diff da8c53f HEAD -- automation-tier/` empty; spec committed (`5b3030a`); 7 named-path commits, no `-A` (critic #2 verified the uncommitted state file was never swept in), why-rich messages; push + `git status -sb` sync check executed immediately after this file's ratification commit |

## Pre-execution record

- HEAD at dispatch start: `77e9c68` (moved from `da8c53f` by 2 operator commits, mames98, feedback/ only; templates/INSTANTIATE/README/automation-tier byte-identical to `da8c53f` — verified `git diff --stat da8c53f -- automation-tier/ templates/ INSTANTIATE.md README.md` empty). Proceeded per contingency clause (a).
- Untracked at start: exactly `docs/specs/setup-layer-spec-v1.0.md`, `docs/operations/cc-dispatch-setup-layer.md`. ✓ clause (b).
- `~/geo-machine-jancal` present, readable; READ-ONLY fixture. ✓ clause (c).
- Forked subagents: unavailable (agent registry has no fork type) → regular subagents per SUBAGENT MODE.

## Attempt counter

| SC | Consecutive failures |
|---|---|
| (none yet) | 0 |

## Tabu list (failed approaches — never re-propose)

(empty)

## Iteration 2 — friction fixes (from SC-5 run 1's friction log; quality fixes to own drafts, no spec deviation, no SC touched)

1. INSTANTIATE A1: ADR skeleton now travels into the instance's decisions README (kit path is machine-specific; instance must be self-contained). A5 updated to match.
2. INSTANTIATE A7 check 1: bracket-grep replaced with a specific-token grep that CAN return zero (old `grep "\["` structurally always hit template grammar → judgment-call verification).
3. INSTANTIATE A6.2: filled assistant-instructions text now saved as `docs/governance/assistant-instructions.md` in the instance (was transcript-only → lost if operator didn't save chat); A8 add-list updated.
4. session-start.md: [DECISION_OWNER] definition moved from mid-sentence appositive (read as broken prose after fill) to a deletable *Template note* line; multi-operator note wording smoothed ("[BOARD_OWNER_ROLE] stays a single role"). SC-4 grep re-verified clean after edit.
5. INSTANTIATE A4: one sentence added — first board honestly describes the instantiation itself; first working session updates it away.

Not acted on (logged for operator instead): SC-6 friction items 1–5 (ADOPT.md spec gaps: scan-ref, already-governed targets, dual decision logs, local/remote divergence, merge-status depth) — spec §5 doesn't cover them; adding them would change what a PGK user sees beyond the ratified spec → per PAUSE-AND-ASK boundary they go to the comprehension handoff as recommended spec-v1.1 candidates, not into this dispatch's edits.

## Dispositions log (for the critic)

- `templates/decision-log.md`: grep shows zero "operator" occurrences → no neutralization edit needed; file untouched. Spec §6 replaces the word where it names the decision-gating authority; absent occurrences, the conforming change is zero.
- Plan file: scratchpad (not committed — working material, not a spec deliverable).
- Dispatch + spec + this state file will be committed (spec header references the dispatch path; SC-7 requires cross-refs to resolve).

## Next step

DONE pending: commit this state file (named path) → push → verify `git status -sb` sync → delete scratch fixture → comprehension handoff to operator. Recommended follow-ups (operator's call, out of this dispatch's scope): spec v1.1 candidates from the live-run friction logs — ADOPT scan-ref rule, already-governed-target guidance, dual-decision-log handling, local/remote branch divergence, merge-status depth; plus the untested surfaces above.
