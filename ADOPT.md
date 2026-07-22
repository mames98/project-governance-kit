# Adopting PGK into an existing project

**For the operator:** point a Claude session at this file in your existing repo — "read `ADOPT.md` and follow it" — and answer its questions. The session scans your repo read-only, proposes governance drafts reconstructed from evidence, asks you to confirm or correct them, and only then writes files — handing you a ready-to-ratify commit at the end. It never commits without your go-ahead and never pushes.

**Empty repo?** Use `INSTANTIATE.md` instead — greenfield instantiation starts blank and interviews you; adoption reconstructs from history. This procedure is for repos that already have commits, branches, and decisions baked in.

---

## Procedure (addressed to the Claude session)

You are adopting the Project Governance Kit into a repository that already has history. Execute the four phases below **strictly in order** — no writing before C4, no inferring before the scan is done.

**Hard rules, binding throughout:**

- **The scan commits nothing.** C1–C2 are strictly read-only: no file writes inside this repo, no `git add`, no branch creation, nothing. Proposals live in your message to the operator (or a scratch file *outside* the repo).
- **Evidence citations are mandatory.** Every observation and every inference cites its evidence — a file path or a commit hash. A claim you cannot cite is either flagged `[unverified]` or not made.
- **No invented whys — ever.** Repo evidence shows *that* a decision happened (a framework appears in the manifest, a directory was abandoned, a migration landed). It never shows *why*. You leave every "why" blank for the operator. A reconstructed rationale that reads as contemporaneous provenance is the exact failure PGK exists to prevent.
- **This procedure never pushes.** Writing happens only in C4, after operator confirmation; committing happens only with the operator's explicit go-ahead; pushing is the operator's alone.

### C1 — SCAN (read-only)

Survey the repository and produce an **observation list** where every line carries its evidence (path or commit hash). Cover:

- **Repo structure** — top-level layout, where docs live, existing conventions.
- **README and any docs** — what documentation exists and what each piece claims to be.
- **`git log` shape** — age of the repo, contributors, activity bursts and gaps, merge patterns. (`git log --oneline`, `git shortlog -sn`, dates of first/last commits.)
- **Open branches and their staleness** — `git branch -a` with last-commit dates per branch.
- **PRs / issues** — if accessible (e.g. `gh pr list`, `gh issue list`); if not accessible, record that honestly rather than guessing.
- **Manifests / config** — package manifests, lockfiles, CI config, whatever identifies the stack.

Output the observation list to the operator. **Modify nothing.**

### C2 — INFER (proposals only)

From the C1 observations, produce **four proposals — each explicitly marked as a proposal**, each line still citing evidence:

1. **Authority-table candidates.** Which existing docs actually *function* as spec, plan, or work inventory (regardless of what they're named)? If none do, the honest answer is: "none; `git log` is the only authority." Never invent placeholder documents.
2. **A drafted state board.** Current focus inferred from recent commit clustering; work in flight from open branches/PRs; blocked/dormant candidates from stale branches. Every entry cites the commits or branches it's read from; anything uncertain is `[unverified]`.
3. **ADR candidates from decision fossils.** Framework/stack choices (manifest evidence), architecture pivots visible as large refactors, abandoned directories, migrations. **The that/not-why rule is absolute: evidence shows *that* a decision happened, never *why*. The "why" of every candidate is left blank for the operator to fill in C3.** Propose only decisions that appear to still actively bind future work — resist completeness.
4. **Placement.** Where the governance files fit this repo's existing conventions (from C1's structure observations), mapped against the kit's defaults in `INSTANTIATE.md` A1.

### C3 — CONFIRM

Present the four proposals and ask the operator to review — **batched questions, minimal round-trips**:

- Correct any inference that's wrong.
- For each ADR candidate they **accept**: supply the *why* (the Context/rationale only they know). No why supplied = the ADR is written with its why marked `[unverified — operator did not supply rationale]` or the candidate is dropped; you never fill it in yourself.
- Reject freely — a rejected candidate is simply dropped.
- Anything left unresolved is flagged `[unverified]` or dropped. **Never guessed.**

Do not proceed to C4 until the operator has responded.

### C4 — WRITE

Only now touch files.

1. **Place the kit files** per the confirmed placement proposal (templates per `INSTANTIATE.md` A1 conventions, adapted to this repo).
2. **Write ADR-000 — mandatory, the adoption line:**

   > PGK adopted at commit `<hash>` on `<date>`. Records before this line are reconstructed from evidence; records after are contemporaneous.

   `<hash>` is the repo's HEAD at adoption. ADR-000 is what keeps reconstructed history visibly reconstructed.
3. **Write the accepted ADR candidates**, each with:
   - `**Status:** Accepted (reconstructed from evidence, [date])`
   - a Context section containing **only cited evidence** (paths, hashes) plus the operator-supplied why from C3 — clearly attributed to the operator, not to the historical moment.
   - Backfill ONLY decisions that still actively bind future work; resist completeness. History that doesn't constrain the future belongs in `git log`, not the decision log.
4. **Initialize the board** from the confirmed board draft — honest sections, explicit empty-states, freshness marker (as `INSTANTIATE.md` A4).
5. **Wire the one rule** — `INSTANTIATE.md` A6, using the kit's `bootstrap/` skeletons; additive merge if `CLAUDE.md` exists.
6. **Verify** — `INSTANTIATE.md` A7 checks, plus: ADR-000 present and indexed; every reconstructed ADR carries the reconstructed-status marker and no uncited context.
7. **Hand off** — `INSTANTIATE.md` A8: plain-language summary distinguishing *scanned evidence* vs *operator-confirmed* content, then the exact named-path `git add`/commit for the operator to ratify. **Stop. Never push.**
