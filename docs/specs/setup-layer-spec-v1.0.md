# PGK Setup Layer — Specification

**Version:** 1.0
**Status:** Ratified by operator (2026-07-22, Cowork session); implementation via CC dispatch `docs/operations/cc-dispatch-setup-layer.md`
**Scope:** Adds the setup layer to PGK — Claude-executed greenfield instantiation, bootstrap skeletons, brownfield adoption procedure — plus multi-operator language neutralization of the Core templates.

---

## 1. Purpose

PGK Core works only if sessions actually bootstrap into it, and today the kit under-specifies setup in both directions: INSTANTIATE.md is addressed to a human and leaves the critical "wire in the one rule" step as a pointer, and there is no path at all for adopting PGK into an existing repo. This layer closes both, and neutralizes the single-operator language so enterprise/multi-operator use is a fill-in, not a fork.

Four components: (A) INSTANTIATE.md v2 — Claude-executed greenfield setup; (B) `bootstrap/` skeletons that make rule-wiring mechanical; (C) ADOPT.md — Claude-executed brownfield scan→infer→confirm→write; (D) role-language neutralization of Core.

## 2. Binding design principles

1. **Evidence-only inference.** In any Claude-executed procedure, every claim about the project cites its evidence (a file path, a commit hash) or is flagged `[unverified]`. No evidence, no claim.
2. **Never fabricate provenance.** Reconstructed history must be visibly reconstructed. A backfilled record that reads as contemporaneous is the exact failure PGK exists to prevent.
3. **Propose, don't commit.** Setup procedures hand the operator a filled draft to ratify. The procedure never pushes, and never commits without explicit operator go-ahead.
4. **Markdown only, no platform.** No scripts, no tooling, no skill bundles. The procedures are markdown addressed to a Claude session — paste-or-point on any surface.
5. **Governance, not scaffolding.** The setup layer wires governance into a project. It takes no position on repo structure, CI, stack, or tooling.
6. **Instance-first.** Ship the minimum that makes both paths work; let real instantiations teach elaborations.

## 3. Component A — INSTANTIATE.md v2 (greenfield, Claude-executed)

Rewrite INSTANTIATE.md as a procedure addressed to a Claude session, with a short operator-facing preamble ("point a Claude session at this file in your new repo and answer its questions"). The existing human-runnable checklist survives as a compact appendix ("Manual path") — content preserved, including the current "What NOT to add yet" section (updated per §6/§7 only).

Required phases, in order:

- **A0 — Preconditions.** Confirm working repo and where docs live (default `docs/`); confirm the operator wants governance here. If the target is not a git repo, pause and say so — PGK assumes git.
- **A1 — Place files** from `templates/` per the placement conventions currently in INSTANTIATE.md.
- **A2 — Fill what is verifiable** without asking: project name, paths, anything readable from the repo. Cite evidence per §2.1.
- **A3 — Interview the operator** for what cannot be inferred — at most 4 questions, batched: (i) the project's core premise / measurable success criterion (becomes ADR-001), (ii) decision owner(s) per §6, (iii) session roles in use (advisor/builder/single), (iv) authoritative docs that exist or are planned. Absence is a valid answer recorded honestly ("no authoritative docs yet; `git log` is the only authority").
- **A4 — Initialize the board honestly.** Every section filled with what is actually true; empty sections get explicit empty-state statements; freshness marker set.
- **A5 — Write ADR-001** from the interview answer, in the decision-log format.
- **A6 — Wire the one rule** using the `bootstrap/` skeletons (§4): place/merge the CLAUDE.md skeleton and offer the assistant-instructions text for whatever surface the operator's sessions bootstrap from. If a CLAUDE.md already exists, propose an additive merge — never overwrite.
- **A7 — Verify.** Zero unresolved `[PLACEHOLDER]` fields (or each remaining one explicitly deferred with reason); board sections all present in order; ADR-001 exists; the one rule is reachable from the session bootstrap path.
- **A8 — Hand off.** Plain-language summary of what was created and what was inferred vs. answered; then the exact named-path `git add`/commit for the operator to ratify. Stop.

## 4. Component B — `bootstrap/` skeletons

Two files, both placeholder-driven, both referenced by A6:

- **`bootstrap/CLAUDE-md-skeleton.md`** — a fill-in CLAUDE.md using the two-part pattern: PART 1 standing rules (stable; contains the board-read-first directive, role separation per the session-discipline template, and a pointer to the authority table) and PART 2 operational state (living; points to the state board rather than duplicating it). Kept minimal — PGK wiring only, no project scaffolding opinions.
- **`bootstrap/assistant-instructions.md`** — surface-agnostic instructions text for advisor-type sessions (claude.ai project instructions, system prompt, or equivalent): read the board before substantive work, update before ending, honesty rules (`[unverified]`, stale-worse-than-none), and where the board lives. Written to be pasted verbatim after placeholder fill.

## 5. Component C — ADOPT.md (brownfield, Claude-executed)

A procedure addressed to a Claude session, operator-facing preamble as in §3. Four phases, strictly ordered:

