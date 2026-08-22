---
name: check-refs
description: Use when finalizing an article or document with numbered references or footnotes - verifies citation ordering, primary-source accuracy, and catches dead, duplicate, or missing references.
---

# Check References

Runs a reference integrity pass on any article or document with numbered inline citations. This ensures credibility and prevents misattribution before publishing.

## Arguments

- `/check-refs` (no args) - Run the full reference integrity pass on the target document
- `/check-refs --help` - Do not run any other part of this skill. Read and display the contents of `help.md` (in this skill's folder) verbatim, then stop.

## Target Document

Before running the pass: if the user's invocation or the current conversation already names a specific file, confirm it before proceeding: "Running check-refs against `<path>` - confirm?" If no target was specified, ask the user which document to check. Never guess or scan a whole project for candidate documents.

## What Requires a Reference

- Any specific number, statistic, or data point (e.g., "$300 billion", "40% increase")
- Any direct quotation of a person, article, report, or other source
- Any claim attributed to a specific organization or study
- Paraphrased findings from a specific source (even if not directly quoted)

## The Reference Integrity Pass

1. **Sequential ordering.** References in the reference list MUST appear in the same order as they are first cited in the document body. Number them sequentially: the first reference cited gets [1], the second gets [2], etc. If references are out of order, renumber them.
2. **Number-to-link validation.** For each numbered reference in the body (e.g., [1], [2]), verify that the corresponding entry in the reference list points to the correct source. Read the reference link/description and confirm it matches the claim being cited.
3. **Coverage check.** Scan the document for any unattributed statistics, direct quotes, or specific claims that lack a reference. Add references where needed.
4. **Dead reference check.** Verify that every entry in the reference list is actually cited somewhere in the body. Remove orphaned references that are listed but never cited.
5. **Duplicate check.** Ensure no source appears more than once in the reference list under different numbers. If the same source is cited multiple times in the body, all citations should use the same reference number.
6. **Primary source check.** Every reference should cite the primary source, not a secondary article that reports on it. If the body attributes a finding to Organization X but the reference links to a blog post by Organization Y that merely summarizes X's work, replace it with the original source from Organization X. Search for the primary report, paper, or announcement. Secondary sources are acceptable only when the primary source is unavailable (paywalled, taken down, etc.), in which case note "Referenced in [secondary source]" in the citation.

## Reference Format

Use numbered inline references in the document body: `[1]`, `[2]`, etc.

At the end of the document (before any closing/engagement material), include a references section:

```
## References

1. [Description or title](URL)
2. [Description or title](URL)
```

## Output Format

```
CHECK-REFS REPORT: [document title]

REORDERED: [N references renumbered, if any]
- [1] was [old link/desc] -> now [new link/desc], moved from position [X] to [Y]

MISSING REFERENCES ADDED: [N]
- "[claim/stat]" at [location] -> added as [N]

DEAD REFERENCES REMOVED: [N]
- [N]. [description] - never cited in body

DUPLICATES MERGED: [N]
- [old N] and [old M] both pointed to [source] -> merged to [N]

PRIMARY SOURCE SWAPS: [N]
- [N]. was [secondary source] -> now [primary source]

UNRESOLVED (needs human input):
- [item]: [why it couldn't be auto-fixed - e.g. primary source not found]

STATUS: [Pass / N issues fixed / N issues need review]
```
