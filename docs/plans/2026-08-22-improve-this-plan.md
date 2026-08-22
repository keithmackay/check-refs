# Implementation Plan: check-refs improve-this findings

**Source:** `docs/reviews/2026-08-21-improve-this.md`
**Status of already-fixed findings:** #1 (no target-document resolution) and #3 (help.md orphaned) were fixed in the 2026-08-22 `--help`/target-resolution pass. Excluded below.

Phased so the behavioral question (destructive-by-default) is settled first, since it changes the shape of the Output Format that later phases also edit; citation-convention flexibility and non-URL sources come next since they're the largest generalization gaps; step reordering and mechanical cleanup come last.

---

## Phase 1: Resolve destructive-by-default behavior (Finding #4, High/Medium)

### Task 1.1 — Add a `--dry-run` argument, and decide the default

Add to Arguments: `/check-refs --dry-run` - Run the full pass and produce the report, but do not edit the document; every action that would normally be applied (renumber, add, remove, merge, swap) is listed under a `WOULD DO:` heading instead. Default behavior: keep `/check-refs` (no args) as edit-and-report, since that matches the skill's original design intent and its name change would otherwise be needed — but require an explicit up-front confirmation step (Task 1.2) so a first-time user isn't surprised.

### Task 1.2 — Add a confirmation gate to the Workflow

Before Step 1 (Sequential ordering) of the Reference Integrity Pass, insert: "State plainly that this pass edits the document directly (renumbering, adding, removing, and merging references) unless `--dry-run` was specified, and confirm with the user before proceeding if this is the first invocation in the current session against this document."

### Task 1.3 — Update `README.md` and `help.md`

Add `--dry-run` to both, and adjust the framing from "verifies... catches" to acknowledge the pass edits by default: e.g. "runs a reference integrity pass and applies fixes directly unless `--dry-run` is passed."

**Verification:** Confirm the Output Format section (edited in Phase 5) has a clear distinct heading for dry-run vs. applied mode, and that the Workflow explicitly calls out the confirmation gate before any edit step.

---

## Phase 2: Support non-URL sources (Finding #2, High/High)

### Task 2.1 — Extend the Reference Format to allow URL-less citations

Add an alternative citation form alongside `[Description or title](URL)`:
```
1. [Description, author, publication/venue, and date — no stable URL] (e.g. "Smith, J. *Systems of Scale*, O'Reilly, 2023, print" or "Interview with Jane Doe, conducted by the author, 2026-01-15")
```
State explicitly: a reference is valid without a URL if it names a verifiable source (author, title, venue, date) even without a link.

### Task 2.2 — Update steps 2 and 6 to branch on source type

Step 2 (number-to-link validation): "For URL-backed references, verify the link resolves and matches the claim (see Phase 4 for fetch behavior). For non-URL references, verify the citation is complete enough to be independently locatable (author, title, venue, date) — this is a completeness check, not a link check."

Step 6 (primary source check): "Applies only to sources that have a discoverable primary version. Skip this step for interviews the author personally conducted, unpublished internal data, and out-of-print works with no accessible primary alternative — note these as 'primary by nature' rather than flagging them."

**Verification:** Run the pass against a synthetic document with one book citation, one personal-interview citation, and one URL-backed citation, and confirm each is handled by its correct branch rather than the pass demanding a URL for all three.

---

## Phase 3: Support multiple citation conventions (Finding #5, Medium/High)

Depends on Phase 2 (the format now has two valid shapes to detect between).

### Task 3.1 — Add a "detect existing convention" step at the start of the pass

Insert before Step 1: "Before applying any convention, detect what the document already uses: numbered `[N]` inline markers vs. footnote-style `[^N]` vs. superscript vs. parenthetical `(N)`; and the heading used for the reference list (`References`, `Sources`, `Notes`, `Bibliography`, `Works Cited`, or none). Preserve the document's existing convention throughout the pass. Only apply this skill's own default convention (numbered `[N]` + `## References`) when creating a references section from scratch in a document that has none."

### Task 3.2 — Add a rule for "no references section at all"

Add: "If the document has citable claims (per 'What Requires a Reference') but no references section, ask the user whether to create one before adding it — do not silently invent a new section in a document structure the user may not want changed."

### Task 3.3 — Remove the LinkedIn-specific placement assumption

Edit "before any closing/engagement material" in the Reference Format section to: "immediately before any closing or appendix material the document already has (acknowledgments, author bio, etc.); if the document has no such trailing material, place it as the final section."

**Verification:** Confirm the pass, run against a document using `[^N]` footnotes and a "Sources" heading, preserves both rather than converting to `[N]` + `## References`.

