<!--
PGK AUTOMATION TIER — VERBATIM REFERENCE COPY (GEO Machine implementation)
Source file:    docs/governance/traceability-check-skill-spec.md
Source version: v1.7 (2026-05-26)
Source repo:    mames98/GEO-Machine, HEAD 4579313 (45793134dcaf1d1e8e243dcdab0f9ae3cc7a92c4, 2026-06-12)
Copied:         2026-07-11 (PGK automation-tier codification)
Everything below this comment block is byte-faithful to the source file.
Generalization map: ../generalize/traceability-check-skill-spec-notes.md
-->

# Traceability Check Skill — Specification

**Type:** Cowork skill specification
**Version:** 1.7
**Status:** Current
**Last revised:** 2026-05-26 (v1.7 amendment — adds §5.8 Auto-remediation tier classification rubric; §6.1 pattern catalog gains "Default AR tier" column; §6.2 finding format block gains `Remediation tier:` field (mandatory-only tier) with mandatory/optional edit labeling; §6.4 bundle proposal gains "Bundle AR tier" + "Full-bundle tier" fields. Prerequisite for auto-remediation cron task (Auto-Remediation Spec v1.0) which reads §12 finding tiers to determine execution behavior without re-classifying at runtime. Prior: 2026-05-26 (v1.6 amendment — §5.7 gains Step 4: Discord notification on scheduled-run completion via existing `DISCORD_WEBHOOK_COMMANDS` channel; no new env var; helper at `pipelines/shared/audit_notify.py`))
**Codex reference:** §3.1 (Document hierarchy and trace pattern), §3.2 (Sub-spec standard), §3.4 (State-tracking)

---

## 1. Codex reference

This spec implements Codex §3.1 — the operational enforcement of the document hierarchy and trace pattern. Codex §3.1 names the trace pattern (Strategic Narrative → Codex → Sub-specs → Backlog → Roadmap → CC dispatches) and identifies the Cowork traceability check skill as the mechanism that makes trace pattern compliance verifiable rather than aspirational.

Secondary references:
- Codex §3.2 — this spec conforms to the §3.2 sub-spec standard structure
- Codex §3.4 — skill outputs feed §3.4 state-tracking discipline
- Codex §3.5 — skill execution itself follows §3.5 verification cadence principle (the skill produces evidence; the operator decides when to invoke or schedule)

---

## 2. Scope

The Traceability Check Skill is a Cowork capability — not an agent, not a continuous-runtime component — that audits the project's governance documents and shipped artifacts against Codex §3.1 trace pattern requirements. The skill is invoked on operator instruction or operator-defined schedule; it reads the relevant governance documents and produces a report classifying any trace pattern violations or drift. Starting with the v1.1 amendment (2026-05-18), the skill also drafts remediation for drift-class findings matching established mechanical-fix patterns; violations and ambiguity findings remain operator-decision.

Specifically, the skill performs six checks:

**Check 1 — Roadmap stages reference valid backlog entries.**
Every stage in the canonical roadmap should reference one or more backlog entries that compose the stage. Each referenced backlog entry must exist in the backlog. The check verifies that no roadmap stage points at a backlog entry that doesn't exist or has been removed without the roadmap reference being updated.

**Check 2 — Backlog entries reference valid Codex sections or sub-specs.**
Every backlog entry should reference the Codex section or sub-spec that authorizes the work. The check verifies that each backlog entry's Codex/sub-spec reference resolves to a valid §N anchor or a valid sub-spec path. Backlog entries without such a reference are flagged as untraced work.

