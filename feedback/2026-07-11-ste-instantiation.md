# Feedback from first real instantiation — Satoshi Trading Engine (STE), 2026-07-11

PGK's first instantiation into a real project (STE, governed pre-build phase). Total time ~30 min including four opening ADRs — within INSTANTIATE.md's estimate. Everything below is friction actually hit, not speculation. Ordered by usefulness of fixing.

## 1. Pre-existing decision records: the kit is silent
STE arrived with four ratified decision documents predating the log (edge decision, OSS deps, borrow decision, architecture doc). INSTANTIATE.md doesn't say whether to renumber/absorb them into the ADR sequence or leave them standing. Chosen resolution (worked well, recommend codifying): leave pre-existing records authoritative as-is, state that explicitly in the decisions README, start ADR numbering fresh from the instantiation date. Suggest a sentence in INSTANTIATE.md step 4 naming this pattern — most non-greenfield projects will hit it.

## 2. decision-log.md's triple role is underspecified
The template is simultaneously (a) conventions → README, (b) the Index table, (c) the ADR skeleton. Step 1 says use "the conventions section as docs/decisions/README.md" but doesn't say where the Index lives or whether the skeleton travels into the instance. Chosen resolution: README = conventions + Index; skeleton NOT copied (it lives in the kit; instances copy from there). Fine, but each instantiation will re-decide this. One clarifying sentence fixes it.

## 3. Status vocabulary lacks the operator-directed case
Status values assume propose→sign-off inside the project. STE's opening ADRs arrived pre-decided via operator handoff; a session only recorded them. Improvised: `Accepted — operator handoff [date], recorded by [session] [date]`. Recommend blessing this form in the conventions.

## 4. [DECISIONS_DIR] missing from the placeholder list
INSTANTIATE.md step 2 enumerates placeholders but omits `[DECISIONS_DIR]` (present in decision-log.md). Trivial; add it.

## 5. What worked without friction (no change needed)
Explicit empty-state statements; the authority-table instruction ("don't invent placeholder documents"); the deferred Automation Tier framing — recording it as available-but-not-activated in the instance's board took one bullet and felt natural, exactly as designed; the two-part CLAUDE.md bootstrap pattern (PART 1 standing rule) slotted cleanly into an existing ratified CLAUDE.md without disturbing it.

## Provenance
Instance: satoshi-trading-engine commits `527dd43` (PGK Core instantiation) and `0535548` (ADRs 001–004), local-only pending remote creation.
