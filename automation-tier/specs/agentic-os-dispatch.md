<!--
PGK AUTOMATION TIER — VERBATIM REFERENCE COPY (GEO Machine implementation)
Source file:    docs/superpowers/plans/2026-05-31-agentic-os-v1.md
Source version: v1.0 implementation plan (2026-05-31)
Source repo:    mames98/GEO-Machine, HEAD 4579313 (45793134dcaf1d1e8e243dcdab0f9ae3cc7a92c4, 2026-06-12)
Copied:         2026-07-11 (PGK automation-tier codification)
Everything below this comment block is byte-faithful to the source file.
Generalization map: ../generalize/agentic-os-dispatch-notes.md
-->

# GEO Machine Agentic OS v1.0 — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the operator-side intelligence layer (Operator Knowledge Layer + self-maintaining hot cache + extended governance Loop + dashboard visibility) so new sessions orient in <30s and governance drift is caught and reported without operator polling.

**Architecture:** An in-repo Obsidian-readable wiki (`docs/operator-wiki/`) holds the OKL. A marker-based Python generator (`scripts/update_hot_cache.py`) synthesizes `hot.md` from authoritative sources (state board, git log, ADR index, AR state, status_collector). Three autonomous refresh triggers keep it fresh (governance orchestrator on a 6h cron as the reliable baseline; a post-AR cron post-step; CC's own Stop hook). The existing `/status` dashboard and `status_collector.py` are extended with two new collectors and panels — zero regression. Named Routines are `.claude/commands/` slash commands wrapping the scripts.

**Tech Stack:** Python 3 (stdlib + existing `pipelines/shared`), pytest, FastAPI + HTMX + React-CDN (existing admin app), Markdown, Claude Code hooks (settings.json) + slash commands, cron (Mac Mini).

**Pre-execution findings that shaped this plan (verified 2026-05-31):**
- claude-mem WORKS (:37777, context injection confirmed) → hot.md is **additive**, claude-mem untouched.
- AR git push is **NOT broken** — remote already SSH, pushes clean. **No git-remote fix task.**
- The Loop (audit 6:00/12:00 + AR 6:15/12:15) runs on **Mac Mini cron**, not VPS → refresh trigger is a **direct same-host call**, not n8n (n8n is VPS-side; cross-host round-trip rejected for fragility). Mac-Mini cron is in-scope; VPS crontab is not touched.
- `docs/.obsidian` already exists + `.obsidian/` already gitignored → OKL at `docs/operator-wiki/` joins the existing docs vault, zero new setup, zero contamination (**ADR-005**).
- agent_events.db schema is `agent`/`pipeline`. CIS-v2 stages (faithfulness, editorial_review, structural_editor, citation_binding, editorial_apply, assemble_final, publish_prep) have real records but are **sequential sub-stages → pipeline-flow panel, not 7 new minions.** Keep the 6 existing minions.

**Principles Alignment:** §2 Default to Autonomous (self-maintaining cache, no operator polling) — primary driver. §6 Traceable by Default (every hot.md section cites its source; generator is idempotent + logged). §5 $0 Until Proven (direct cron call, no new infra/n8n). §0 Truth Over Closure (generator marks stale sources explicitly rather than emitting confident stale data; hot.md carries a `Validity` line). §7 Operable by Non-Technical Operator (Discord alerts, Obsidian visualization, no CLI needed day-to-day). §9 Machine-Readable (OKL is plain Markdown). No tensions identified.

**Scope limits (HARD — from dispatch):** Do NOT touch `pipelines/orchestrator/` (content drainer), VPS crontab, AR quarantine branches, `data/orchestrator_state.json`, or any content-pipeline agent code. AR's `cycle.py`/`executor.py` are read for understanding but **not modified** — the Loop is extended via an independent cron post-step, not by editing tested AR code. Karpathy-8: every changed line traces to this plan.

---

## File Structure

**New — OKL (`docs/operator-wiki/wiki/`):**
- `hot.md` — the hot cache (generated; <250 lines; 7 marked sections)
- `index.md` — master catalog of OKL entries
- `log.md` — append-only operation log
- `decisions/ADR-001..005.md` — architectural decision records (seeded from existing docs)
- `context/the-loop.md`, `session-surfaces.md`, `stack-roles.md`, `active-specs.md` — background context
- `sessions/.gitkeep` — session archive (filled by Stop hook / handoff over time)
- `README.md` — what the OKL is + how Obsidian opens it

**New — generator + orchestrator:**
- `scripts/update_hot_cache.py` — marker-based hot.md synthesizer (TDD)
- `scripts/install-governance-cron.sh` — idempotent Mac-Mini cron installer (post-AR refresh + orchestrator)
- `pipelines/governance/__init__.py`
- `pipelines/governance/orchestrator.py` — staleness/threshold monitor → Discord (TDD)
- `pipelines/governance/hot_cache.py` — shared synthesis helpers imported by both generator + orchestrator (TDD)
- `pipelines/governance/tests/test_hot_cache.py`, `test_orchestrator.py`, `test_update_hot_cache.py`

**New — Routines (slash commands):**
- `.claude/commands/governance-cycle-check.md`
- `.claude/commands/update-hot-cache.md`
- `.claude/commands/new-decision.md`
- `.claude/commands/loop-status.md`

**Modified:**
- `.claude/settings.json` — add SessionStart `cat hot.md` hook + Stop staleness-update prompt (additive; existing hooks preserved). **NOTE: `.claude/` is gitignored (`.gitignore:33`) — settings.json is untracked + host-local (only the Mac Mini runs CC). Edit in place; do NOT commit. Durability: the exact hook JSON is documented in `docs/operator-wiki/wiki/context/session-surfaces.md` so it can be re-applied if `.claude/` is ever regenerated.**
- `pipelines/admin/status_collector.py` — add `collect_okl()`, `collect_loop()`, CIS-v2 flow data (additive)
- `pipelines/admin/app.py` — add new keys to `/status/json` (additive)
- `pipelines/admin/templates/status.html` (or the React block) — OKL panel, Loop panel, CIS-v2 flow strip, HTMX 60s poll (additive)
- `.gitignore` — confirm `docs/operator-wiki/**/.obsidian/` covered (already by `.obsidian/`)
- `docs/strategy/geo-codex-v2.0.md` §4.1 — register OKL Spec v1.0
- `CLAUDE.md` PART 2 — session-log row + banner note
- `docs/operations/state-board.md` — Cowork-owned; CC drafts the delta, does not edit directly (per state-board-spec)
- `docs/governance/cowork-operating-discipline.md` — NOT edited here; §9.0 amendment auto-drafted as a ready-to-send CC dispatch

---

## Phase A — OKL Foundation + Hot-Cache Generator (must complete before B–E)

### Task A1: Scaffold the OKL vault

**Files:**
- Create: `docs/operator-wiki/wiki/{decisions,context,sessions}/` dirs, `docs/operator-wiki/wiki/sessions/.gitkeep`, `docs/operator-wiki/README.md`

- [ ] **Step 1:** Create the directory tree and `.gitkeep`:
```bash
mkdir -p docs/operator-wiki/wiki/decisions docs/operator-wiki/wiki/context docs/operator-wiki/wiki/sessions
touch docs/operator-wiki/wiki/sessions/.gitkeep
```
- [ ] **Step 2:** Write `docs/operator-wiki/README.md` — 1 paragraph: "This is the Operator Knowledge Layer (OKL), the build-governance analog of the content KCL. Open `docs/` as an Obsidian vault (already configured) to visualize. `wiki/hot.md` is the hot cache — read it first. It is auto-generated; do not hand-edit (changes are overwritten by `scripts/update_hot_cache.py`)."
- [ ] **Step 3:** Verify structure: `find docs/operator-wiki -type d` shows the 4 dirs.
- [ ] **Step 4:** Commit: `git add docs/operator-wiki && git commit -m "feat(okl): scaffold operator knowledge layer vault structure"`

### Task A2: hot.md section-marker contract + template

**Files:**
- Create: `docs/operator-wiki/wiki/hot.md` (initial template with markers)

The marker contract is THE interface the generator and all readers depend on. Each section is wrapped:
```
<!-- BEGIN:focus -->
## Current Build Focus
...
<!-- END:focus -->
```
Sections (in order): `focus`, `decisions`, `dispatch`, `open-decisions`, `dont-touch`, `system-state`, `loop`.

- [ ] **Step 1:** Write `hot.md` with the header (title, `**Last updated:**` line, `**Validity:** Treat as stale if >24h old. Read state board for depth.`) and all 7 marker-wrapped sections, each with a placeholder line `_(pending first generation)_`.
- [ ] **Step 2:** Verify 7 sections present:
```bash
grep -c "<!-- BEGIN:" docs/operator-wiki/wiki/hot.md   # Expected: 7
```
- [ ] **Step 3:** Commit: `git add docs/operator-wiki/wiki/hot.md && git commit -m "feat(okl): hot.md section-marker template (7 sections)"`

### Task A3: `pipelines/governance/hot_cache.py` — source readers (TDD)

**Files:**
- Create: `pipelines/governance/__init__.py`, `pipelines/governance/hot_cache.py`
- Test: `pipelines/governance/tests/__init__.py`, `pipelines/governance/tests/test_hot_cache.py`

`hot_cache.py` exposes pure reader functions, each returning a Markdown string for one section. Each tolerates a missing/empty source by returning a `_⚠ source unavailable_` marker (Principle 0).

Interface:
```python
def read_focus(state_board_path: Path, git_log: list[str]) -> str
def read_recent_decisions(adr_dir: Path, git_log: list[str], n: int = 7) -> str
def read_dispatch_queue(okl_root: Path) -> str          # reads sessions/ + a dispatch tracker; empty-safe
def read_open_decisions(state_board_path: Path) -> str   # parses state board "open decisions" section
def read_dont_touch(state_board_path: Path) -> str
def read_system_state(status_json: dict | None) -> str   # from status_collector snapshot
def read_loop(ar_state: dict | None, latest_audit_path: Path | None) -> str
def git_log_lines(repo_root: Path, n: int = 30) -> list[str]   # subprocess git log --oneline --date
```

- [ ] **Step 1: Write the failing test** for `read_loop` (most contract-critical — feeds the autonomous trigger):
```python
# pipelines/governance/tests/test_hot_cache.py
from pathlib import Path
from pipelines.governance import hot_cache

def test_read_loop_formats_ar_state_and_audit(tmp_path):
    audit = tmp_path / "2026-05-30-traceability-audit-run12.md"
    audit.write_text("# Run 12\n9 drift / 0 violations\n")
    ar_state = {
        "quarantine_status": "quarantined", "cycles_in_window": 1,
        "last_cycle_ts": "2026-05-31T16:15:01+00:00", "last_cycle_outcome": "audit_stale",
    }
    out = hot_cache.read_loop(ar_state, audit)
    assert "run12" in out.lower() or "run 12" in out.lower()
    assert "audit_stale" in out
    assert "quarantined" in out

def test_read_loop_marks_missing_sources():
    out = hot_cache.read_loop(None, None)
    assert "⚠" in out   # Principle 0: explicit unavailability, not fabricated state
```
- [ ] **Step 2: Run to verify it fails:** `python3 -m pytest pipelines/governance/tests/test_hot_cache.py -v` → FAIL (module not found).
- [ ] **Step 3: Implement** `hot_cache.py` minimally to pass — `read_loop` first, then the other readers. Reuse `pipelines/shared/config.py` for paths where they exist. Use `subprocess` for git (60s timeout, `cwd=PROJECT_ROOT`).
- [ ] **Step 4: Run:** `python3 -m pytest pipelines/governance/tests/test_hot_cache.py -v` → PASS.
- [ ] **Step 5:** Add tests for the remaining readers (focus, recent_decisions, open_decisions, dont_touch, system_state) — each with a happy-path + a missing-source assertion. Implement to green.
- [ ] **Step 6: Commit:** `git add pipelines/governance && git commit -m "feat(governance): hot_cache source readers with missing-source safety (TDD)"`

### Task A4: `scripts/update_hot_cache.py` — marker-based writer (TDD)

**Files:**
- Create: `scripts/update_hot_cache.py`
- Test: `pipelines/governance/tests/test_update_hot_cache.py`

CLI: `python3 scripts/update_hot_cache.py --section [all|focus|decisions|dispatch|open-decisions|dont-touch|system-state|loop]` (default `all`). Replaces ONLY content between the named `<!-- BEGIN:x -->`/`<!-- END:x -->` markers; updates the `**Last updated:**` line; leaves other sections byte-identical. Idempotent. Exit 0 on success; exit 2 if `hot.md` missing; never crashes on a missing source (writes the ⚠ marker).

- [ ] **Step 1: Write the failing test** — partial update preserves other sections:
```python
# pipelines/governance/tests/test_update_hot_cache.py
import subprocess, sys
from pathlib import Path

def test_section_update_preserves_others(tmp_path, monkeypatch):
    hot = tmp_path / "hot.md"
    hot.write_text(
        "# Hot\n**Last updated:** never\n"
        "<!-- BEGIN:loop -->\nOLD LOOP\n<!-- END:loop -->\n"
        "<!-- BEGIN:focus -->\nKEEP FOCUS\n<!-- END:focus -->\n"
    )
    from scripts import update_hot_cache as u
    u.update_section(hot, "loop", "NEW LOOP")
    text = hot.read_text()
    assert "NEW LOOP" in text and "OLD LOOP" not in text
    assert "KEEP FOCUS" in text            # other section untouched
    assert "**Last updated:** never" not in text   # timestamp line refreshed
```
- [ ] **Step 2: Run to verify fail:** `python3 -m pytest pipelines/governance/tests/test_update_hot_cache.py -v` → FAIL.
- [ ] **Step 3: Implement** `update_section(path, name, body)` (regex on the marker pair, refresh timestamp) and a `main()` dispatching `--section` → the matching `hot_cache.read_*` + `update_section`. `--section all` runs every section.
- [ ] **Step 4: Run:** PASS. Then run it for real against the live tree:
```bash
python3 scripts/update_hot_cache.py --section all
grep -c "<!-- BEGIN:" docs/operator-wiki/wiki/hot.md   # still 7
wc -l docs/operator-wiki/wiki/hot.md                   # < 250
```
- [ ] **Step 5: Idempotency check:** run twice, diff is only the timestamp line:
```bash
python3 scripts/update_hot_cache.py --section all && cp docs/operator-wiki/wiki/hot.md /tmp/h1
python3 scripts/update_hot_cache.py --section all && diff /tmp/h1 docs/operator-wiki/wiki/hot.md | grep -v "Last updated" | grep -c "^[<>]"  # Expected: 0
```
- [ ] **Step 6: Commit:** `git add scripts/update_hot_cache.py pipelines/governance/tests && git commit -m "feat(okl): marker-based hot.md generator (idempotent, TDD)"`

---

## Phase B — OKL Content (ADRs + context entries) — parallelizable after A

> These are documentation-synthesis tasks. Verification is **structural + accuracy-against-source**, not pytest (prose has no unit tests — stating this honestly per Principle 0). Each ADR follows MADR-lite: Context · Decision · Status · Consequences · Source. Each must be accurate to the cited source doc — the implementer reads the source before writing.

### Task B1: ADR-001..004 (seeded from existing decisions)

**Files:** Create `docs/operator-wiki/wiki/decisions/ADR-00{1,2,3,4}.md`

Sources to read and distill (do not invent — cite the source path in each ADR):
- ADR-001 Orchestrator drainer (Pattern b): `CLAUDE.md` session-log row C2 + `pipelines/orchestrator/README` + spike verdict `8ec23bf`.
- ADR-002 Ruflo routing failure / single-host: `docs/audits/2026-05-19-ruflo-c1a-capability-check.md` + Tool Registry §1.
- ADR-003 Cross-host git-push-as-message-bus: `docs/specs/cross-host-coordination-mechanism-spec-v1.0.md`.
- ADR-004 CIS-v2 pipeline merge: `CLAUDE.md` session-log row CISv2 + `docs/audits/2026-05-24-traceability-audit.md`.

- [ ] **Step 1:** For each ADR, read the cited source(s), then write the ADR (Context/Decision/Status/Consequences/Source, ≤60 lines each). Link related entries with `[[ADR-00x]]` / `[[the-loop]]`.
- [ ] **Step 2:** Accuracy check — for each ADR, the Decision + Status must match the source. Spot-grep a known fact:
```bash
grep -l "git-push-as-message-bus" docs/operator-wiki/wiki/decisions/ADR-003.md
grep -l "81b4f73\|Pattern b" docs/operator-wiki/wiki/decisions/ADR-001.md
```
- [ ] **Step 3: Commit:** `git add docs/operator-wiki/wiki/decisions && git commit -m "docs(okl): ADR-001..004 seeded from existing governance decisions"`

### Task B2: ADR-005 (this build's vault decision) + index.md + log.md

**Files:** Create `docs/operator-wiki/wiki/decisions/ADR-005-operator-wiki.md`, `docs/operator-wiki/wiki/index.md`, `docs/operator-wiki/wiki/log.md`

- [ ] **Step 1:** Write ADR-005. **Decision:** OKL lives in-repo at `docs/operator-wiki/`. **Context:** evaluated in-repo vs separate vault vs hybrid against Codex §1.2 (durability > reliability > effectiveness). **Rationale (evidence):** `docs/.obsidian` already exists (operator already opens docs/ as a vault); `.obsidian/` already gitignored (`.gitignore:75`) so no config contamination; in-repo = same git cross-host sync as all other state (durability via git history, reliability via existing fabric, effectiveness via existing Obsidian graph). Separate/hybrid rejected: fragments from the existing docs vault, loses markdown versioning, adds sync surface for no gain. **Consequences:** OKL markdown is versioned + cross-host; Obsidian config stays local-only.
- [ ] **Step 2:** Write `index.md` — a catalog table linking every OKL entry (hot.md, 5 ADRs, 4 context entries) with a one-line description each.
- [ ] **Step 3:** Write `log.md` — append-only header + first entry: `YYYY-MM-DD · OKL initialized · agentic-os-v1 build`.
- [ ] **Step 4:** Verify: `grep -c "docs/.obsidian\|gitignore" docs/operator-wiki/wiki/decisions/ADR-005-operator-wiki.md` ≥ 1.
- [ ] **Step 5: Commit:** `git add docs/operator-wiki/wiki && git commit -m "docs(okl): ADR-005 vault decision + index + log"`

### Task B3: Four context entries

**Files:** Create `docs/operator-wiki/wiki/context/{the-loop,session-surfaces,stack-roles,active-specs}.md`

- [ ] **Step 1:** `the-loop.md` — how governance enforcement works end-to-end: audit (Mac Mini cron 6:00/12:00) → AR cycle (6:15/12:15, `pipelines/auto_remediation/`) → Discord digest → (new) hot.md refresh. Note quarantine state + graduation. Cite `docs/specs/auto-remediation-spec-v1.0.md`.
- [ ] **Step 2:** `session-surfaces.md` — the 3 Claude surfaces (CC, Cowork, strategic advisor), their memory mechanisms (CC: claude-mem :37777 + hot.md; Cowork: state board + hot.md; advisor: hot.md via Project instructions), from the v2 spec Part 2 table.
- [ ] **Step 3:** `stack-roles.md` — the content-pipeline-vs-build-governance role table for each component (LightRAG, n8n, Obsidian, SQLite, Discord, claude-mem, GitHub, drainer, Ruflo) from v2 spec Part 5.
- [ ] **Step 4:** `active-specs.md` — current sub-specs + status, distilled from CLAUDE.md Strategic Authority Hierarchy "Active sub-specs".
- [ ] **Step 5:** Verify all 4 exist + cross-link to `[[hot]]`/`[[ADR-001]]` etc: `ls docs/operator-wiki/wiki/context/ | wc -l` → 4.
- [ ] **Step 6: Commit:** `git add docs/operator-wiki/wiki/context && git commit -m "docs(okl): four context entries (the-loop, surfaces, stack-roles, specs)"`

### Task B4: Regenerate hot.md from real sources

- [ ] **Step 1:** Now that ADRs/context exist, run the generator for real: `python3 scripts/update_hot_cache.py --section all`.
- [ ] **Step 2:** Read `hot.md` end-to-end. Confirm: Current Build Focus is accurate (CIS-v2 merged, articles offline pending remediation), Last 7 Decisions reflects the ADRs + recent git, The Loop shows run12 + audit_stale, no fabricated facts. Hand-correct any source-reader formatting bug in `hot_cache.py` (re-run tests).
- [ ] **Step 3: Commit:** `git add docs/operator-wiki/wiki/hot.md && git commit -m "feat(okl): first real hot.md generation from live sources"`

---

## Phase C — Triggers, Orchestrator, Session Injection, Dashboard — parallelizable after A (B not required for C1/C2/C3)

### Task C1: Session injection hooks (settings.json)

**Files:** Modify `.claude/settings.json`

Additive only — every existing hook (claude-flow handler chain, claude-mem import/sync) is preserved. Add to the EXISTING `SessionStart` hooks array a fast command hook that prints hot.md; add a `Stop` staleness-gated update prompt.

- [ ] **Step 1:** Back up: `cp .claude/settings.json /tmp/settings.bak.json`.
- [ ] **Step 2:** Add to `SessionStart.hooks[]` (after the existing two):
```json
{ "type": "command", "command": "sh -c 'cat \"${CLAUDE_PROJECT_DIR:-.}/docs/operator-wiki/wiki/hot.md\" 2>/dev/null || true'", "timeout": 5000 }
```
- [ ] **Step 3:** Add to the EXISTING `Stop.hooks[]` (after the auto-memory sync) a staleness-gated reminder (does NOT auto-rewrite — the orchestrator owns autonomous rewrite; this only nudges CC if it did governance work and hot.md is stale):
```json
{ "type": "command", "command": "sh -c 'f=\"${CLAUDE_PROJECT_DIR:-.}/docs/operator-wiki/wiki/hot.md\"; if [ -f \"$f\" ] && [ -n \"$(find \"$f\" -mmin +360 2>/dev/null)\" ]; then echo \"hot.md >6h stale — run /update-hot-cache if this session changed governance state\"; fi'", "timeout": 4000 }
```
- [ ] **Step 4:** Validate JSON parses: `python3 -c "import json; json.load(open('.claude/settings.json'))" && echo OK`.
- [ ] **Step 5:** Verify hot.md hook present: `grep -c "operator-wiki/wiki/hot.md" .claude/settings.json` ≥ 1.
- [ ] **Step 6:** Confirm the SessionStart cat command works standalone: `sh -c 'cat docs/operator-wiki/wiki/hot.md' | head -3`.
- [ ] **Step 7: Do NOT commit settings.json** (`.claude/` is gitignored, file is host-local + untracked). Instead, record the two exact hook JSON snippets in `docs/operator-wiki/wiki/context/session-surfaces.md` under a "Session hooks (Mac Mini)" heading so the config is recoverable, and commit THAT doc: `git add docs/operator-wiki/wiki/context/session-surfaces.md && git commit -m "docs(okl): document session-injection hooks for durability (.claude is gitignored)"`

> Note: live "fires on next SessionStart" verification happens on the next real CC session (acknowledged in dispatch next-steps). This task verifies the hook config parses + the command it runs works. The hook itself lives only in the Mac Mini `.claude/settings.json` (gitignored) — recoverable from the documented snippet.

### Task C2: Governance orchestrator (TDD)

**Files:** Create `pipelines/governance/orchestrator.py`; Test `pipelines/governance/tests/test_orchestrator.py`

Cron-fired (Mac Mini `0 */6 * * *`). Four checks, each independent and Discord-noisy only when a threshold is crossed:
1. hot.md staleness >6h → regenerate via `update_hot_cache.main(['--section','loop,system-state'])` (autonomous refresh — the reliable baseline).
2. AR graduation progress → weekly (only on the first 6h-run of Monday) Discord post `N/10 cycles, D days left`.
3. health_score < threshold (default 70) → Discord alert.
4. Open decisions aging >3 days → Discord reminder.

Posts via existing `pipelines/shared/discord_notify` to `DISCORD_WEBHOOK_COMMANDS`. Does NOT touch content pipeline.

- [ ] **Step 1: Write the failing test** for the staleness→regenerate branch (mock the generator + filesystem mtime):
```python
# pipelines/governance/tests/test_orchestrator.py
from unittest import mock
from pipelines.governance import orchestrator

def test_stale_hot_cache_triggers_regen(tmp_path):
    hot = tmp_path / "hot.md"; hot.write_text("x")
    # force mtime 7h old
    import os, time; old = time.time() - 7*3600; os.utime(hot, (old, old))
    with mock.patch.object(orchestrator, "regenerate_hot_cache") as regen:
        orchestrator.check_hot_cache_staleness(hot, max_age_h=6)
        regen.assert_called_once()

def test_fresh_hot_cache_no_regen(tmp_path):
    hot = tmp_path / "hot.md"; hot.write_text("x")   # fresh mtime
    with mock.patch.object(orchestrator, "regenerate_hot_cache") as regen:
        orchestrator.check_hot_cache_staleness(hot, max_age_h=6)
        regen.assert_not_called()
```
- [ ] **Step 2: Run → FAIL.**
- [ ] **Step 3: Implement** the four `check_*` functions + a `main()` that runs all four and logs an `agent_events` row via `pipelines/shared/event_logger.log_event(agent='governance', pipeline='orchestrator', ...)`. Each Discord post guarded so a check failure can't crash the others (try/except per check).
- [ ] **Step 4: Run → PASS.** Add tests for the health-threshold branch + the open-decisions-age branch (mock Discord, assert called/not-called).
- [ ] **Step 5: Dry-run live** (no Discord spam — add a `--dry-run` flag that prints instead of posting):
```bash
python3 -m pipelines.governance.orchestrator --dry-run
```
Expected: prints each check's verdict, exits 0.
- [ ] **Step 6: Commit:** `git add pipelines/governance/orchestrator.py pipelines/governance/tests/test_orchestrator.py && git commit -m "feat(governance): orchestrator — staleness/graduation/health/decision-age monitor (TDD)"`

### Task C3: Dashboard — OKL collector + Loop collector (TDD)

**Files:** Modify `pipelines/admin/status_collector.py`; Test `pipelines/admin/tests/test_status_collector_governance.py` (create if dir absent)

- [ ] **Step 1: Write failing tests** for two new collectors returning the documented shape:
```python
def test_collect_okl_shape(tmp_path):
    from pipelines.admin import status_collector as sc
    out = sc.collect_okl(okl_hot=tmp_path/"hot.md")   # missing file → safe defaults
    assert set(out) >= {"hot_cache_age_hours", "current_focus", "dispatch_queue", "open_decisions_count"}

def test_collect_loop_shape():
    from pipelines.admin import status_collector as sc
    out = sc.collect_loop()
    assert set(out) >= {"last_audit_run", "ar_outcome", "graduation_cycles", "next_audit"}
```
- [ ] **Step 2: Run → FAIL.**
- [ ] **Step 3: Implement** `collect_okl()` (reads hot.md mtime → age, parses the `focus` marker for current_focus, counts open-decisions) and `collect_loop()` (reads `data/auto_remediation_state.json` + latest `docs/audits/*traceability*` filename + cron next-fire). Both missing-source-safe.
- [ ] **Step 4: Run → PASS.** Run the FULL existing collector suite to prove zero regression:
```bash
python3 -m pytest pipelines/admin/tests/ -v   # all pre-existing PASS + 2 new PASS
```
- [ ] **Step 5: Commit:** `git add pipelines/admin/status_collector.py pipelines/admin/tests && git commit -m "feat(dashboard): OKL + Loop status collectors (TDD, zero regression)"`

### Task C4: Dashboard — wire collectors into /status/json + render panels (front-end iteration, not TDD)

**Files:** Modify `pipelines/admin/app.py`, `pipelines/admin/templates/status.html` (or the React block serving `/status`)

> Front-end exception (CLAUDE.md §5): iterate visually, verify by running the app locally. No RED-GREEN here.

- [ ] **Step 1:** In `app.py`, add `okl`, `loop`, and `cisV2Flow` keys to the `/status/json` payload (call the new collectors). Confirm existing keys unchanged.
- [ ] **Step 2:** Run the admin app locally to verify JSON:
```bash
# from repo root, with env sourced
python3 -m uvicorn pipelines.admin.app:app --port 8099 &  # or the app's documented run cmd
curl -s localhost:8099/status/json | python3 -c "import json,sys; d=json.load(sys.stdin); print('okl' in d, 'loop' in d, len(d['agents']))"
```
Expected: `True True 6` (6 minions preserved).
- [ ] **Step 3:** Add three panels to the status page, matching existing panel markup/Neon-Precision styling: **OKL Panel** (hot-cache age badge green/amber/red, current focus, dispatch queue table, open-decisions count), **Loop Status Panel** (last audit run, AR outcome, graduation N/10 + days, next audit), **CIS-v2 Flow strip** (the 7 sub-stages as a horizontal pipeline-flow with last-run + count per stage from agent_events — NOT new minions). Add HTMX `hx-trigger="every 60s"` on the OKL + Loop panels only.
- [ ] **Step 4:** Visually verify locally: load `localhost:8099/status`, confirm 6 minions still animate, all 11 original panels render, 3 new panels render, no console errors. Screenshot for the operator.
- [ ] **Step 5: Regression gate (STOP condition):** confirm every original panel still present. If any original panel fails → revert this task's changes, halt.
- [ ] **Step 6: Commit:** `git add pipelines/admin && git commit -m "feat(dashboard): OKL panel + Loop panel + CIS-v2 flow strip (6 minions preserved, zero regression)"`

---

## Phase D — Loop Extension Cron + Named Routines (after A; C2/C4 inform D1)

### Task D1: Autonomous refresh cron (post-AR) + orchestrator cron — install script

**Files:** Create `scripts/install-governance-cron.sh` (idempotent, mirrors existing `scripts/install-*-cron.sh`)

Adds TWO Mac-Mini cron lines (idempotent — grep-guarded, never double-adds):
- `20 6,12 * * *` → `update_hot_cache.py --section loop,system-state` (5 min after AR's :15 — the "on Loop completion" refresh, decoupled from AR code).
- `0 */6 * * *` → `python3 -m pipelines.governance.orchestrator` (the monitor).

- [ ] **Step 1:** Write `install-governance-cron.sh` following the existing installer pattern (read `crontab -l`, append only if the marker comment absent, re-install cleanly). Include `SHELL=/bin/bash` guard if not already set (the dash/`source` lesson from the Scrapy cron audit — though these lines use `/usr/bin/python3` directly, so no `source`).
- [ ] **Step 2:** Dry-run print what it WOULD add (`--dry-run`), review, then it is an **operator-gated install** — the script is committed but RUNNING it on the Mac Mini is surfaced to the operator (system-state change). Default: do not auto-install; report the exact command.
- [ ] **Step 3:** Verify script is idempotent by static review + shellcheck if available: `bash -n scripts/install-governance-cron.sh`.
- [ ] **Step 4: Commit:** `git add scripts/install-governance-cron.sh && git commit -m "feat(governance): idempotent Mac-Mini cron installer (post-AR refresh + orchestrator)"`

### Task D2: Named Routines (slash commands)

**Files:** Create `.claude/commands/{governance-cycle-check,update-hot-cache,new-decision,loop-status}.md`

Each is a Claude Code slash command (the durable Routine mechanism — no dependency on a separate Routines feature, resolving the dispatch checklist item).

- [ ] **Step 1:** `update-hot-cache.md` — instructs CC to run `python3 scripts/update_hot_cache.py --section all`, then `git add docs/operator-wiki/wiki/hot.md && git commit && git push`.
- [ ] **Step 2:** `loop-status.md` — read `data/auto_remediation_state.json` + latest audit file + cron, print a Loop status summary.
- [ ] **Step 3:** `governance-cycle-check.md` — read hot.md + state board + `git log --oneline -10`, produce a 3-sentence current-state summary.
- [ ] **Step 4:** `new-decision.md` — scaffold the next `ADR-00N.md` from the MADR-lite template, append a row to `index.md`.
- [ ] **Step 5:** Verify all 4 exist: `ls .claude/commands/{governance-cycle-check,update-hot-cache,new-decision,loop-status}.md`.
- [ ] **Step 6: Commit with `-f`** (`.claude/` is gitignored, but existing commands like `claude-flow-help.md` are tracked — match that by force-adding so the Routines are recoverable in git): `git add -f .claude/commands/governance-cycle-check.md .claude/commands/update-hot-cache.md .claude/commands/new-decision.md .claude/commands/loop-status.md && git commit -m "feat(governance): named Routines as slash commands (force-add; .claude gitignored)"`

---

## Phase E — Governance Close

### Task E1: Codex §4.1 registration + CLAUDE.md PART 2

**Files:** Modify `docs/strategy/geo-codex-v2.0.md` (§4.1 registry), `CLAUDE.md` (PART 2 session log + banner)

- [ ] **Step 1:** Add an OKL Spec v1.0 row to Codex §4.1 governed-document registry pointing at the OKL + ADR-005.
- [ ] **Step 2:** Add a CLAUDE.md PART 2 session-log row (Agentic OS v1.0 — what shipped, commit range) and a one-line banner note.
- [ ] **Step 3:** Verify: `grep -c "Operator Knowledge Layer" docs/strategy/geo-codex-v2.0.md` ≥ 1.
- [ ] **Step 4: Commit:** `git add docs/strategy/geo-codex-v2.0.md CLAUDE.md && git commit -m "docs(governance): register OKL Spec v1.0 in Codex §4.1 + CLAUDE.md PART 2"`

### Task E2: hot.md final regen + state-board delta + Cowork §9.0 amendment dispatch (auto-drafted)

**Files:** Modify `docs/operator-wiki/wiki/hot.md` (regen); Create `docs/operations/cc-dispatch-cowork-discipline-s9-amendment.md` (ready-to-send draft); draft state-board delta (do NOT edit state-board.md — Cowork owns it)

- [ ] **Step 1:** Final `python3 scripts/update_hot_cache.py --section all` so hot.md reflects the completed build.
- [ ] **Step 2:** Write the Cowork §9.0 amendment as a ready-to-send CC dispatch: add §9.0 "First action every session: read `docs/operator-wiki/wiki/hot.md`; if >24h stale, read state board instead" + §9.3 session-end "run /update-hot-cache". Self-contained dispatch file.
- [ ] **Step 3:** Draft the state-board delta (new OKL/Loop visibility row) as a snippet in the halt report for Cowork to apply — per state-board-spec, CC does not edit the board directly.
- [ ] **Step 4: Commit:** `git add docs/operator-wiki/wiki/hot.md docs/operations/cc-dispatch-cowork-discipline-s9-amendment.md && git commit -m "docs(governance): final hot.md regen + auto-drafted Cowork §9 amendment dispatch"`

### Task E3: Full verification suite + finishing-a-development-branch

- [ ] **Step 1:** Run the dispatch's minimum verification suite (hot.md 7 sections, <250 lines, settings.json hook present, orchestrator exists, generator runs clean, dashboard 200, remote SSH, Codex OKL row, Routines present) + the full pytest suites (`pipelines/governance/tests`, `pipelines/admin/tests`). Capture all outputs.
- [ ] **Step 2:** Confirm zero regression: original 11 dashboard panels + 6 minions intact; existing admin tests still pass.
- [ ] **Step 3:** Invoke `superpowers:finishing-a-development-branch` to choose integration path (operator gates merge to main per Global Design Principle 2).
- [ ] **Step 4:** Push the branch; surface the halt report (per dispatch "Halt and Report" — 8 items).

---

## Self-Review

**Spec coverage (8 "What Must Ship" items):** OKL → A+B ✓ · self-maintaining hot cache → A4+D1 ✓ · session injection → C1 ✓ · extended Loop → C2+D1 ✓ · dashboard extension → C3+C4 ✓ · governance orchestrator → C2 ✓ · Named Routines → D2 ✓ · governance close → E ✓. All 8 covered.

**Open decisions:** ADR-005 vault → B2 ✓ · claude-mem (keep, untouched) → C1 note ✓ · minion expansion (flow strip, 6 minions) → C4 ✓ · n8n→direct → D1 ✓.

**Placeholder scan:** test code is concrete; prose tasks (B) verified structurally + against named sources; no "TBD"/"handle edge cases" left.

**Type consistency:** `collect_okl`/`collect_loop` keys in C3 tests match the C4 consumers; `update_section(path,name,body)` signature in A4 matches the orchestrator's regen call in C2; section marker names (`focus/decisions/dispatch/open-decisions/dont-touch/system-state/loop`) consistent across A2/A3/A4/C3.

**Divergences from spec, flagged:** (1) n8n → direct same-host cron (Loop is Mac-Mini-local); (2) AR-git-push fix dropped (not broken); (3) CIS-v2 as flow strip not minions; (4) Stop hook nudges rather than auto-rewrites (orchestrator owns autonomous rewrite). All within dispatch-delegated latitude.

**Risk notes:** dashboard local-run command (C4 Step 2) assumed `uvicorn pipelines.admin.app:app` — implementer confirms the app's actual run entrypoint first. AR code is read-only throughout (scope limit honored). Cron install is operator-gated (system-state change surfaced).