**Check 3 — Sub-spec status matches implementation reality (sampled).**
Every sub-spec listed in Codex §4.1 has a declared status (current / partial / planned / deprecated / superseded / draft). The check samples a subset of sub-specs each run and verifies the declared status against observable implementation evidence (does the implementation exist where the sub-spec says it should; does the implementation conform to the sub-spec's described behavior at the surface level). Sampling rather than exhaustive review keeps the skill's runtime bounded; over multiple runs, all sub-specs eventually get checked.

**Check 4 — Shipped artifacts identified in CLAUDE.md PART 2 trace upward.**
Every shipped artifact mentioned in CLAUDE.md PART 2 Operational State (or its session log entries) should trace to a Codex section, sub-spec, and backlog entry. The check verifies that shipped artifacts have complete trace paths. Artifacts shipped without trace paths are flagged as governance debt.

**Check 5 — Cross-branch governance divergence.**
For every local and remote branch ahead of `main`, the check determines whether the branch carries code or specs that `main`'s governance (Codex §4.1, §2.3, CLAUDE.md PART 2 session log) describes as shipped or Current. The failure mode this catches: governance promoted to `main` while the implementation it describes lives only on an unmerged feature branch — the "main-only-audit blind spot" surfaced by the 2026-05-15 and 2026-05-23 ad-hoc branch audits. Cross-branch divergence is distinct from Violation: the referenced code *exists*, just not on `main`. Classified per the branch-divergence class in §5.5.

**Check 6 — Ratified-spec git-tracking sweep.**
Every spec file present on disk under `docs/specs/` and `docs/governance/` whose self-header declares a ratified/current status must be git-tracked (`git ls-files` returns it). The failure mode this catches: a ratified spec placed on disk but never `git add`-ed — operationally authoritative yet invisible to every git-based audit and to other clones. Precedent: the Cross-Host Coordination Mechanism Spec was untracked from authoring (2026-05-19) until 2026-05-24 despite ratifying Codex §2.6.

**Cadence note.** Checks 1–4 and Check 6 are lightweight (document + `git ls-files` reads) and run on every invocation. **Check 5 (cross-branch) is heavier** (`git log` + file-diff across all branches ahead of `main`) and runs on a **separate cadence: per-merge-to-main and on operator request, not every daily run.** A run that includes Check 5 is a "full + cross-branch" run; runs without it are "standard." The report states which mode the run was.

---

## 3. Out-of-scope

The skill explicitly does not do the following:

- **The skill does not fix violations.** It surfaces findings. Operator decides what to act on. Auto-fixing trace violations would either patch governance documents (risking drift in the opposite direction) or revert shipped artifacts (destructive). Both are wrong actions for an audit skill. (Note: per §6 added in v1.1, the skill may *draft* remediation for drift-class findings matching established mechanical-fix patterns. Drafting is not fixing; the operator gate in §6.5 is canonical authorization.)
- **The skill is not a continuous-runtime agent.** It runs when invoked. It does not watch for governance changes in real-time. The §3.4 state-tracking discipline governs cadence; the skill is invoked according to that cadence.
- **The skill does not enforce sub-spec content quality.** It checks that sub-specs exist where Codex §4.1 says they do and that the declared status matches reality at the surface level. Deep sub-spec quality review (is the spec content correct, is the §3.2 standard structure followed) is operator and Cowork work, not skill work.
- **The skill does not validate Codex content.** The Codex is the authority the skill checks against. Validating the Codex itself requires strategic review, not mechanical traceability checking.
- **The skill does not check operational state correctness.** Whether the operational claims in CLAUDE.md PART 2 are true is the work of the verification audits per Codex §3.4 — a different and broader scope. The traceability skill checks that shipped artifacts have trace paths, not whether the artifacts behave as claimed.
- **The skill does not cover Strategic Narrative or audit reports.** Strategic Narrative changes via its own protocol and doesn't have downstream trace requirements in the same shape. Audit reports are inputs to the skill (and history), not subjects of trace checking.
- **Check 5 does not assess merge-readiness or recommend merges.** It surfaces that a divergence exists (main governance ahead of main code, implementation on a tracked branch). Whether and when to merge the branch is operator strategic decision, not skill output.

---

## 4. Interfaces

### Inputs

The skill reads (read-only) the following governance documents and artifacts on each invocation:

| Input | Path | Purpose |
|---|---|---|
| Codex v2.0 | `docs/strategy/geo-codex-v2.0.md` | Source of truth for §N anchors, §4.1 sub-spec index, §4.2 governed document registry |
| v2.0 Realization Plan | `docs/strategy/v2.0-realization-plan.md` | Phase-level reference for what work each phase authorizes |
| Roadmap | `docs/strategy/v2.0-aligned-roadmap.md` | Sequenced execution stages; Check 1 input |
| Backlog | `docs/operations/backlog.md` | Work inventory; Check 1 and Check 2 input |
| CLAUDE.md | Repo root | PART 1 governance facts (Strategic Authority Hierarchy, Tool Registry); PART 2 operational state including session log; Check 4 input |
| Sub-specs | `docs/specs/`, `docs/architecture/`, `docs/governance/`, `docs/principles/`, `docs/editorial/`, `docs/frameworks/`, `docs/videographer/` | Check 3 input; verified against Codex §4.1 declarations |
| Audit history | `docs/audits/` | Historical reference; helps the skill distinguish between current state and known prior states |
| Branch graph | git refs (all local + remote branches ahead of `main`) | Check 5 input — branch-ahead detection + per-branch file inventory vs `main` (full + cross-branch runs only) |
| Git tracking index | `git ls-files` over `docs/specs/` + `docs/governance/` | Check 6 input — confirms ratified specs on disk are tracked |

### Outputs

The skill produces one output per invocation:

| Output | Path | Format |
|---|---|---|
| Traceability audit report | `docs/audits/YYYY-MM-DD-traceability-audit.md` | Markdown |

Report structure (required sections):

1. **Summary statistics.** Total checks performed (per check), total findings (per classification), comparison to most recent prior traceability audit if one exists.
2. **Check 1 findings** — roadmap stages with broken or stale backlog references. Each finding includes: stage location, referenced backlog entry, evidence of mismatch, recommended action.
3. **Check 2 findings** — backlog entries without valid Codex/sub-spec references. Each finding: entry location, current reference (or absence), evidence, recommended action.
4. **Check 3 findings** — sub-spec status mismatches. Each finding: sub-spec path, declared status in Codex §4.1, observed implementation state, recommended action.
5. **Check 4 findings** — shipped artifacts without complete trace paths. Each finding: artifact identity, trace gaps, recommended action.
6. **Check 5 findings** (full + cross-branch runs only) — branches ahead of `main` carrying code/specs that `main` governance describes as shipped or Current. Each finding: branch name, tip SHA, commits/files unique to the branch, the `main` governance claim that describes the divergent work, recommended action. Standard runs state this section "not run this invocation (standard run; Check 5 fires per-merge + on request)."
7. **Check 6 findings** — ratified specs on disk but not git-tracked. Each finding: file path, self-header status, `git ls-files` result, recommended action (`git add` + commit).
8. **Findings classified by severity:**
 - **Violation** — broken trace (a reference points at something that doesn't exist on any branch and isn't on disk)
 - **Drift** — trace exists but stale (the referent has changed in a way that invalidates the trace)
 - **Branch-divergence** — `main` governance describes code/specs as shipped/Current that exist only on a tracked branch ahead of `main` (Check 5). Distinct from Violation: the referent exists and is merge-recoverable. See §5.5.
 - **Ambiguity** — operator decision needed (the trace is plausible but unclear; resolution requires judgment Cowork shouldn't make autonomously)
9. **Sampling note for Check 3** — which sub-specs were sampled this run, which were skipped, plan for future-run coverage.
10. **Carry-forward items** — findings flagged in a prior audit that remain unresolved. Tracks whether the trace pattern is stable, improving, or degrading over time.
11. **Synthesis** — operator-facing interpretation of the run's findings: stability/improvement/degradation trend, notable patterns across findings, recommended audit cadence adjustment if applicable.
12. **Remediation drafts** (when applicable; see §6) — drafted edits for drift-class findings matching established mechanical-fix patterns, plus a consolidated bundle proposal for operator review.

### Interface contracts

- The skill reads only; it never modifies governance documents.
- The skill commits exactly one new file per invocation (the report). Single explicit named-path commit.
- The skill does not require access to CC dispatches, code, or running systems. It operates entirely on documents in the repo.

---

## 5. Success criteria

The skill is implemented correctly when all of the following hold:

1. **Runtime.** Standard runs (Checks 1–4 + Check 6) complete in under 15 minutes of Cowork-session time end-to-end (including report generation and commit). Full + cross-branch runs (adding Check 5) complete in under 30 minutes. Exceeding these caps suggests the skill has expanded scope beyond what this spec authorizes.
2. **Coverage.** The skill exercises Checks 1–4 + Check 6 on every run, and Check 5 on full + cross-branch runs (per the §2 cadence note). No check is skipped without explicit reason (Check 3's sampling is intentional, not a skip; Check 5's standard-run absence is the defined cadence, not a skip).
3. **Output completeness.** The report contains all required sections per §4 Outputs structure. Empty sections are stated as empty rather than omitted.
4. **Accuracy.** Findings classification (violation vs drift vs ambiguity) is operator-validated on the first run after skill implementation. Subsequent runs maintain consistent classification logic.
5. **Drift detection.** Subsequent runs successfully detect new trace violations introduced since the previous run. The skill is useful in proportion to how reliably it catches drift early.
6. **Operator usefulness.** The report is actionable. Each finding includes a recommended action specific enough that the operator can decide what to do without re-investigating the underlying state.
7. **Remediation drafts (when applicable).** Drift findings matching §6.1 patterns are drafted with exact-text edits per §6.2 format. Bundle proposal includes commit message + version classification per Codex §3.8. Stop conditions properly identified per §6.3.

### 5.5 Three-tier acceptance criteria for phase-closure gates

Audit runs surface findings across a spectrum from blocking violations to background housekeeping. Not all findings gate phase closure. This section defines the three tiers, which findings map to each tier, and which tier gates closure.

**Tier 1 — Blockers.** Findings in this tier must be resolved before the phase may close.

A finding is Tier 1 if ANY of the following hold:

- It is classified **Violation** (broken trace — a reference points at something that does not exist).
- It creates a documented gap where production agents cannot operate correctly or where strategic decisions would be made on materially incorrect information.
- Operator explicitly escalates it to Tier 1 during a prior session.

*Gate rule: Phase closure is blocked until all Tier 1 findings have received a closing commit or an explicit operator waiver with rationale.*

**Tier 2 — Working set.** Findings in this tier are tracked openly and resolved within a bounded audit horizon (default: 3 audit runs from first appearance). They do not block phase closure.

A finding is Tier 2 if ALL of the following hold:

- It is classified **Drift** (trace stale or inconsistent) OR **Ambiguity** (operator judgment needed, but no urgency signal).
- It does not meet any Tier 1 criterion.
- It has a clear remediation path (e.g., eligible for §6 remediation drafting, or awaits an operator decision that has been surfaced).

*Gate rule: Tier 2 findings are summarized in the closure declaration — "N Tier 2 items remain open; carried forward to next audit run." Phase closes without waiting for resolution.*

**Tier 3 — Background debt.** Findings in this tier are acknowledged and scheduled for future cleanup. They do not block phase closure and do not appear as carry-forward items in the Tier 2 working set.

A finding is Tier 3 if:

- It is classified Drift but the referent is historical context only (e.g., a changelog entry that records an old version number).
- Fixing it would be purely cosmetic with no downstream impact on traceability or operational correctness.
- Operator designates it Tier 3 on first appearance.

*Gate rule: Tier 3 findings are listed in a dedicated "Background debt" section of the audit report. They accumulate until an operator-scheduled cleanup pass; they do not appear in carry-forward counts.*

**Branch-divergence (Check 5) — a classification distinct from Violation.** `main` governance (Codex §4.1, §2.3, CLAUDE.md PART 2) describes code or specs as shipped or Current, but the implementation lives only on a tracked branch ahead of `main` — not on `main`, not untracked, not missing. This is distinct from Violation: the referent exists and is recoverable via merge; an in-flight feature branch is a normal development state, not a broken trace. **Default tier: Tier 2** (working set — surfaced and tracked, does not block phase closure). It escalates to Tier 1 only if the operator determines the divergence is causing active incorrect decisions (e.g., a fresh session dispatched work assuming code was on `main` when it wasn't). *Contrast with two genuine Tier-1 cases:* (a) a governance reference to code that exists on **no** branch and is **not** on disk remains a **Violation → Tier 1**; (b) the git-tracking gap from Check 6 (a ratified spec present on disk but **untracked**) is also **Violation → Tier 1** — an untracked ratified spec is a broken durability guarantee (invisible to git, lost on a clean clone), not mere branch divergence.

**Tier assignment defaults:**

| Finding classification | Default tier |
|---|---|
| Violation | Tier 1 |
| Drift — eligible for §6 mechanical fix | Tier 2 (unless Tier 1 criteria met) |
| Drift — requires operator decision | Tier 2 |
| Drift — historical context / cosmetic | Tier 3 |
| Ambiguity | Tier 2 |
| Branch-divergence (Check 5) | Tier 2 (escalates to Tier 1 only on operator determination of active incorrect decisions) |
| Untracked ratified spec (Check 6) | Tier 1 (broken durability guarantee) |

The auditor applies default tier assignments; the operator may promote or demote any finding during review. Tier changes are recorded in the audit report and carry forward to subsequent runs.

**Phase closure declaration format (amended per this section):**

A phase closure declaration must include:

1. Confirmation that all Tier 1 findings are resolved or waived (with waiver rationale).
2. Count and summary of open Tier 2 items (carried forward).
3. Count and summary of open Tier 3 items (deferred to cleanup pass).

Absence of Tier 1 findings (or operator-waived Tier 1 findings with documented rationale) is the closure gate. The presence of Tier 2 or Tier 3 findings is acknowledged, not blocking.

**Rationale (2026-05-20):** Phase B closure surfaced a gap in the acceptance criteria: sampling-driven audit cycles produce new drift findings indefinitely. Without explicit tier classification, every new finding — no matter how mechanical or cosmetic — potentially re-opens the closure gate. This creates a runaway gate problem where governance improvement work consumes more time than the substantive work it governs. The three-tier model resolves this: operators close phases on substantive correctness (Tier 1), not perfect governance hygiene (Tier 2 and 3). Full reference: operator decision 2026-05-20 (Phase B formal closure dispatch, Pending Dispatch 3 note).

### 5.6 Output placement

**Cadence (descriptive).** Scheduled invocations currently fire twice daily (approximately 06:00 and 12:00 Mac Mini local time) per the existing scheduled-task pattern in the Claude.ai task history. This is recorded for context, not as a spec requirement — the schedule is owned by the runner configuration, not by this skill spec.

**Default output path.** Audit reports are written to `docs/audits/<YYYY-MM-DD>-traceability-audit-run<N>.md` for both scheduled and manual invocations. The `<YYYY-MM-DD>` is the date the audit ran; `<N>` is the run number.

**Run-number determination.** Before writing, the skill scans `docs/audits/` for files matching the pattern `*-traceability-audit-run*.md`, extracts the integer `N` from each filename, and uses `max(N) + 1` for the new run. If no matches are found, the skill starts at the next number after the highest known historical run (Run 6 ships before this section's amendment, so a fresh execution under v1.5+ produces Run 7 at the earliest).

**Filename convention.** The `-run<N>` suffix is mandatory for all runs authored under v1.5+. If two scheduled runs fire on the same calendar date, the second uses `run<N+1>` — same date, incremented number. Historical audit files predating v1.5 that lack the `-run<N>` suffix remain in place as historical artifacts; the run-number scan tolerates their absence and continues counting from the highest suffixed value it finds.

**Manual-run `outputs/` option.** Manual runs default to `docs/audits/` same as scheduled runs. If the operator wants the legacy `outputs/` → review → promote workflow for a specific manual run, they instruct so in the invocation prompt; the skill honors operator instruction and writes to `outputs/<YYYY-MM-DD>-traceability-audit-run<N>.md` instead. No flag-parsing surface is assumed.

### 5.7 Scheduled-run auto-commit

After writing the audit file (per §5.6), scheduled invocations execute:

```
git add docs/audits/<filename>
git commit -m "audit: traceability Run <N> (scheduled) — <total> findings / <violations> violations / <drift> drift / <branch-div> branch-div / <ambiguity> ambiguity"
git push
```

The commit message format is mandatory. The five headline counts come from the §1 Summary statistics table in the audit report.

**Halt-on-failure.** If any git operation fails (lock contention, push rejection, merge conflict, authentication failure, etc.), the skill halts after writing the file, surfaces the git error in the run summary, and exits non-zero. The audit file remains on disk for operator recovery. The skill does not retry inside the invocation; recovery is operator-initiated.

**Manual runs do not auto-commit.** Manual invocations write the file (to `docs/audits/` by default, or to `outputs/` per §5.6 operator instruction) but do not invoke git, regardless of output path. Commit and push for manual runs is an explicit operator action — the audit file may be staged, modified, or discarded at operator discretion.

#### Step 4 — Discord notification

After `git push` completes (Step 3), the skill invokes:

```
python -m pipelines.shared.audit_notify docs/audits/<filename>
```

The helper parses the §1 Summary statistics totals row, constructs a GitHub blob URL on `main`, and posts the following message to Discord `#commands` via the existing `DISCORD_WEBHOOK_COMMANDS` channel:

```
{emoji} Daily audit Run {N} complete. {V} violations / {D} drift / {B} branch-div / {A} ambiguity ({T} total findings). {url}
```

**Severity emoji:** 🟢 when violations == 0; 🔴 when violations > 0.

**Post-then-exit-cleanly contract.** The helper exits non-zero on Discord failure (5xx, webhook misconfiguration, network error) but does not mask the audit or the git push that preceded it. The audit file is on disk, the push succeeded; the notification is mobile glue, not a gate. The skill surfaces the non-zero exit in its run summary but does not retroactively halt or invalidate Steps 1–3.

**Dry-run behavior.** When `DISCORD_WEBHOOK_COMMANDS` is empty or unset, the helper prints the formatted message to stdout and exits 0 — matching the existing `discord_notify.send_alert()` and `roadmap_ack.py` dry-run patterns. No operator action is required if `DISCORD_WEBHOOK_COMMANDS` is already configured; the helper reuses the existing webhook and channel.

**Manual runs do not invoke Step 4.** Manual runs do not execute git (Step 3 is scheduled-only) — and therefore do not proceed to Step 4. Notification of manual-run audit completion is at operator discretion.

**Rationale.** Auto-commit on scheduled runs matches the existing autonomous-pipeline patterns (Scout citation audit, FEMA ingest) where the operator does not review each cycle individually. The operator-review gate that motivated the `outputs/` workflow for scheduled runs is not load-bearing — the operator does not read each scheduled audit at write time. Reclassification of findings (if needed) happens via follow-on operator-initiated edits, not via a placement-blocking gate. Without Step 4, a successful scheduled audit produces a commit on `main` with no mobile-visible signal; the operator only sees results if they check git history, typically 12–24 hours later. Step 4 closes the visibility gap: zero-violation / violation status appears in Discord `#commands` within seconds of the push.

### 5.8 Auto-remediation tier classification

Auto-remediation tier is the classification assigned to each §12 Remediation draft entry to signal how the Auto-Remediation Spec v1.0 execution cycle handles the finding. It is **distinct from the §5.5 phase-closure tier** — a finding carries both classifications independently.

**Vocabulary.**

| Term | Where defined | What it governs |
|---|---|---|
| Phase-closure tier | §5.5 | Whether a finding blocks phase close (Tier 1 = blocker, Tier 2 = carry, Tier 3 = background debt) |
| Auto-remediation tier / Remediation tier | This section | How the auto-remediation cron task handles the finding (Tier 1 = auto-execute, Tier 2 = execute + flag, Tier 3 = halt) |

Both use a 1/2/3 scale; the scales are independent. A §5.5-Tier-2 working-set finding typically maps to AR Tier 1 or Tier 2 when it matches an established §6.1 pattern. A §5.5-Tier-1 violation always maps to AR Tier 3. A §5.5-Tier-3 cosmetic drift finding typically maps to AR Tier 3 (it won't have a §6.1 pattern match). For full execution behavior per AR tier — commit authority, Discord digest format, quarantine branch behavior, halt conditions — see **Auto-Remediation Spec v1.0 §3** (`docs/specs/auto-remediation-spec-v1.0.md`).

**When the auditor assigns AR tier.** AR tier appears in the `Remediation tier:` line in each §12 finding block per §6.2 format. For Tier 3 findings (§6.3 stop conditions), the `Remediation tier: 3` annotation appears in the §12 "Findings deferred to operator/Cowork decision" section, not in a full §6.2 draft block (since no draft is produced).

**Decision tree (apply in order; first match wins).**

**Step 1 — Check for AR Tier 3 triggers.** Any one of the following is sufficient:

| Condition | Rationale |
|---|---|
| Finding classification is **violation** | Broken trace; auto-remediation does not fix violations |
| Finding classification is **ambiguity** | Requires operator judgment; no mechanical fix |
| Finding classification is **drift** but **no §6.1 pattern matched** | Pattern not pre-ratified; improvisation prohibited |
| Finding classification is **drift** + §6.1 pattern matched but a **§6.3 stop condition fired** | Auditor flagged: ambiguous targets, cascade, or substantive governance section |
| Finding cites a **recent operator decision** (within trailing 7 days) that **contradicts the proposed remediation** | Operator signal takes precedence |
| Finding is a **Check 5 branch-divergence** finding | Merge decisions are strategic; explicitly non-draftable per §6.1 |

→ If any Step 1 condition holds: **AR Tier 3.**

**Step 2 — Check for AR Tier 2 awareness conditions.** (Finding has §6.1 pattern, no §6.3 stop, no Step 1 trigger.) Any one of the following is sufficient:

| Condition | Rationale |
|---|---|
| Edit touches a **Codex section anchor** (§N.M label change, not text-internal-to-section) | Section anchor changes affect downstream references |
| Edit propagates change across **two or more files** | Multi-file edits have higher blast radius if pattern-match is wrong |
| Edit **establishes a versioned artifact** (version-history table row, registration entry) that downstream sub-specs may reference | Versioned artifacts can propagate label expectations |

→ If any Step 2 condition holds: **AR Tier 2.**

**Step 3 — Default.** Finding passed Steps 1 and 2 without triggering any condition. Single-file, pre-ratified pattern, no stop conditions, no cascade, no anchor changes, no versioned artifacts.

→ **AR Tier 1.**

**Summary table:**

| Condition | AR tier |
|---|---|
| Violation | Tier 3 |
| Ambiguity | Tier 3 |
| Drift, no §6.1 pattern | Tier 3 |
| Drift, §6.1 pattern, §6.3 stop condition | Tier 3 |
| Drift, §6.1 pattern, trailing-7-day operator-decision conflict | Tier 3 |
| Check 5 branch-divergence | Tier 3 |
| Drift, §6.1 pattern matched, no stop — edit spans ≥2 files | Tier 2 |
| Drift, §6.1 pattern matched, no stop — establishes versioned artifact | Tier 2 |
| Drift, §6.1 pattern matched, no stop — touches Codex section anchor | Tier 2 |
| Drift, §6.1 pattern matched, no stop — single file, no awareness conditions | Tier 1 |

**Sanity check against Run 8 findings:**

| Finding | §5.5 tier | §6.1 pattern | Stop conditions | Awareness | AR tier |
|---|---|---|---|---|---|
| 2.1 — Backlog archived-roadmap path | Tier 3 | None | — | — | **3** (no pattern) |
| 3.1 — Faithfulness Check row intro placeholder | Tier 2 | §4.1 row text update | None | Single file, no versioned artifact | **1** |
| 4.1 — CLAUDE.md PART 2 banner stale | Tier 2 | None (§6.3 stop: substantive Cowork-maintained surface) | §6.3 | — | **3** (stop condition) |
| 4.2 — Orchestrator row label v1.3→v1.4 (label only) | Tier 2 | §4.1 row text update | None | Single file | **1** |
| 4.2 — Orchestrator row label + v2.1.7 patch bundle | Tier 2 | §4.1 row text update | None | Establishes version-history row | **2** |

Finding 4.2 illustrates that AR tier is per-instance, not per-pattern. The row-label swap alone is AR Tier 1. Including the v2.1.7 version-history row in the same edit establishes a versioned artifact → AR Tier 2. The §12 bundle proposal should note which bundle shape was selected so the auto-remediation cron task reads the correct tier.

**Multi-edit findings: mandatory vs optional edits.**

When a §12 finding contains both mandatory and optional edits (e.g., the mandatory row-label swap + the optional version-history row for Finding 4.2), the Remediation tier is determined as follows:

- **`Remediation tier:` in §6.2 always records the mandatory-only tier** — the result of running the §5.8 decision tree against the mandatory edits alone. This is what the auto-remediation cron task reads and acts on.
- **Optional edits are labeled `Optional:` in the §6.2 edit block** and are never auto-executed by the cron task regardless of their tier. The cron task treats any edit marked `Optional:` as out of scope.
- **If the operator chooses to include optional edits in a manual dispatch**, the effective tier for that dispatch is the maximum across all included edits (mandatory + optional). This is the "full-bundle tier" and is surfaced in the §6.4 bundle proposal's `Bundle AR tier:` field when optional edits exist, not in the `Remediation tier:` field.

This rule gives the cron task a single deterministic value to read (`Remediation tier:`) while preserving the operator's ability to include optional edits — which may raise the tier and require operator-dispatched execution rather than auto-execution.

**Verification against Finding 4.2:**

| Bundle shape | `Remediation tier:` field | Auto-executed? | `Bundle AR tier:` in §6.4 |
|---|---|---|---|
| Mandatory only (row-label swap) | 1 | Yes — cron auto-executes | 1 |
| Mandatory + optional (row-label + v2.1.7 history row) | 1 (unchanged — mandatory-only tier) | Mandatory edit auto-executed; optional edit operator-dispatched | 2 |

---

## 6. Remediation drafting for drift-class findings

After completing the audit phase (§§1–5), the skill enters a remediation-drafting phase for drift-class findings with established mechanical-fix patterns.

### 6.1 Eligible finding classes

A finding is eligible for remediation drafting if BOTH conditions hold:

(a) **Classification: Drift.** Per §5.4 the auditor classified the finding as drift (trace exists but stale or inconsistent). Violations (trace broken) and ambiguity (trace unclear) remain operator-decision and do not enter remediation drafting.

(b) **Established mechanical-fix pattern.** The finding's remediation follows a pattern previously applied and ratified in a Codex patch or governance document. The skill recognizes the following established patterns:

| Pattern | Example | Where established | Default AR tier |
|---|---|---|---|
| Sub-spec self-header Codex version label sync | "Sub-spec governed by GEO Codex v2.0.4 → v2.1.0" | Codex v2.1.0 patch (Phase B Session 4.5) | Tier 1 |
| Roadmap/backlog header Codex version reference update | "Codex authority: v2.0.1 → v2.1.0" | Same | Tier 1 |
| Codex §4.1 row path correction (file-vs-directory drift) | "docs/videographer/ → docs/specs/videographer-agent-spec-v1.0.md" | Codex v2.0.1 patch (Run-1 Finding 3.2 closure) | Tier 1 |
| Codex §4.2 row path correction (path-drift class) | "docs/governance/X.md → docs/brand/X.md" | Codex v2.0.4 patch (Phase B Session 5 Issue 2) | Tier 1 |
| Codex §4.1 row text update for shipped sub-spec | "Planned → Current with canonical path" | Codex v2.0.3 / v2.0.4 / v2.0.6 patches | Tier 1 |
| Untracked ratified spec → git-track (Check 6) | "git add docs/specs/X-v1.0.md" | v1.4 amendment (Check 6) | Tier 1 |

> **Default AR tier applies when the finding instance has no §5.8 Step 2 awareness conditions.** Specific instances may be promoted to AR Tier 2 by the §5.8 decision tree — for example, a "Codex §4.1 row text update" that also establishes a version-history row in the same bundle is AR Tier 2 (awareness condition: establishes versioned artifact). The Default AR tier is the baseline for the pattern applied in isolation.
>
> **New patterns added via future amendments must declare their default AR tier explicitly** in the pattern definition row. Pre-ratification, a new pattern is not AR Tier 1 eligible regardless of confidence — see §5.8 Step 1 ("no §6.1 pattern matched → Tier 3").

These are all the patterns currently eligible for AR Tier 1 (and potentially Tier 2 if awareness conditions apply per §5.8). All six patterns are single-file mechanical edits in their canonical form.

Patterns may be added to this table only via amendment to this skill spec; the auditor does not improvise new mechanical-fix patterns during execution.

**Check 5 (branch-divergence) findings are NOT mechanically draftable.** Resolving a divergence means merging a branch (or deciding not to) — a strategic decision per §3 (out-of-scope: "Check 5 does not assess merge-readiness or recommend merges"). Check 5 findings always hit the §6.3 stop condition (substantive/strategic) and surface for operator decision; the skill does not draft merge actions. Check 6 findings, by contrast, are mechanical (a single `git add`) and draftable per the pattern above.

### 6.2 Drafting requirements

For each eligible drift finding, the remediation drafting phase produces:

- **Exact-text edit anchors.** Before-and-after text for each edit, suitable for direct application via str_replace or equivalent.
- **File path.** Canonical path of each affected file.
- **Verification check.** What the auditor would verify after the fix to confirm closure (e.g., grep pattern returning expected count).

Drafting must include the pattern-match rationale explicitly. The format:

```
Finding [N]: [Drift classification]
Pattern matched: [pattern from §6.1 table]
Files affected: [paths]
Edits required: [count] ([M] mandatory + [N] optional)
Remediation tier: [1 / 2 / 3] (mandatory edits only, per §5.8 — this is what auto-remediation executes)

Edit 1 (mandatory):
  File: [path]
  Before: [exact text]
  After: [exact text]

Edit 2 (optional — auto-remediation does not apply; operator dispatch required):
  File: [path]
  Before: [exact text]
  After: [exact text]

Verification: [exact check] (covers mandatory edits only)
```

**Notes on the format:**

- **`Remediation tier:` always reflects the mandatory-only tier.** This is the deterministic value the auto-remediation cron task reads. The cron task applies only mandatory edits and ignores any `(optional)` labeled edits.
- **`Edits required:` lists mandatory + optional counts separately** so the auditor and operator can see at a glance what the cron task will apply vs. what requires operator dispatch.
- **Optional edits are labeled `(optional — auto-remediation does not apply; operator dispatch required)`** in the edit block header. This label is the cron task's sentinel — any edit block labeled `optional` is skipped.
- **Findings with no optional edits** omit the `(M mandatory + N optional)` count note and simply state `Edits required: [count]`. Optional edit labeling is only needed when both mandatory and optional edits exist.

> **Remediation tier for deferred findings.** Findings that reach §6.3 stop conditions do not receive a full §6.2 draft block — they are listed in the §12 "Findings deferred to operator/Cowork decision" section. The auditor annotates each deferred finding with `Remediation tier: 3` and the specific §5.8 Step 1 condition that triggered it, so the auto-remediation cron task can parse the deferred section and confirm the Tier 3 classification without ambiguity.
>
> **Bundle proposal AR tier.** The §6.4 "Bundle proposal" includes the aggregate AR tier for the bundle: the highest `Remediation tier:` value across all findings' mandatory edits. When optional edits exist for any finding, the bundle proposal also states the `Full-bundle tier:` (max tier if all optional edits were included), so the operator can see what the tier would be if they choose to include optional edits in a manual dispatch. The auto-remediation cron task reads only the `Bundle AR tier:` (mandatory edits), not the `Full-bundle tier:`.

### 6.3 Stop conditions for remediation drafting

The auditor halts remediation drafting and surfaces for operator decision if:

- **The drift finding doesn't match an established pattern.** Even if the finding is clearly drift, if remediation requires a new pattern not in §6.1, surface for operator decision before drafting.
- **The mechanical fix has ambiguous targets.** Example: a sub-spec self-header sync where the "correct" Codex anchor could plausibly be one of two §-anchors. Surface for operator clarification.
- **The fix would cascade into changes outside the finding's scope.** Example: a §4.2 path correction that requires updating five other documents that reference that path. Surface; the cascade may warrant a different fix shape than single-row edit.
- **The fix would touch a substantive governance section.** Drift findings that mechanically resolve to mechanical fixes are in scope; drift findings whose remediation requires substantive §3.x or §1.x content is operator decision.

### 6.4 Output format

The audit report includes a section §12 "Remediation drafts" (after §11 Synthesis) containing:

- Drift findings eligible for remediation drafting (per §6.1).
- Per-finding edit text (per §6.2 format).
- Drift findings that triggered §6.3 stop conditions, with the specific stop-condition cited.
- A consolidated **"Bundle proposal"** — single commit shape covering all eligible findings, with suggested commit message including the patch class (Codex registry update / sub-spec sync / roadmap reconciliation / etc.) and the version-classification recommendation (patch or minor per Codex §3.8).
- **Bundle AR tier** (aggregate of all included findings' Remediation tiers — maximum tier value, e.g., a bundle with Tier 1 + Tier 2 findings has bundle AR tier 2; a bundle with any Tier 3 finding has bundle AR tier 3 and the auto-remediation cron task will halt on it).

### 6.5 Operator review gate

The remediation drafts are not executed automatically. The operator reviews the bundle proposal and decides:

- **Approve as-drafted:** Operator dispatches CC to apply the bundle as drafted.
- **Approve with modifications:** Operator amends specific edits or drops findings from the bundle; dispatches CC.
- **Reject and request alternative:** Operator surfaces a different fix approach; the auditor re-drafts.
- **Defer specific findings:** Findings deferred remain open and surface in the next audit run.

The operator gate is the canonical authorization for governance commits. The skill does not bypass operator review.

### 6.6 Coverage and traceability

The audit report explicitly distinguishes:

- Findings the audit **identified**.
- Findings the audit **drafted remediation for** (eligible per §6.1).
- Findings the audit **deferred to operator decision** (per §6.3 stop conditions OR non-drift classification).

This allows subsequent audit runs to verify whether prior remediation drafts actually landed (and to flag if they didn't).

### 6.7 State board cross-reference

When §6 drafts remediation bundles for drift findings, the audit output must reference the operational state board (`docs/operations/state-board.md`) for tracking continuity. Specifically:

- The audit output document at `docs/audits/[date]-traceability-audit-runN.md` includes a closing section: *"State board update required after operator review of this bundle. Cowork updates state board §3.9 (Traceability audit state) and §3.3 (Work in flight) at next session start per Cowork operating discipline §9.4."*
- This ensures fresh sessions reading the state board after an audit run see audit findings without needing to open the audit document. The audit document remains authoritative for full detail; the state board surfaces the actionable summary.
- The skill executes the audit and produces the output document; Cowork synthesizes the audit output into the state board update at next session start. **The skill does NOT write directly to the state board** — that preserves the §9 Cowork-as-state-board-maintainer discipline (CC and the skill emit signals; Cowork updates the synthesis surface).

---

## 7. Implementation status

**Current as of 2026-05-18:** Spec ratified at v1.0 (2026-05-17); amended to v1.1 (2026-05-18) adding §6 Remediation drafting phase. Skill implementation work follows in Phase B Session 2.

**Skill implementation work** (Phase B Session 2 scope):
- Cowork-side execution capability for the four checks
- Report template aligned with §4 Outputs structure (now including §11 Synthesis and §12 Remediation drafts)
- First execution against current state (produces the inaugural traceability audit)
- Operator review of first execution's findings classification
- Calibration of Check 3 sampling strategy based on first run's coverage
- Calibration of §6 remediation drafting against the established pattern table (first eligible run is the v1.1 first-execution opportunity)

**Ongoing operation** (Phase B Session 2 completion onward):
- Operator invokes the skill on judged cadence; suggested baseline cadence is weekly during Phase B and Phase C (active governance change periods), monthly thereafter
- Each invocation produces a date-stamped audit report
- Carry-forward tracking accumulates across runs

---

## 8. Backlog and roadmap cross-references

**Codex §4.1 sub-spec index entry:** Traceability Check Skill is registered in Codex §4.1 with status updated from "(planned)" to "current" upon ratification of this spec.

**Backlog cross-references:**
- "Implement Traceability Check Skill" — follows ratification of this spec; sequenced as Phase B Session 2 dispatch
- "First execution of Traceability Check Skill against current state" — folded into Phase B Session 2 implementation work
- "Operator review and classification calibration" — follows first execution

**Roadmap cross-references:**
- Current product roadmap (v1.0) does not yet have a stage for this skill. The v2.0-aligned roadmap (produced in Phase B Session 3) will include Phase B Sessions 1 and 2 as stages with this spec as their authority.

---

## Version history

| Version | Date | Nature of change |
|---|---|---|
| v1.0 | 2026-05-17 | Initial spec ratification. Phase B Session 1 deliverable. |
| v1.1 | 2026-05-18 | Amendment: Added §6 Remediation drafting phase. Drift-class findings with established mechanical-fix patterns receive drafted edit text + bundle proposal; violations and ambiguity findings remain operator-decision. Reduces traceability audit loop from "audit → operator dispositions → Cowork drafts → operator approves → CC commits" to "audit-with-drafts → operator approves → CC commits." Established patterns enumerated in §6.1; new patterns require skill spec amendment, not auditor improvisation. Closes operating-discipline gap surfaced after Run 3 (2026-05-18) where 5 audit findings required 4-5 operator round-trips to resolve. |
| v1.3 | 2026-05-20 | Amendment: Added §5.5 Three-tier acceptance criteria. Defines Tier 1 (Blockers — gate phase closure), Tier 2 (Working set — carried forward, not blocking), Tier 3 (Background debt — deferred, not tracked in carry-forward). Phase closure gates only on Tier 1 resolution; Tier 2/3 acknowledged in closure declaration. Rationale: sampling-driven audits produce drift indefinitely; without tier classification, any new drift finding re-opens the closure gate (runaway gate problem). Closes Pending Dispatch 3 from Phase B formal closure dispatch (2026-05-20). |
| v1.2 | 2026-05-19 | Amendment: Added §6.7 State board cross-reference. When §6 drafts remediation, the audit output document references the state board; Cowork synthesizes audit findings into state board §3.9 at next session start per Cowork operating discipline §9.4. Preserves §9 Cowork-only-writes-state-board principle — the skill emits signals to `docs/audits/`, never writes directly to the state board. Part of the 2026-05-19 audit cohesion amendment bundle (state board spec v1.1 + Cowork operating discipline v1.2 + Traceability Check Skill v1.2). Closes the gap where audit findings sat in `docs/audits/` without state-board surfacing — fresh sessions now orient on the most recent audit run via the state board. |
| v1.7 | 2026-05-26 | Amendment: adds §5.8 Auto-remediation tier classification rubric. Each §12 Remediation draft finding gains a `Remediation tier:` field (per §6.2 format, amended this version) declaring whether auto-remediation should auto-execute (Tier 1), auto-execute with awareness flag (Tier 2), or halt for operator decision (Tier 3). §6.1 pattern catalog gains "Default AR tier" column (all six existing patterns: Tier 1) with a per-instance promotion note for §5.8 Step 2 awareness conditions. §6.4 bundle proposal gains a "bundle AR tier" requirement. Prerequisite for the auto-remediation cron task (Auto-Remediation Spec v1.0) — cron task reads §12 tier annotations rather than re-classifying at runtime, eliminating spec-logic duplication between the audit skill and the remediation cycle. Vocabulary disambiguation added: "Phase-closure tier" (§5.5) and "Auto-remediation tier" (§5.8) are orthogonal; both use 1/2/3 but govern different behaviors. |
| v1.6 | 2026-05-26 | Amendment: §5.7 gains Step 4 — Discord notification on scheduled-run completion. After `git push` (Step 3), the skill invokes `pipelines.shared.audit_notify` which parses the §1 totals row, builds a GitHub URL, and posts a one-line 🟢/🔴 summary to Discord `#commands` via the existing `DISCORD_WEBHOOK_COMMANDS` channel (no new env var). Post-then-exit-cleanly contract: Discord failure exits non-zero but does not mask the audit. Dry-run when webhook is empty (matches `roadmap_ack.py` pattern). Manual runs do not invoke Step 4. Adds `pipelines/shared/audit_notify.py` (helper), `pipelines/shared/tests/test_audit_notify.py` (18 cases), `pipelines/shared/audit_notify_README.md` (operator docs). |
| v1.5 | 2026-05-24 | Amendment: §5.6 codifies output placement — scheduled and manual runs write directly to `docs/audits/<date>-traceability-audit-run<N>.md` by default with mandatory `-run<N>` suffix and `max(N)+1` numbering; manual runs may instruct `outputs/` workflow per-invocation. §5.7 codifies scheduled-run auto-commit (`git add`/`commit`/`push` with mandatory commit-message format; halt-on-failure, non-zero exit, no in-skill retry). The original plan included a §5.4 amendment scoping operator review to manual runs; retired during Phase 1 discovery because §5.4 did not exist — the `outputs/`-review-promote workflow was operational convention, not spec text. Driven by the 2026-05-18 → 2026-05-24 `outputs/` accumulation incident (12+ drafts, several never promoted), which established that the operator-review gate on scheduled audit placement was not load-bearing. |
| v1.4 | 2026-05-24 | Amendment: expands the core check set from four to six. **Check 5 (cross-branch governance divergence)** — for every branch ahead of `main`, detects code/specs that `main` governance describes as shipped/Current but that live only on an unmerged branch. Codifies the cross-branch capability run ad-hoc 2026-05-15 + 2026-05-23 (the "main-only-audit blind spot"); separate cadence (per-merge-to-main + on operator request, not every daily run) + 30-min runtime cap for runs including it. **Check 6 (ratified-spec git-tracking sweep)** — confirms ratified specs on disk under `docs/specs/` + `docs/governance/` are git-tracked; runs every invocation. New **branch-divergence** classification added to §5.5 → Tier 2 (in-flight feature branches don't gate phase closure; escalates to Tier 1 only on operator determination of active incorrect decisions); untracked-ratified-spec (Check 6) → Violation/Tier 1 (broken durability guarantee). §6.1 gains the `git add` mechanical-fix pattern for Check 6; Check 5 findings are explicitly non-draftable (merge decisions are strategic, §6.3 stop condition). §4 output structure renumbered (Check 5/6 finding sections inserted; severity/sampling/carry-forward/synthesis/remediation shifted to §8–§12). Driven by the CIS v2 merge blind spot (Researcher/Editor code lived only on `geo/cis-v2` while Codex §4.1 claimed Researcher Current) + the Cross-Host Coordination Mechanism Spec untracked-on-disk-for-5-days case (both surfaced 2026-05-23/24). |