- **C1 — SCAN (read-only).** Survey: repo structure; README and any docs; `git log` shape (age, contributors, activity bursts, merge patterns); open branches and their staleness; PRs/issues if accessible; manifests/config for stack identification. Output: an observation list where every line carries evidence (path or commit hash). The scan modifies nothing.
- **C2 — INFER (proposals only).** From C1 produce four proposals, each explicitly marked as proposal: (i) authority-table candidates — which existing docs actually function as spec/plan/inventory, or the honest "none; `git log` is the only authority"; (ii) a drafted state board — current focus from recent commit clustering, work in flight from open branches/PRs, blocked/dormant candidates from stale branches; (iii) ADR candidates from decision fossils — framework/stack choices, architecture pivots visible as large refactors, abandoned directories, migrations. **The that/not-why rule is absolute:** evidence shows *that* a decision happened, never *why*; the why of every candidate is left blank for the operator; (iv) placement — where governance files fit the repo's existing conventions.
- **C3 — CONFIRM.** Operator reviews: corrects inferences, supplies the *why* for each ADR candidate they accept, rejects freely. Anything unresolved is flagged `[unverified]` or dropped — never guessed. Batched questions, minimal round-trips.
- **C4 — WRITE.** Place the files. **ADR-000 is mandatory and is the adoption line:** "PGK adopted at commit `<hash>` on `<date>`. Records before this line are reconstructed from evidence; records after are contemporaneous." Accepted ADR candidates are written with `**Status:** Accepted (reconstructed from evidence, [date])` and evidence-only Context sections. Backfill ONLY decisions that still actively bind future work; resist completeness. Then wire the rule (A6 equivalent), verify (A7 equivalent), and hand off (A8 equivalent — summary + ratification commit, stop).

Hard rules restated inside ADOPT.md itself: the scan commits nothing; no invented whys; evidence citations mandatory; the procedure never pushes.

## 6. Component D — role-language neutralization (Core templates + README)

- In `templates/state-board.md`, `templates/decision-log.md`, `templates/session-start.md`: replace role-singular "operator" with `[DECISION_OWNER]` where it names the decision-gating authority, defined once per file on first use: *"the person or role who gates decisions for this scope; in a single-operator project, that's you."* Where "operator" means "whoever is running the session" generically, prefer neutral phrasing.
- `templates/state-board.md`: the Open decisions section gains an owner column or note so each decision names its `[DECISION_OWNER]` (multi-operator projects route decisions; single-operator projects have one value everywhere).
- `templates/session-start.md` role-separation section: add a short multi-operator note — with multiple humans, designate a board owner; decision entries carry owners; the separation logic is unchanged.
- **Semantics are frozen.** This is renaming and parameterizing, not redesigning: no extracted discipline rule changes meaning. Single-operator remains the fully-supported default (a special case where every owner placeholder holds one value).
- README: one sentence noting multi-session works across multiple operators and pointing at the placeholders.

## 7. Documentation updates

- README "What's in the kit" table gains rows for `ADOPT.md` and `bootstrap/`; "How to use it" describes the two paths (new project → INSTANTIATE.md; existing repo → ADOPT.md), both runnable by pointing a Claude session at the file.
- INSTANTIATE.md v2 and ADOPT.md cross-reference each other ("existing repo? use ADOPT.md" / "empty repo? use INSTANTIATE.md").
- All cross-references in the repo resolve to files that exist.

## 8. Verification requirements (feed the dispatch's success criteria)

1. **Live greenfield run:** execute the INSTANTIATE.md v2 procedure against a throwaway scratch repo, simulating the operator with the fixture answers in Appendix A. The resulting instance must have: zero unresolved placeholders, all board sections present in template order with explicit empty-states, ADR-001 present in format, freshness marker set, the one rule reachable from the bootstrap files.
2. **Live brownfield scan:** execute ADOPT.md phases C1–C2 only (read-only) against a real repo with history (`~/geo-machine-jancal`, strictly read-only). The proposal output must contain zero uncited claims and zero filled-in whys.
3. **Neutralization check:** in the three Core templates, every remaining "operator" is either generic-session usage or inside the `[DECISION_OWNER]` definition; a diff review confirms no discipline rule changed meaning.
4. **Markdown-only check:** the PGK tree contains no scripts, binaries, or tooling after the change.

## 9. Out of scope

Skill/bundle packaging; any change to `automation-tier/` (the verbatim copies are frozen); enterprise multi-board or cross-team features beyond §6's placeholders; project scaffolding of any kind; automation of the setup procedures themselves.

## Appendix A — greenfield fixture (operator answers for verification run §8.1)

- Project: `pgk-fixture-demo` (scratch repo created under `/tmp`, deleted after verification)
- Core premise (ADR-001): "Ship a working demo CLI in 30 days; success = a stranger can install and run it from the README alone."
- Decision owner: single operator, "Demo Operator"
- Roles: advisor + builder split
- Authoritative docs: none yet — `git log` only

## Version history

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-07-22 | Initial spec. Ratified in Cowork session; scope calls confirmed by operator (include neutralization; markdown-only packaging). |
