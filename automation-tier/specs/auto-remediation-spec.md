<!--
PGK AUTOMATION TIER — VERBATIM REFERENCE COPY (GEO Machine implementation)
Source file:    docs/specs/auto-remediation-spec-v1.0.md
Source version: v1.0.1 (2026-05-27)
Source repo:    mames98/GEO-Machine, HEAD 4579313 (45793134dcaf1d1e8e243dcdab0f9ae3cc7a92c4, 2026-06-12)
Copied:         2026-07-11 (PGK automation-tier codification)
Everything below this comment block is byte-faithful to the source file.
Generalization map: ../generalize/auto-remediation-spec-notes.md
-->

# Auto-Remediation Spec

**Type:** Sub-spec of GEO Codex v2.2.0 (registered in §4.1)
**Version:** 1.0.1
**Status:** Ratified (operator-confirmed 2026-05-27)
**Canonical path:** `docs/specs/auto-remediation-spec-v1.0.md`
**Last revised:** 2026-05-27 (v1.0.1 amendment — removes §4.2 step 2 head-advance check. The check produced false halts on the audit's own Skill v1.6 §5.7 Step-3 auto-commit and on any operator commit landing in the 15-minute window between scheduled audit (HH:00) and scheduled cycle (HH:15). `str_replace`'s atomic-failure semantics already detect stale Before: text — if HEAD moved AND the Before: text no longer matches, the replace fails and the cycle halts with `verification_failed` (exit 4). The head-advance check was defensive armor against a problem `str_replace` already solves. Exit code 3 (`head_advanced`) is now unused. Driven by Run 9 cycle 1 evidence: the 12:15 cycle false-halted because the audit's own 10:14 Step-3 commit advanced HEAD by one commit, blocking the 3 Tier 1 findings (3.1 / 3.2 / 4.2) the audit explicitly drafted for auto-execution.) Prior: 2026-05-27 (initial ratification)
**Codex authority:** v2.2.0 (`docs/strategy/geo-codex-v2.0.md`)

---

## 1. Purpose and design principle

### 1.1 What this spec covers

This spec codifies an autonomous remediation system that consumes the daily traceability audit output (Skill v1.7+, `docs/audits/<date>-traceability-audit-run<N>.md`) and addresses findings without requiring per-finding operator dispatch.

The system closes the gap between *"audit found it"* and *"audit finding addressed"* on routine work that does not require operator strategic input. Operator time is conserved for productive work — shipping features, content, and decisions — not for ratifying mechanical governance hygiene.

### 1.2 Why this exists

The daily traceability audit (Skill v1.6 + v1.7) runs autonomously and produces a structured findings report. As of Run 8 (2026-05-26), Finding 2.1 (backlog line 787 archived-roadmap path) has carried forward across four consecutive runs without addressing. The remediation draft has been available in §6 of every run; the operator has not dispatched it because each individual carry-forward fix is too small to justify the operator's attention.

Auto-remediation addresses this class of finding autonomously while preserving operator authority over substantive work. The system is constrained to a narrow safety envelope (§3); findings outside that envelope continue to surface for operator decision.

### 1.3 Foundational principle: default to action, halt only on high-blast-radius-if-wrong

Auto-remediation defaults to action. The friction of executing a wrong fix on a §6.1 mechanical-pattern finding is bounded: the change is small, lives on an isolated quarantine branch for 30 days (§7), and is mechanically detectable in the next audit run if it introduces a new violation.

Auto-remediation halts only when:
- The finding falls outside the established mechanical-fix patterns (`Skill v1.7 §6.1`)
- The classification confidence is low
- The finding contradicts a recent operator decision
- The finding affects branch-divergence or merge state

This calibration matches the Codex §1.2 effectiveness principle: speculative defensive design for problems that haven't occurred is over-engineering. Auto-remediation runs unless there is concrete reason to stop.

### 1.4 Codex Principle 0 alignment

Principle 0 (Truth Over Closure) governs every auto-remediation cycle. The system **never fabricates** a fix when the audit finding is ambiguous. Preferred failure modes per Principle 0: drop the finding from the auto-remediation cycle, flag with uncertainty marker, surface to operator. Producing nothing is preferable to producing something fabricated. When auto-remediation cannot determine the correct fix with high confidence, it halts and surfaces — it does not improvise.

This is operationalized via the §6.1 mechanical-fix allowlist: only findings that match a pre-ratified pattern with explicit Before:/After: drafting (per Skill v1.6 §6.2) are eligible for auto-execution. Pattern-match failure is a halt condition, not a guess-and-commit.

---

## 2. Scope

### 2.1 In scope

- Findings drafted in the audit's §12 "Remediation drafts" section (Skill v1.6 §6.4) that match a Skill v1.6/v1.7 §6.1 mechanical-fix pattern with explicit Before:/After: text
- Single-commit-per-cycle execution on the auto-remediation quarantine branch (§7) during the first 30 days; direct-to-main after graduation
- Discord digest of cycle outcome to `#commands` (reuses `pipelines/shared/audit_notify.py` infrastructure pattern)
- Cycle-level halt with operator interrupt when any halt condition fires

### 2.2 Out of scope

- Findings classified as **violation** (broken trace) or **ambiguity** (operator judgment needed) — these remain operator-decision per Skill v1.6 §6.1
- Findings classified as **drift** but NOT matching an established §6.1 mechanical-fix pattern — these surface to operator per Skill v1.6 §6.3 stop conditions
- Check 5 branch-divergence findings — explicitly non-draftable per Skill v1.6 §6.1
- Cross-audit-run finding aggregation (each run is processed independently in v1.0)
- Cross-branch auto-remediation (`main` only in v1.0)
- Operator decision-queue persistent storage (Discord digest is the sole surface in v1.0)
- Substantive governance edits (§3.x or §1.x Codex content) — Skill v1.6 §6.3 stop condition
- Edits that cascade beyond the finding's scope — Skill v1.6 §6.3 stop condition

### 2.3 v1.0 → future-version surface

The dispatch scoping this spec contemplated broader Tier 1 coverage (sibling format matching, placeholder text selection, Cowork-maintained surfaces like CLAUDE.md PART 2 backfill). These categories are explicitly **deferred** to future versions of this spec, gated on:

(a) The corresponding mechanical-fix patterns being ratified into Skill spec §6.1 (with Before:/After: drafting requirements), and
(b) The 30-day graduation criteria (§7) having flipped for the existing §6.1 patterns.

This sequencing prevents pre-authorization of categories whose mechanical-fix shape has not been operationally proven.

---

## 3. Three-tier classification model

Every audit finding eligible for auto-remediation processing is classified into one of three tiers. Tier determines execution behavior.

### 3.1 Tier 1 — Auto-execute and commit

A finding is Tier 1 if **all** of the following hold:

(a) The finding's audit-classification is **drift** (per Skill v1.6 §5.4).
(b) The audit's §12 "Remediation drafts" section contains an entry for the finding with explicit Before:/After: text per Skill v1.6 §6.2 format.
(c) The finding's matched pattern is one of the entries enumerated in Skill v1.7 §6.1 (the mechanical-fix allowlist).
(d) The remediation draft did not trigger any Skill v1.6 §6.3 stop condition (ambiguous targets, cascade, substantive content).

**Behavior:** Auto-remediation applies the drafted edits exactly as written in §12, verifies via the drafted verification check, and commits.

**v1.0 §6.1 pattern coverage** (mirrored from Skill v1.7 §6.1 — kept in sync via §10.2 of this spec):

| Pattern | Example |
|---|---|
| Sub-spec self-header Codex version label sync | `"Sub-spec governed by GEO Codex v2.0.4 → v2.1.0"` |
| Roadmap/backlog header Codex version reference update | `"Codex authority: v2.0.1 → v2.1.0"` |
| Codex §4.1 row path correction (file-vs-directory drift) | `"docs/videographer/ → docs/specs/videographer-agent-spec-v1.0.md"` |
| Codex §4.2 row path correction (path-drift class) | `"docs/governance/X.md → docs/brand/X.md"` |
| Codex §4.1 row text update for shipped sub-spec | `"Planned → Current with canonical path"` |
| Untracked ratified spec → git-track (Check 6) | `git add docs/specs/X-v1.0.md` |

### 3.2 Tier 2 — Auto-execute and flag for operator awareness

A finding is Tier 2 if it would otherwise be Tier 1 (clauses a–d above) BUT it carries one or more **awareness conditions**:

- The edit touches a Codex section anchor (§N.M label change, not text-internal-to-section)
- The edit propagates a change across **two or more** files (e.g., a §4.2 path correction that requires updating documents that cite the moved path)
- The edit establishes a versioned artifact (version-history table row, registration entry) that downstream sub-specs reference

**Behavior:** Auto-remediation executes per Tier 1 procedure. The Discord digest flags Tier 2 entries explicitly: `🟡 N decisions executed — quick glance recommended`. Cycle continues; no halt.

**Why "execute then flag" instead of "halt and ask":** awareness conditions describe blast radius if the underlying pattern-match is wrong, not blast radius of the standalone edit. The §6.1 pattern remains pre-ratified; the awareness flag signals that *if* a subsequent operator review catches a problem, the cleanup cost is moderately higher (two files instead of one). This is information for the operator, not a gate.

### 3.3 Tier 3 — Halt and interrupt

A finding is Tier 3 if **any** of the following hold:

(a) The audit classified it as **violation** (any violation in the run is a Tier 3 trigger — auto-remediation cycle halts entirely until operator dispatches).
(b) The audit classified it as **ambiguity**.
(c) The audit classified it as drift but the remediation phase triggered a Skill v1.6 §6.3 stop condition.
(d) The finding's matched pattern is **not** in the Skill v1.7 §6.1 allowlist (even if remediation text was drafted speculatively).
(e) The finding cites a recent operator decision (within trailing 7 days) that contradicts the proposed remediation.
(f) The finding affects branch-divergence (Check 5) or merge state.

**Behavior:** Auto-remediation halts the entire cycle (no Tier 1 or Tier 2 commits proceed). The Discord interrupt fires with the format specified in §5. The next cycle does not run until the operator either:
- Dispatches a fix and the next audit run shows the finding resolved, OR
- Explicitly invokes auto-remediation with `--skip-tier3-block` (operator override; logged in cycle output for the audit trail)

**Why a single Tier 3 finding halts the whole cycle:** Tier 3 indicates either a structural problem (new violation) or a confidence failure (unrecognized pattern). Either way, executing the Tier 1/2 edits while one Tier 3 sits unaddressed risks the operator missing the Tier 3 surface in the digest noise. Halt-first preserves Tier 3 visibility.

---

## 4. Execution flow

### 4.1 Invocation

Auto-remediation is invoked as a separate cron entry, **not** chained to the audit task's last step (per dispatch guidance and §6.7 of the Cowork operating discipline — separation of concerns and easier rollback).

The cron entry runs on the Mac Mini (where claude_code CLI is installed and Discord webhooks are reachable). Sequencing: 5–10 minutes after the scheduled audit completes. The audit's post-push Discord notification (Skill v1.6 §5.7 Step 4) lands first; the auto-remediation cycle digest lands second.

The invocation surface is a Claude Code headless session loaded with the `auto-remediation` skill. This is **CC running in sub-mode**, not a fifth role — per CLAUDE.md PART 1 "Team structure — CC sub-mode (auto-remediation)" subsection (added in v1.0 of this spec). The skill instructs the session to:

1. Locate the most recent traceability audit file at `docs/audits/<date>-traceability-audit-run<N>.md`.
2. Parse the §1 Summary statistics table and the §12 Remediation drafts section.
3. Classify each finding per §3 of this spec.
4. Execute per §4.2.
5. Post the Discord digest per §5.
6. Exit.

### 4.2 Per-cycle algorithm

```
1. Read latest audit file. If audit_file.mtime > 24h ago: halt with reason
   "audit_stale". (Auto-remediation only runs against fresh audits.)
2. (Removed v1.0.1.) The cycle previously verified HEAD on main matched the audit's
   "Audit-state" line; if not, halted with reason "head_advanced". This check
   false-halted on every cycle under normal operation (the audit's own Step-3
   auto-commit advances HEAD between audit-time and cycle-time). str_replace's
   atomic-failure semantics in step 5.b already detect stale Before: text — if
   HEAD moved AND the Before: text changed, the replace fails and the cycle
   halts with verification_failed (exit 4). No replacement check is needed.
3. Parse audit findings into a list. For each finding, determine tier per §3.
4. If ANY finding is Tier 3: halt the cycle. Post Discord interrupt per §5.3.
   No Tier 1 or Tier 2 commits are made.
5. If all findings are Tier 1 or Tier 2 (or there are no findings):
   a. Switch to or create the auto-remediation branch (§7.1).
   b. For each Tier 1 + Tier 2 finding:
      - Apply the Before:/After: edits from §12.
      - Run the verification check from §12. If verification fails: halt
        the entire cycle, revert any edits applied this cycle, post Discord
        per §5.4 (verification_failed).
   c. Stage edits via explicit named-path git add (per Cowork operating
      discipline §10.1 dispatch convention — no `-A`, no `.`).
   d. Commit with the message format in §6.
   e. Push.
   f. Post Discord digest per §5.1 (clean) or §5.2 (with-Tier-2-awareness).
6. Cycle ends. Exit 0 on clean completion, non-zero on any halt path
   (audit_stale=2, verification_failed=4, tier3_halt=5). Exit 3 (`head_advanced`)
   removed in v1.0.1.
```

### 4.3 Idempotency

Auto-remediation must be idempotent against the same audit file. Re-invocation after a successful cycle for the same audit file is a no-op: the findings have been remediated, the next audit run will not reproduce them, and the auto-remediation cycle exits 0 with reason "nothing_to_remediate" after parsing.

### 4.4 Failure isolation

If verification fails on any single Tier 1/2 finding during cycle execution, **all edits applied during the cycle revert** (via `git restore` against the working tree before any staging). The cycle does not partially-commit. This preserves the all-or-nothing property of the cycle's commit on `main` (or the quarantine branch during the first 30 days).

---

## 5. Discord digest format

All digests post to `#commands` via the existing `DISCORD_WEBHOOK_COMMANDS` channel (reuses `pipelines/shared/discord_notify.send_alert(channel="commands", ...)`). No new env var. Mirrors the v1.6 §5.7 Step 4 pattern.

### 5.1 Clean cycle (all Tier 1, no Tier 2, no halts)

```
🟢 Auto-remediation Run N complete. N findings remediated, all Tier 1. Commit <sha7>. <github-url>
```

Example:

```
🟢 Auto-remediation Run 9 complete. 1 finding remediated, all Tier 1. Commit a1b2c3d. https://github.com/mames98/GEO-Machine/commit/a1b2c3d
```

### 5.2 Cycle with Tier 2 awareness flags

```
🟡 Auto-remediation Run N complete. M findings remediated (X Tier 1 + Y Tier 2 — quick glance recommended). Commit <sha7>. <github-url>
Tier 2 entries:
• <one-line plain-English summary of finding + what changed>
• <one-line plain-English summary of finding + what changed>
```

The body section lists Tier 2 entries inline. Plain English; no §-anchor jargon; the operator should be able to read the summary on a phone and understand the change in 5 seconds.

### 5.3 Cycle halted on Tier 3

```
🔴 Auto-remediation halted — operator decision needed.
What tripped: <one-sentence plain-English description of the Tier 3 finding>
Why it matters: <one-sentence business-impact framing>
Recommended action: <specific operator action — dispatch text or decision>
Alternatives: <1–2 alternative paths>
Blast radius: <bounded / cross-spec / cross-repo>
Time-to-decide: <urgent / this week / no rush>
Audit findings: <github-url to the audit file>
```

**Plain-English requirements** (per dispatch §1.3 operator-facing surface):
- No §-anchor jargon in the operator-facing lines ("what tripped", "why it matters", "recommended action")
- Every halt includes a recommended action — operator's job is to ratify or override, never to think from scratch
- "Why it matters" frames business impact, not internal-consistency abstraction
- Blast-radius assessment is one of three values: **bounded** (single sub-spec or doc), **cross-spec** (multiple sub-specs), **cross-repo** (touches code + governance both)

If multiple Tier 3 findings fire in the same run, the digest lists them in audit-file order, each with its own block. The cycle halts on the first; subsequent Tier 3 surfaces are informational only.

### 5.4 Cycle halted on verification failure mid-execution

```
🔴 Auto-remediation halted — verification check failed mid-cycle.
Finding: <finding ID + plain-English description>
Pattern matched: <§6.1 pattern name>
Verification command: <the exact verification command from §12>
Verification result: <stdout/stderr>
All cycle edits reverted. Working tree restored to pre-cycle state.
Audit findings: <github-url to the audit file>
```

This is a stronger halt than Tier 3 because the failure indicates the §6.1 pattern's verification logic produced an unexpected result — either the pattern's verification is incorrect or the underlying state is in a configuration the pattern didn't anticipate. The skill spec should be amended to handle the case before re-invocation.

### 5.5 Cycle halt for environmental reasons (audit_stale)

```
⚪ Auto-remediation skipped this cycle — <reason in plain English>. Next attempt next scheduled cycle.
```

`⚪` (white circle) for environmental/infrastructure halts. Distinct from `🔴` (operator action needed) and `🟡` (operator glance). No operator action required; cycle will retry on its next schedule.

---

## 6. Commit message format

The commit message follows the Conventional Commits shape used elsewhere in the repo, with auto-remediation-specific subject prefix.

```
auto-remediate: traceability audit Run N — M findings (X Tier 1 + Y Tier 2)

Applied mechanical-fix patterns per Skill v1.7 §6.1, drafted by audit Run N
(`docs/audits/<date>-traceability-audit-runN.md` §12).

Findings remediated:
1. [Tier 1] Finding <ID> — <one-line plain-English summary>
   Pattern: <§6.1 pattern name>
   Files: <comma-separated paths>
   Verification: passed (<one-line description of the check>)
2. [Tier 2] Finding <ID> — <one-line plain-English summary>
   Pattern: <§6.1 pattern name>
   Files: <comma-separated paths>
   Awareness condition: <which §3.2 condition triggered the Tier 2 flag>
   Verification: passed

[... per finding ...]

Cycle metadata:
- Audit file: docs/audits/<date>-traceability-audit-runN.md
- Audit run number: N
- Audit HEAD at time of audit: <commit sha7>
- Auto-remediation cycle started: <ISO 8601 UTC>
- Auto-remediation cycle completed: <ISO 8601 UTC>
- Quarantine status: <on auto-remediation branch / graduated to main>

Per `docs/specs/auto-remediation-spec-v1.0.md`.
```

**Why this detail in the commit message:** the commit is the audit trail for the autonomous action. If an operator later reverts the commit, the message tells them exactly which findings were addressed, which patterns were applied, and how to verify the state. The §6.1 pattern names are jargon-bearing (operator-internal) because the commit message is a developer-facing surface; the Discord digest is the operator-facing surface and uses plain English.

---

## 7. Operational protections — quarantine branch and graduation

### 7.1 Auto-remediation quarantine branch

For the first 30 days of operation following ratification of this spec, auto-remediation commits do **not** land directly on `main`. They land on a date-stamped quarantine branch:

```
auto-remediation/YYYY-MM-DD
```

The branch is created (if not yet existing) at the start of each cycle, with `main` as its base. Commits accumulate on the branch through the day. The operator merges the branch to `main` once daily via a single explicit operator action (typically a one-click PR merge or a `git merge` invocation). This preserves the operator-gates-publish discipline (Global Design Principle 2) even though CC is auto-committing.

The merge action is fast for the operator (one decision per day, reviewing accumulated commits before merging). If the operator chooses not to merge on a given day, the next cycle's branch supersedes — the unmerged branch remains as an artifact for inspection but does not accrue new commits.

### 7.2 Graduation criteria

Auto-remediation graduates to direct-to-main commits when **both** of the following hold continuously over a **trailing 30-day window**:

(a) **Zero reverts.** No auto-remediation commit has been reverted by the operator or by any subsequent dispatch within the 30-day window.
(b) **Zero new violations traceable to auto-remediation.** No subsequent audit run (within the 30-day window) has flagged a new violation whose root cause is a prior auto-remediation commit.

Both criteria check disjoint failure modes:
- (a) catches the case where the operator caught an auto-remediation mistake and undid it (a known failure)
- (b) catches the case where auto-remediation introduced drift or a violation that wasn't caught immediately but surfaced in a later audit (a silent failure)

### 7.3 Graduation flip is automatic

When both criteria hold, the graduation flip is automatic. No operator sign-off required. The graduation check runs at the start of each cycle:

```
If (trailing_30_day_revert_count == 0) AND
   (trailing_30_day_traceable_violation_count == 0) AND
   (auto_remediation_cycles_in_window >= 10):
    Set quarantine_status = "graduated"
    Set commit_target_branch = "main"
Else if quarantine_status == "graduated" AND either criterion fails:
    Set quarantine_status = "quarantined"
    Set commit_target_branch = "auto-remediation/YYYY-MM-DD"
    Reset window to day 0 from the failure event
    Post Discord alert: "🟠 Auto-remediation re-quarantined — <reason>"
```

The `>= 10` floor on cycle count is a safety guard: graduation requires both 30 calendar days AND at least 10 cycles of clean operation. Daily cycles meet this trivially; if cycles are sparse for any reason (operator pause, infrastructure issue), graduation waits until both clocks elapse.

### 7.4 Re-quarantine on graduation failure

If a graduated auto-remediation system later fails either criterion (a revert lands, or a new violation traces to a prior auto-remediation commit), the system **automatically re-quarantines**. The trailing 30-day window resets to day 0 from the failure event. The operator does not need to manually intervene; the same mechanical gate enforces graduation in both directions.

This prevents a single bad cycle from being papered over by 29 subsequent clean cycles — the gate is strict in both directions.

### 7.5 Quarantine status surface

The current quarantine status is stored at `data/auto_remediation_state.json`:

```json
{
  "quarantine_status": "quarantined" | "graduated",
  "window_start": "<ISO 8601 UTC date>",
  "cycles_in_window": <int>,
  "reverts_in_window": <int>,
  "traceable_violations_in_window": <int>,
  "last_cycle_ts": "<ISO 8601 UTC>",
  "last_cycle_outcome": "clean" | "tier3_halt" | "verification_failed" | "audit_stale" | "head_advanced",
  "last_graduation_flip_ts": "<ISO 8601 UTC>" | null
}
```

Reads are atomic (single-file read). Writes follow the existing JSON-state-file pattern (`data/orchestrator_state.json` is the precedent).

---

## 8. Cron sequencing and integration

### 8.1 Cron entry

A new cron entry on the Mac Mini (sequenced 5–10 minutes after the scheduled audit):

```
# Auto-remediation cycle — runs after each scheduled traceability audit.
# Per docs/specs/auto-remediation-spec-v1.0.md.
15 6 * * * cd ~/geo-machine && /usr/bin/python3 -m pipelines.auto_remediation.cycle >> data/logs/auto-remediation.log 2>&1
15 12 * * * cd ~/geo-machine && /usr/bin/python3 -m pipelines.auto_remediation.cycle >> data/logs/auto-remediation.log 2>&1
```

The scheduled traceability audit fires twice daily at approximately 06:00 and 12:00 Mac Mini local time (Skill v1.6 §5.6 cadence note). The auto-remediation cycle fires at :15 past each — 5–10 minute buffer over the audit completion ensures the audit's git push has settled before the auto-remediation cycle reads `main`.

(If the audit cadence changes, the auto-remediation cron lines update in lockstep. The skill spec § for cadence is descriptive, not normative.)

### 8.2 Cron entrypoint module

A new Python module at `pipelines/auto_remediation/cycle.py`. The module is the cron entry; it reads the latest audit file, classifies findings, executes per §4, posts digest per §5, and exits.

The module is **not** itself the agentic invocation surface. The module is a thin orchestrator that invokes the headless Claude Code session (per §8.3) and surfaces the session's structured output. This separation mirrors the existing pattern where Python orchestrates and Claude executes.

**Optional design note:** for v1.0, the entry can be a pure Python module that performs the deterministic parsing + classification + verification work and only invokes Claude when actual edit-application requires judgment (e.g., parsing the §12 Before:/After: text into the right `str_replace` calls). This minimizes Claude invocations and matches Codex §1.2 effectiveness ("simple solutions over sophisticated ones"). Final v1.0 implementation may bypass the Claude headless session entirely if the deterministic path covers all §6.1 patterns. Implementation is at CC's discretion within v1.0 scope.

### 8.3 Skill loading (if Claude invocation is needed)

If the v1.0 implementation requires Claude to apply edits (i.e., not pure-Python deterministic), the invocation loads a new skill at `~/.claude/skills/auto-remediation/SKILL.md` (Mac Mini) that:

(a) Reads the audit file
(b) Applies the §12 Before:/After: edits via str_replace tools
(c) Runs the §12 verification check
(d) Returns structured output: `{outcome, findings_applied, verification_results}`

The skill is loaded via `claude_code --skill auto-remediation` invocation pattern. Per CLAUDE.md PART 1 "Team structure — CC sub-mode (auto-remediation)" subsection (added in this spec): CC running this skill is CC in sub-mode — same role, scheduled invocation, narrow scope.

### 8.4 Logging

All cycles log to `data/logs/auto-remediation.log` (mirrors `data/logs/scout.log`, `data/logs/writer.log` conventions). Log line format:

```
<ISO 8601 UTC>  cycle_start  audit_file=<path>  audit_run=N
<ISO 8601 UTC>  classify     finding_count=M  tier1=X  tier2=Y  tier3=Z
<ISO 8601 UTC>  execute      finding=<id>  pattern=<§6.1 name>  outcome=applied
<ISO 8601 UTC>  verify       finding=<id>  outcome=passed
<ISO 8601 UTC>  commit       sha=<sha7>  branch=<branch>
<ISO 8601 UTC>  push         remote=origin  outcome=success
<ISO 8601 UTC>  digest       channel=#commands  emoji=🟢
<ISO 8601 UTC>  cycle_end    outcome=clean  exit_code=0
```

Halts log with the relevant `cycle_end outcome=<reason>` line and a non-zero exit code per §4.2.

### 8.5 Reuse — what auto-remediation does not build

Per dispatch §3 (operational integration): use existing pipelines/shared/ infrastructure; do not duplicate.

| Need | Existing surface to reuse |
|---|---|
| Discord posting | `pipelines/shared/discord_notify.send_alert(channel="commands", ...)` |
| Audit file parsing (totals table, run number) | `pipelines/shared/audit_notify.extract_totals()`, `extract_run_number()` |
| GitHub URL construction | `pipelines/shared/audit_notify.build_github_url()` (refactor to accept arbitrary blob path if needed) |
| Env-var loading | `pipelines/shared/config._load_dotenv()` (auto-loads on import) |
| Event logging to agent_events.db | `pipelines/shared/event_logger.log_event()` |
| Webhook channel | `DISCORD_WEBHOOK_COMMANDS` (no new env var) |

The §12 Remediation drafts parsing is **new** — no existing helper parses that section. The new parser lives at `pipelines/auto_remediation/remediation_parser.py` and may eventually be promoted to `pipelines/shared/` if other callers need it.

### 8.6 State board interaction

Auto-remediation does **not** write the state board directly. This preserves Cowork operating discipline §9 (Cowork is the sole state-board maintainer).

Auto-remediation commits feed into Cowork's session-start sync (Cowork operating discipline §9.2): Cowork reads recent `git log` on session start, sees auto-remediation commits, and updates state board sections §3.3 (Recent shipped) and §3.9 (Traceability audit state) per its existing §9.4 trigger-based update discipline.

The Discord digest in `#commands` is sufficient operator-facing signal between Cowork sessions. The state board reflects the cumulative state once Cowork next runs.

---

## 9. Failure modes and recovery

### 9.1 Failure mode: verification check fails mid-cycle

**Symptom:** A Tier 1 or Tier 2 edit applied, but the §12 verification check returned an unexpected result.

**Mechanism:** §4.4 — all cycle edits revert via `git restore`. Working tree returns to pre-cycle state. Cycle exits with code 4 and posts Discord per §5.4.

**Recovery:** Operator investigates the verification failure. Likely causes:
- The §6.1 pattern's verification logic is incorrect — amend the skill spec
- The audit's §12 Before:/After: drafting captured the wrong file state — surface to operator for re-drafting in the next audit run
- The underlying file changed between audit-run-time and auto-remediation-cycle-time — str_replace's atomic-failure semantics in §4.2 step 5.b detect this (the Before: text no longer matches; the replace fails and the cycle halts with verification_failed)

The cycle does not retry. The next scheduled cycle re-evaluates against the next audit run.

### 9.2 Failure mode: Discord post fails

**Symptom:** Cycle completed successfully (commit pushed) but Discord digest could not be posted (network, 5xx, webhook revoked).

**Mechanism:** Per the audit_notify post-then-exit-cleanly contract (Skill v1.6 §5.7 Step 4), Discord failures **do not** roll back the commit. The commit is on `main` (or the quarantine branch); the audit trail is the git log. Discord failure is logged but does not mask cycle success.

**Recovery:** Operator inspects `data/logs/auto-remediation.log` if a cycle outcome is unclear from Discord. The log is authoritative; Discord is the convenience surface.

### 9.3 Failure mode: cycle takes longer than expected

**Symptom:** Cycle exceeds an upper-bound runtime (e.g., 5 minutes for a finding count of 4).

**Mechanism:** v1.0 does not implement a hard runtime cap (no precedent in existing cron entries; the Skill v1.6 §5.6 30-min ceiling is the audit's, not auto-remediation's). The cron entry will be killed if it runs past the next cycle's scheduled start, but this is a hard kill from cron's side, not a graceful timeout.

**Recovery:** v1.0 accepts this risk because cycle runtime is expected to be measured in seconds (mechanical edits + verification commands). If runtime becomes a problem operationally, v1.1 of this spec adds a `--timeout-seconds` argparse arg with a graceful-shutdown contract (revert partial work, post `⚪` digest).

### 9.4 Failure mode: operator missed a Tier 2 awareness flag in real-time

**Symptom:** A Tier 2 commit went out; the operator didn't catch it in the digest; the change has compounding downstream effects.

**Mechanism:** The 30-day quarantine branch (§7.1) is the safety net. Tier 2 commits sit on the quarantine branch until the operator merges. The operator's daily merge review is the chance to catch missed-in-real-time Tier 2 issues before they reach `main`.

After graduation (§7.2), Tier 2 commits land directly on `main` — but the graduation criteria require zero reverts and zero new violations over 30 days, which means Tier 2 awareness flags have been operating cleanly across the window. Post-graduation, a missed Tier 2 issue surfaces in the next audit run (which would re-flag the underlying drift if it wasn't actually resolved), and the spec's re-quarantine mechanism (§7.4) catches it.

### 9.5 Failure mode: Tier 3 halt is wrong (false positive)

**Symptom:** Auto-remediation halted on a finding that was actually safely Tier 1, but the spec's classification rules incorrectly tagged it Tier 3.

**Mechanism:** The operator overrides via `--skip-tier3-block` (per §3.3). The override is logged in the cycle output for the audit trail.

**Recovery:** If the same finding repeatedly mis-classifies as Tier 3, the Tier 3 trigger condition in §3.3 needs amendment via a v1.1+ patch to this spec.

---

## 10. Spec maintenance

### 10.1 Codex registration

This spec is registered in Codex v2.2.0 §4.1 as:

```
| Auto-Remediation Spec v1.0 | `docs/specs/auto-remediation-spec-v1.0.md` | Current | §2.3, §3.6 | Autonomous remediation of mechanical-fix audit findings per Skill v1.7+ §6.1 allowlist. CC sub-mode (CLAUDE.md PART 1 "Team structure — CC sub-mode (auto-remediation)" subsection). Quarantine branch + 30-day graduation gate per §7. |
```

Codex bumps v2.1.6 → v2.2.0 (minor — substantive new governance addition per §3.8).

### 10.2 Skill v1.6 §6.1 ↔ this spec §3.1 sync discipline

The §6.1 mechanical-fix pattern allowlist in the Skill spec is the **canonical** source for Tier 1 eligibility. The pattern table in §3.1 of this spec is a mirror copy maintained for reader convenience.

When Skill v1.7+ amends §6.1 (adds, modifies, or removes a pattern):

(a) The Skill spec amendment is the load-bearing change. Auto-remediation cycle behavior follows the Skill spec immediately on the next cycle that reads an audit produced under the amended skill version.
(b) This spec's §3.1 mirror table is updated in the same dispatch as the Skill spec amendment, OR in the next governance cleanup dispatch — whichever lands first. The mirror table going stale relative to the Skill spec is a Tier 3-class drift finding the audit itself will surface in subsequent runs.
(c) The §6.1 pattern allowlist is the ratification gate. A new pattern is *not* eligible for Tier 1 auto-remediation until Skill v1.7+ ratifies it with explicit Before:/After: drafting requirements. Pre-ratification, the pattern surfaces in audit §12 only via §6.3 stop-condition handling (operator-decision).

### 10.3 Revision history maintenance

This spec's revision history (table in §11) is updated on every amendment. Per Codex §3.8, the revision history is the change-protocol record for sub-specs.

### 10.4 Forward pointers — known v1.1+ work

The following items are explicitly out of scope for v1.0 and recorded as forward-pointer candidates:

- **Cycle runtime cap** with graceful shutdown (§9.3)
- **Cross-audit-run aggregation** — process multiple recent audit files in one cycle
- **Cross-branch auto-remediation** — apply remediation to other branches beyond `main`
- **Persistent operator decision queue** — replace Discord-only digest with a `data/auto_remediation_decisions.db`-style persistent surface
- **Broader Tier 1 categories** beyond Skill §6.1 (sibling format matching, placeholder text, Cowork-maintained surfaces) per §2.3 graduation gate

Each is a candidate for v1.1+ if and when operational evidence justifies the additional surface.

---

## 11. Revision history

| Version | Date | Nature of change |
|---|---|---|
| v1.0 | 2026-05-27 | Initial spec ratification. Codifies autonomous remediation of audit findings matching the Skill v1.7+ §6.1 mechanical-fix allowlist. Three-tier classification (auto-execute, execute-and-flag, halt). Plain-English operator-facing Discord digests with recommended actions on every halt. 30-day quarantine branch with zero-reverts + zero-new-violations graduation criteria (both clocks must elapse; automatic flip; automatic re-quarantine on failure). Cron-fired (separate entry from audit task per dispatch guidance — cleaner separation of concerns, easier rollback). Reuses `pipelines/shared/` infrastructure (audit_notify, discord_notify, DISCORD_WEBHOOK_COMMANDS); does not duplicate webhook plumbing or env vars. CC sub-mode classification (CLAUDE.md PART 1 "Team structure — CC sub-mode (auto-remediation)" subsection added in this dispatch — no fifth role). Closes the carry-forward gap surfaced in Run 8 (Finding 2.1's 4-consecutive-run persistence). |
| v1.0.1 | 2026-05-27 | Patch: removes §4.2 step 2 head-advance check + §5.5 `head_advanced` reference + exit code 3. The check false-halted on the audit's own Skill v1.6 §5.7 Step-3 auto-commit (every cycle, under normal operation) and on any operator commit landing between scheduled audit (HH:00) and scheduled cycle (HH:15). `str_replace`'s atomic-failure semantics already detect stale Before: text. Surfaced by Run 9 cycle 1 false-halt 2026-05-27T16:15 — blocked 3 Tier 1 findings (3.1 / 3.2 / 4.2) from auto-executing. |

---

## 12. Cross-references

- **Authority document:** GEO Codex v2.2.0 (`docs/strategy/geo-codex-v2.0.md`) §4.1 sub-spec index
- **Prerequisite skill:** Traceability Check Skill v1.7+ (`docs/governance/traceability-check-skill-spec.md`) §6.1 (pattern allowlist), §6.2 (drafting requirements), §6.3 (stop conditions), §6.4 (output format), §12 (Remediation drafts section)
- **Discord post primitive:** `pipelines/shared/discord_notify.py` `send_alert()`
- **Audit file parsing helpers:** `pipelines/shared/audit_notify.py` `extract_totals()`, `extract_run_number()`, `build_github_url()`
- **State board maintenance:** Cowork operating discipline (`docs/governance/cowork-operating-discipline.md`) §9.2, §9.4 — Cowork's session-start sync surfaces auto-remediation commits to state board
- **Role discipline:** CLAUDE.md PART 1 — "Team structure" role table (CC sub-mode per §4.4 added by this spec's dispatch)
- **Quarantine state file:** `data/auto_remediation_state.json` (new in v1.0)
- **Cycle entrypoint:** `pipelines/auto_remediation/cycle.py` (new in v1.0)
- **Cycle log:** `data/logs/auto-remediation.log` (new in v1.0)
