# Feedback from STE build-phase operation — the dispatch cycle, 2026-07-12

Second feedback batch from the STE instance (first: 2026-07-11 instantiation note). One day of real advisor/builder operation: two Goal+Loop dispatches authored, run, adversarially verified, merged. Everything below is observed, not speculative.

## 1. The automation tier's "dispatch templating" earn-condition is now MET at the source instance
`automation-tier` notes dispatch templating is earned "when an advisor/builder split is actively running and briefs are rewritten from scratch each time." STE now runs that split live (Cowork authors → operator relays → builder runs → advisor verifies). STE solves it with a skill (goal-loop-dispatch) rather than PGK templates — data point: the capability may belong as a *skill reference* in the tier, not a template.

## 2. Pattern worth codifying: the dispatch cycle with mandatory independent verification
Proven twice: a builder handoff is a CLAIM; the advisor verifies against git objects (log, file contents, its own test run) before confirming and issuing the next instruction. Both verifications added real value (caught nothing false, but produced evidence the operator could trust without reading code). Candidate for a PGK Core sentence in session-start's role-separation section.

## 3. Incident + rule that generalizes: shared-working-tree contamination
The advisor session committed governance content onto the builder's branch because the shared checkout was still on that branch (STE incident `e6fd155`). Fix adopted in the instance: while a builder holds a coordination flag, advisor verification is read-only against git objects, or in a separate clone; no state-changing commands in the shared tree. Generic enough for PGK Core (session-start accuracy/role discipline).

## 4. Board mechanics that worked under real load
Coordination flags transitioning session-hold → standing freeze/pin (two different end-states, both needed); "Recent shipped" absorbing build merges with one-line evidence summaries; trigger-based mid-session updates (5 board commits in one day, none onerous). No template change needed — recording that the load test passed.

## 5. Minor: builder sessions correctly refused board edits twice and said so in their handoffs
The role-separation rule held without enforcement tooling — the builder cited the rule and delegated the update both times. Evidence the written rule suffices at this scale; automation-tier machinery remains unearned.

## Provenance
STE commits: `26c8482` (Dispatch #1 merge), `7ffbd6c` (Dispatch #2 merge), `e6fd155` (incident), `febe69e` (hygiene rule), session-start.md dispatch-cycle agreement. Operator: mames98.
