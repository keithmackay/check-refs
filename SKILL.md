---
name: check-refs
description: Use when finalizing an article or document with numbered references or footnotes - verifies citation ordering, primary-source accuracy, and catches orphaned, duplicate, broken, or missing references.
---

# Check References

Runs a reference integrity pass on any article or document with numbered inline citations. This ensures credibility and prevents misattribution before publishing.

**This pass edits the target document directly** (renumbering, adding, removing, and merging references) unless `--dry-run` is passed.

## Arguments

- `/check-refs` (no args) - Run the full reference integrity pass on the target document and apply the fixes
- `/check-refs --dry-run` - Run the full pass and produce the report, but do not edit the document. Every action that would normally be applied (renumber, add, remove, merge, swap) is listed under a `WOULD DO:` heading instead.
- `/check-refs --help` - Do not run any other part of this skill. Read and display the contents of `help.md` (in this skill's folder) verbatim, then stop.

## Target Document

Before running the pass: if the user's invocation or the current conversation already names a specific file, confirm it before proceeding: "Running check-refs against `<path>` - confirm?" If no target was specified, ask the user which document to check. Never guess or scan a whole project for candidate documents.

## Confirmation Gate

Before running any step of the pass, state plainly that this pass edits the document directly (renumbering, adding, removing, and merging references) unless `--dry-run` was specified, and confirm with the user before proceeding if this is the first invocation in the current session against this document.

## What Requires a Reference

- Any specific number, statistic, or data point (e.g., "$300 billion", "40% increase")
- Any direct quotation of a person, article, report, or other source
- Any claim attributed to a specific organization or study
- Paraphrased findings from a specific source (even if not directly quoted)

## The Reference Integrity Pass

Run these steps in order. Renumbering is deliberately last, because every step before it can change which references exist.

1. **Detect the existing convention.** Before applying any convention, detect what the document already uses: numbered `[N]` inline markers vs. footnote-style `[^N]` vs. superscript vs. parenthetical `(N)`; and the heading used for the reference list (`References`, `Sources`, `Notes`, `Bibliography`, `Works Cited`, or none). Preserve the document's existing convention throughout the pass. Only apply this skill's own default convention (numbered `[N]` + `## References`) when creating a references section from scratch in a document that has none.

   If the document has citable claims (per "What Requires a Reference") but no references section at all, ask the user whether to create one before adding it — do not silently invent a new section in a document structure the user may not want changed.

2. **Coverage check.** Scan the document for any unattributed statistics, direct quotes, or specific claims that lack a reference. Add references where needed.

3. **Number-to-link validation.** For each numbered reference in the body, verify that the corresponding entry in the reference list points to the correct source, and that it matches the claim being cited.

   - **URL-backed references:** verify the link resolves and matches the claim. Use WebFetch (or the session's equivalent web-fetch tool) to do this. On a document with more than ~15 references, triage: always verify references backing a specific numeric statistic or direct quote; sample-check the rest. If network access is unavailable or blocked, state this plainly in the report's UNRESOLVED section rather than silently skipping verification.
   - **Non-URL references:** verify only that the citation names the source the claim attributes it to. Its completeness is checked separately at step 8 — never demand a URL here.

4. **Orphaned reference check.** Verify that every entry in the reference list is actually cited somewhere in the body. Remove orphaned references that are listed but never cited.

5. **Duplicate check.** Ensure no source appears more than once in the reference list under different numbers. If the same source is cited multiple times in the body, all citations should use the same reference number.

6. **Primary source check.** Every reference should cite the primary source, not a secondary article that reports on it. If the body attributes a finding to Organization X but the reference links to a blog post by Organization Y that merely summarizes X's work, replace it with the original source from Organization X. Search for the primary report, paper, or announcement. Secondary sources are acceptable only when the primary source is unavailable (paywalled, taken down, etc.), in which case note "Referenced in [secondary source]" in the citation.

   **Never invent, guess, or fabricate a primary source citation.** If a primary source cannot be located after a genuine search, leave the existing secondary source in place with the "Referenced in [secondary source]" note, or flag it under UNRESOLVED in the report — do not supply an unverified or plausible-sounding replacement citation.

   This step applies only to sources that have a discoverable primary version. Skip it for interviews the author personally conducted, unpublished internal data, and out-of-print works with no accessible primary alternative — note these as "primary by nature" rather than flagging them.

7. **Broken-link check.** For every URL-backed reference, confirm the URL resolves (not a 404, not a redirect to an unrelated homepage). If a reference is broken, attempt an archive.org fallback before flagging; if no working version exists, mark it as `BROKEN` in the report rather than silently leaving it in place.

8. **Non-URL source completeness check.** For every reference without a URL, confirm it names enough to be independently located: author, title, publication/venue, and date. Flag any that fall short; do not demand a URL.

9. **Sequential renumbering.** References in the reference list MUST appear in the same order as they are first cited in the document body. Number them sequentially: the first reference cited gets [1], the second gets [2], etc. Renumber once, here, after all additions, removals, merges, and swaps are settled.

## Reference Format

Use the document's detected inline convention (see step 1). This skill's default for a document with none is numbered inline references in the body: `[1]`, `[2]`, etc.

Place the reference list immediately before any closing or appendix material the document already has (acknowledgments, author bio, etc.); if the document has no such trailing material, place it as the final section.

A reference may take either of two valid forms:

```
## References

1. [Description or title](URL)
2. Description, author, publication/venue, and date — no stable URL
```

Examples of the URL-less form: `Smith, J. *Systems of Scale*, O'Reilly, 2023, print` or `Interview with Jane Doe, conducted by the author, 2026-01-15`.

A reference is valid without a URL if it names a verifiable source (author, title, venue, date) even without a link.

## Output Format

In applied mode (default), report what was changed. In `--dry-run` mode, change nothing and list every action under a `WOULD DO:` heading instead.

```
CHECK-REFS REPORT: [document title]
MODE: [applied (document edited) | dry-run (no changes made)]

CONVENTION DETECTED: [e.g. [^N] footnotes + "Sources" heading - preserved]

WOULD DO:  (dry-run mode only - replaces the applied-mode change lists below)
- Add reference for "[claim/stat]" at [location]
- Remove orphaned reference [N]. [description]
- Merge [old N] and [old M] -> [N]
- Swap [N] from [secondary source] to [primary source]
- Replace broken [N] with archive.org copy
- Renumber [old N] -> [new N]

MISSING REFERENCES ADDED: [N]
- "[claim/stat]" at [location] -> added as [N]

ORPHANED REFERENCES REMOVED: [N]
- [N]. [description] - never cited in body

DUPLICATES MERGED: [N]
- [old N] and [old M] both pointed to [source] -> merged to [N]

PRIMARY SOURCE SWAPS: [N]
- [N]. was [secondary source] -> now [primary source]

BROKEN REFERENCES: [N]
- [N]. [URL] - [404 / redirects to unrelated page]; [archive.org copy substituted | no working version found]

NON-URL SOURCES: [N] - exempted from link validation, checked for completeness instead
- [N]. [citation] - [complete | missing: venue, date]
- [N]. [citation] - primary by nature (author-conducted interview / unpublished internal data / out-of-print)

RENUMBERED: [N]
- [old N] [description] -> now [new N]

UNRESOLVED (needs human input):
- [item]: [why it couldn't be auto-fixed - e.g. primary source not found; left as secondary rather than replaced with an unverified substitute]

UNRESOLVED (no network access):
- [N]. [URL] - could not be verified; network access unavailable or blocked

STATUS: [Pass / N issues fixed / N issues need review]
```

Reporting-only findings (CONVENTION DETECTED, BROKEN REFERENCES, NON-URL SOURCES, UNRESOLVED) appear in both modes. Never list a primary-source swap to a source that was not actually found and verified — if none was located, say so under UNRESOLVED.
