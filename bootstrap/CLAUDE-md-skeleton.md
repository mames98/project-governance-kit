# CLAUDE.md skeleton — PGK wiring

Fill-in skeleton for the project's `CLAUDE.md` (or equivalent session-bootstrap file). This is how "the one rule" gets wired in: whatever file your Claude sessions load first must carry the board-read directive. Fill every `[PLACEHOLDER]`, then use the content below the line as your `CLAUDE.md` — or, if a `CLAUDE.md` already exists, merge PART 1 and PART 2 into it **additively**; never overwrite what's there.

Placeholders:

- `[PROJECT_NAME]` — the project.
- `[STATE_BOARD_PATH]` — where the state board lives (e.g. `docs/operations/state-board.md`).
- `[SESSION_START_PATH]` — where the session discipline doc lives (e.g. `docs/governance/session-start.md`).
- `[BOARD_OWNER_ROLE]` / `[BUILDER_ROLE]` — the session roles (e.g. "advisor session" / "build session"). Single-session project: one role carries both.
- `[DECISION_OWNER]` — the person or role who gates decisions for this scope; in a single-operator project, that's you.

This skeleton is PGK wiring only. It takes no position on your stack, repo structure, build commands, or coding conventions — add those yourself, outside the two parts, as your project needs them.

---

# [PROJECT_NAME] — session bootstrap

## PART 1 — STANDING RULES (stable; changes require an explicit [DECISION_OWNER] decision)

1. **Read the state board first.** Before any substantive work, read `[STATE_BOARD_PATH]` and verify its claims against actual project state (`git log`, the files themselves). Update it before the session ends. This is the one rule; everything else supports it.
2. **Session discipline.** Follow `[SESSION_START_PATH]` for how sessions start, end, update mid-session, and hand off.
3. **Role separation.** `[BOARD_OWNER_ROLE]` updates the board; `[BUILDER_ROLE]` commits work and does not modify the board directly; `[DECISION_OWNER]` may amend the board at any time. If one session carries both roles, it owns both duties.
4. **Authority.** The board is a synthesis surface, not a source of truth. The authoritative documents are named in the board's footer (the authority table); when the board conflicts with an authoritative source, the source wins.
5. **Honesty.** Never state as fact what you haven't verified — flag it `[unverified]`. A stale record is worse than no record.

## PART 2 — OPERATIONAL STATE (living)

Current project state lives on the state board: `[STATE_BOARD_PATH]`. This section intentionally does not duplicate it — duplicated state goes stale. If something here ever contradicts the board, the board is more current; fix this file.
