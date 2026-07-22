# Operational State Board — [PROJECT_NAME]

**FIRST READ for any session.** Before any substantive work, verify the claims in this document against actual project state via `git log` and the authoritative source documents. Read fully and orient before proposing or executing work.

**Last updated:** [DATE] ([brief context — what changed])

**The single rule:** this board is a **synthesis surface, not a source of truth**. Every claim here either references an authoritative source or is self-evidently transient operational state. If you need to act on a state-board claim, verify the authoritative source first. The board is orientation, not authorization.

---

## Current focus

[What is the active focal point right now — the single most important open work thread. If multiple threads are simultaneously active, name the primary one and list secondary ones. One paragraph.]

## Active sessions

<!-- Which sessions (AI or human) are doing what. Explicit identification prevents cross-session conflicts — two sessions committing to the same file simultaneously. Entries may carry a bracketed plain-language annotation explaining the item to a reader without full context. -->

| Session | Purpose | Current activity | Last known state |
|---|---|---|---|
| [SESSION_NAME] | [what it exists to do] | [what it's doing now] | [last verified state, with date] |

---

## Work in flight

<!-- Uncommitted work in any session. If nothing is in flight, say so explicitly: "Nothing uncommitted as of [date]." -->

| Work | Session | Files | Status |
|---|---|---|---|
| [what the work is] | [which session holds it] | [file(s) being modified] | [draft / ready / blocked + any cross-session dependency] |

---

## Open decisions

<!-- Decisions that require [DECISION_OWNER] input — [DECISION_OWNER] is the person or role who gates decisions for this scope; in a single-operator project, that's you. Each open decision names its owner (multi-operator projects route decisions to different owners; single-operator projects have one value everywhere). If no decisions are open, say so explicitly. Strike through and mark CLOSED with date when resolved — closed entries may be retained briefly for continuity, then pruned. -->

| Decision | Owner | What's blocking | Surfaced |
|---|---|---|---|
| [what decision is needed] | [DECISION_OWNER] | [what's blocking on it — which stage, spec, or work thread] | [date it was surfaced] |

---

## Blocked / pending external

<!-- Items waiting on external events, time-based triggers, or third-party inputs. -->

| Item | Waiting on | Expected timing |
|---|---|---|
| [what is waiting] | [what trigger resolves it] | [expected timing where known] |

---

## Recent shipped (last 7 days — governance-relevant commits only; full history: `git log --oneline`)

<!-- One line each. Cosmetic / infrastructure commits may be grouped or omitted. -->

| Commit | What |
|---|---|
| `[COMMIT_HASH]` | [what it did] |

---

## Parallel-work candidates

<!-- Work items that could be picked up if the current focus has waiting room. -->

| Item | Why it's a parallel candidate | Preconditions |
|---|---|---|
| [what the work is] | [low dependency, [DECISION_OWNER]-only, isolated scope, etc.] | [any readiness preconditions, or "None"] |

**Not currently parallel-eligible (gated):**
- [item — what gates it]

---

## Don't touch right now

<!-- Explicit coordination flags: named files, directories, or work threads actively held by another session or inside a scheduled automated process's window. This section prevents collision. If no coordination flags are active, say so explicitly. -->

| Flag | Reason |
|---|---|
| [file / directory / work thread] | [who holds it, why, and what releases the flag] |

---

*State board maintained by [BOARD_OWNER_ROLE]. Authoritative sources: [AUTHORITATIVE_DOC_1] ([what it governs]), [AUTHORITATIVE_DOC_2] ([what it governs]), `git log` (commit history). When the board conflicts with an authoritative source, the authoritative source wins. When in doubt, read the authoritative source.*