---

## Phase 4: Fetch behavior and dead-link detection (Findings #6, #7)

### Task 4.1 — Name the fetch tool and triage rule (Finding #6, Medium/High)

Add to the Reference Integrity Pass, at step 2: "Use WebFetch (or the session's equivalent web-fetch tool) to verify a reference's link/description matches its claim. On a document with more than ~15 references, triage: always verify references backing a specific numeric statistic or direct quote; sample-check the rest. If network access is unavailable or blocked, state this plainly in the report's UNRESOLVED section rather than silently skipping verification."

### Task 4.2 — Add a genuine dead-link check (Finding #7, Medium/Medium)

Add a new step 7 (after Duplicate check, before Primary source check — see Phase 6 for final ordering): "**Broken-link check.** For every URL-backed reference, confirm the URL resolves (not a 404, not a redirect to an unrelated homepage). If a reference is broken, attempt an archive.org fallback before flagging; if no working version exists, mark it as `BROKEN` in the report rather than silently leaving it in place."

### Task 4.3 — Disambiguate "dead reference" terminology (Finding #11, Low/High)

Rename the existing step 4 from "Dead reference check" to "Orphaned reference check" throughout `SKILL.md`, and reserve "dead" exclusively for the new broken-link check from Task 4.2.

**Verification:** Confirm `SKILL.md` no longer uses "dead" for two different meanings, and that the new broken-link step has its own report heading distinct from "orphaned."

---

## Phase 5: Rework the Output Format (depends on Phases 1-4)

### Task 5.1 — Rebuild the report template

Incorporate: the dry-run/applied-mode distinction (Phase 1), a `NON-URL SOURCES` line (Phase 2) confirming they were exempted from link validation, a note on which convention was detected/preserved (Phase 3), an `UNRESOLVED (no network access)` case (Phase 4.1), and a `BROKEN REFERENCES` heading distinct from `DEAD REFERENCES REMOVED`/now-renamed `ORPHANED REFERENCES REMOVED` (Phase 4.2-4.3).

**Verification:** Walk through the new template against each of the scenarios exercised in Phases 1-4's own verification steps and confirm every new behavior has a corresponding report line.

---

## Phase 6: Reorder steps to remove rework (Finding #8, Medium/Medium)

Do this last since Phases 2-4 changed what the steps are and how many there are.

### Task 6.1 — Resequence the Reference Integrity Pass

New order: detect existing convention (Phase 3) → coverage check (add missing references) → orphaned-reference check → duplicate check → primary-source check → broken-link check → non-URL source completeness check → **sequential renumbering, done once, last**. Renumbering must be the final step since every prior step can change which references exist.

### Task 6.2 — Recommend `plsfix` for a clarity pass

Per the review's note, after the content changes in Phases 1-6 are in place, run the `/plsfix` skill (or a manual clarity pass) over the resequenced `SKILL.md` to tighten phrasing and remove any leftover redundancy from the edits.

**Verification:** Re-read the full pass top-to-bottom and confirm no later step invalidates an earlier step's output (the specific defect the review flagged).

---

## Phase 7: De-duplication and README polish (Findings #9, #10, #12)

### Task 7.1 — Single source of truth for arguments (Finding #9, Low/High)

Trim `README.md`'s argument list to a one-line pointer to `help.md`; keep the full list only in `SKILL.md` and `help.md`.

### Task 7.2 — Move Output Format to `references/report-format.md` (Finding #10, Low/Medium)

Defer this until after Phase 5 stabilizes the template shape. Once stable, extract it to a reference file and replace the inline block with an unconditional read instruction at the start of the reporting step.

### Task 7.3 — README verification and worked example (Finding #12, Low/Medium)

Add to `README.md`: an install-verification line ("run `/check-refs --help` to confirm the install"), a short before/after worked example showing a renumbered/fixed reference list, and a fix for the `git clone` + `ln -s "$(pwd)/...")` ambiguity (state explicitly that the commands run from the parent directory, before `cd`-ing into the clone).

**Verification:** Confirm `README.md` now has exactly one place stating the argument list, a working example a new user can compare their own output against, and unambiguous install instructions.

---

## Next Steps (beyond this plan)

- Consider a `--convention <style>` override argument for users who want to force a specific citation style regardless of what's detected, once Phase 3 ships and real usage shows the auto-detection needs an escape hatch. [Claude's idea]
- Consider whether the primary-source check (step 6, pre-existing) should itself move to a `references/` file with a fuller decision tree, if Phase 2's expanded source-type handling makes it grow significantly. [Claude's idea]
