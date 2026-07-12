# Generalization notes — agentic-os-dispatch.md

> Notes ABOUT the plan, written at PGK codification time (2026-07-11). Not part of the plan. A map for activation, not a rewrite. NOTE: this is an *implementation plan* (task-by-task build dispatch), not a ratified spec — it is the most complete record of how the orchestrator/hot-cache layer was actually built, which is why it's the reference copy.

## The portable pattern

- **An operator knowledge layer (OKL) as an in-repo markdown wiki:** hot cache + ADR index + append-only log + context entries + session archive. In-repo because git already provides versioning, sync, and durability — a separate store adds surface for no gain.
- **The hot cache:** a single generated orientation file (<250 lines) with marker-wrapped sections (`<!-- BEGIN:x -->`/`<!-- END:x -->`), a `Last updated` line and an explicit `Validity: treat as stale if >24h old` line. Fresh sessions orient in under 30 seconds. It is **orientation, not authorization** — same authority rule as the state board; when it conflicts with an authoritative doc, the doc wins.
- **The generator:** idempotent (re-run diff = timestamp line only), section-scoped updates that leave other sections byte-identical, and — load-bearing honesty rule — a missing/empty source produces an explicit `⚠ source unavailable` marker, never fabricated state.
- **Three refresh triggers, layered:** a baseline cron orchestrator (the reliable floor), a post-audit-cycle refresh (closes the gap between a governance change and the next scheduled refresh), and session hooks (SessionStart prints the cache; Stop nudges — nudges, doesn't auto-rewrite; the orchestrator owns autonomous rewrite).
- **The governance orchestrator:** four independent checks, each Discord-noisy only on threshold crossing — cache staleness → regenerate; graduation progress → weekly note; health score below threshold → alert; open operator decisions aging past N days → reminder. Checks isolated so one failure can't crash the others.
- **Named routines as slash commands** (update-cache / loop-status / cycle-check / new-decision) — durable, no separate scheduler feature required.
- **Process discipline worth stealing:** verify-before-planning (the plan's "pre-execution findings" section killed two speculative tasks — a git fix for a git that wasn't broken, and a cross-host round-trip rejected for fragility); hard scope limits with every-changed-line-traces-to-plan; TDD for every generator/orchestrator component; additive-only changes to shared surfaces with explicit zero-regression gates.

## GEO-specific — strip or replace on activation

- **All paths:** `docs/operator-wiki/`, `pipelines/governance/`, `scripts/update_hot_cache.py` → your layout. The 7 section names (focus / decisions / dispatch / open-decisions / dont-touch / system-state / loop) map to your board's sections — keep the marker contract, rename freely.
- **Host topology** (Mac Mini vs VPS, which cron lives where), **claude-mem**, **Obsidian vault** setup, `.claude/settings.json` being gitignored-but-documented — environment-specific. Keep the durability trick: host-local config is documented in a committed doc so it's recoverable.
- **The entire dashboard phase** (Phases C3/C4: FastAPI admin app, status_collector, minions, HTMX panels) — only meaningful if your project has a dashboard. Skippable wholesale.
- **ADR-001..005 seeds and context entries** (the-loop, stack-roles, etc.) — GEO content; your OKL seeds from your own decision history.
- **Discord** → your notification channel; **agent_events.db** event logging → your telemetry, or drop.
- **Superpowers skill invocations** (brainstorming, TDD, finishing-a-development-branch) — GEO's workflow tooling; keep the discipline, use whatever enforces it in your environment.
