# Session Discipline — [PROJECT_NAME]

How every working session on this project starts, ends, and hands off. This document carries the operating rules for the state board (`[STATE_BOARD_PATH]`); the board itself stays a lean living document.

**The one rule:** every working session reads the state board before doing anything else, and updates it before ending. Everything below supports that one rule.

---

## 1. Session start

Before beginning any substantive work:

1. **Read the state board** (`[STATE_BOARD_PATH]`).
2. **Verify its current-state claims** against actual project state: read recent `git log`, check the project's operational-state doc for obvious staleness, confirm which sessions are active.
3. **Update stale sections** before proceeding. A session that starts from a stale state board is operating on false context. The correction cost is minutes; the misdirection cost is hours.

If a session begins substantive work without acknowledging the state board, the operator should redirect before proceeding.

## 2. Session end

Before closing:

1. **Update "Work in flight"** — capture any uncommitted work, which session holds it, and commit status.
2. **Update "Open operator decisions"** — add any newly-surfaced decisions; close any that resolved during the session.
3. **Update "Recent shipped"** — add any commits that landed during the session.
4. **Update "Don't touch right now"** — add new coordination flags for work in progress; remove stale flags for work that has closed.
5. **Update "Current focus"** — if the primary work thread changed during the session, reflect that.

Session-end discipline applies even if the session ends abruptly. **A partial update is better than no update.**

## 3. Trigger-based updates (not just at session boundaries)

Update the board mid-session when:

- An operator decision is made or explicitly deferred
- A commit lands during an active session (update "Recent shipped")
- A work thread closes or a new one opens
- A coordination flag changes (a session starts or completes work in a shared file)
- A blocked item unblocks

Sections may be updated individually; full-section rewrites are not required for small changes. **The goal is accuracy, not document beauty.**

## 4. Accuracy discipline

**The board's value is entirely dependent on accuracy. A stale state board is worse than no state board** — it creates false confidence and misdirects fresh sessions.

- Update the board whenever work state changes meaningfully, not just at convenient moments. "Meaningfully" means: a commit lands, a session starts or completes a work thread, an operator decision is made or deferred, a blocking condition changes.
- If a claim can't be verified during a session (e.g. "is session X still active?"), flag it `[unverified as of DATE]` rather than leaving it stale-as-certain.
- **Freshness marker:** the board begins with `**Last updated:** [date] ([brief context])`. Update this line whenever the board is modified. A board whose last-updated date is more than 3 days old is stale pending verification.
- **The 24-hour cross-check:** the highest-risk staleness pattern is work closing without the board being updated — an "in flight" entry can persist hours or days after the work closes. When a board entry is more than 24 hours old, cross-check the claimed in-flight status against `git log` before acting on it. A closed work thread shown as "in progress" is worse than no entry at all.

The failure mode is recoverable: if the board gets stale, it gets updated. The protection is operator vigilance plus session-boundary discipline. The failure mode to prevent is leaving a stale board that looks authoritative to a fresh session.

## 5. Role separation

Who touches the board:

- **[BOARD_OWNER_ROLE]** (the advisor/synthesis session) updates the board — at session start, session end, and on the triggers above. Board updates are synthesis operations, not build operations.
- **[BUILDER_ROLE]** (the implementation session) commits code — which the board reflects via "Recent shipped" — but does **not** modify the board directly.
- **The operator** may amend the board ad-hoc at any time for clarity or to surface new context. Operator amendments are valid immediately and are incorporated at the next read.

If the project runs with a single session role, that session carries the [BOARD_OWNER_ROLE] duties; the separation matters as soon as more than one session works the project concurrently.

## 6. Handing off

Recommend a fresh session when:

- The current session has accumulated significant tactical context that's no longer load-bearing
- A natural completion point has been reached (deliverable shipped, milestone met, audit completed)
- The next question requires evaluating the current session's own prior work (loss of objectivity — don't ask the session that built X whether X was the right approach)

A handoff captures: core context, key decisions made, what's pending, and anything the next session must not redo. The state board carries the durable state; the handoff carries the conversational context the board doesn't.
