# improve-this review — check-refs

**Date:** 2026-08-21
**Scope:** Full project (`/Users/Keith.MacKay/Projects/check-refs`)
**Project type:** Prompt-only Claude Code skill (no executable code). Files: `SKILL.md` (68 lines, always loaded), `README.md`, `help.md`, `LICENSE`, `.gitignore`.
**Context:** Recently extracted from the personal `writeli` LinkedIn-writing skill into a standalone, general-purpose skill. Evaluation weights generalization to arbitrary writers and document types.

**Categories evaluated:** Clarity & Simplification · Completeness · Accuracy & Consistency · Navigability & Structure · Redundancy · Edge Case Coverage · Token Efficiency & Progressive Disclosure

---

## Part 1 — Priority List

```
#1  [Impact: High   | Confidence: High]   Completeness — No target-document resolution; skill never says what it operates on
#2  [Impact: High   | Confidence: High]   Edge Case Coverage — Non-URL sources (books, interviews, paywalled PDFs) unsupported by the mandated format
#3  [Impact: High   | Confidence: High]   Accuracy & Consistency — `--help` is specified but the mechanism (help.md) is never referenced from SKILL.md
#4  [Impact: High   | Confidence: Medium] Edge Case Coverage — Skill silently mutates the user's document; no dry-run / report-only mode or confirmation gate
#5  [Impact: Medium | Confidence: High]   Edge Case Coverage — Assumes one rigid citation convention (`[1]` + `## References` + Markdown links)
#6  [Impact: Medium | Confidence: High]   Completeness — Step 2 ("number-to-link validation") and step 6 (primary source) require fetching URLs; no tooling or offline fallback stated
#7  [Impact: Medium | Confidence: Medium] Edge Case Coverage — No handling for dead/404 URLs, despite the description promising to catch "dead" references
#8  [Impact: Medium | Confidence: Medium] Clarity & Simplification — Step ordering causes rework: renumbering first, then adding/removing/merging
#9  [Impact: Low    | Confidence: High]   Redundancy — Argument list triplicated across SKILL.md, help.md, README.md
#10 [Impact: Low    | Confidence: Medium] Token Efficiency — Output Format block (~25 lines, 35% of SKILL.md) is always loaded but only needed at report time
#11 [Impact: Low    | Confidence: Medium] Accuracy & Consistency — "dead reference" is used for two different meanings
#12 [Impact: Low    | Confidence: Medium] Navigability & Structure — README omits install verification and a worked example
```

---

## Part 2 — Categorized Breakdown

### Completeness

**#1 — No target-document resolution (Impact: High, Confidence: High)**
`SKILL.md:12` says "Run the full reference integrity pass on the target document," but nothing defines *the target document*. Inside `writeli` this was implicit — the article just drafted was in context. Standalone, a user may invoke `/check-refs` with no file open, several candidate files, or a path/glob argument. Without a resolution rule the skill will either guess wrong or stall. Needs an explicit precedence: explicit path argument → file under discussion in the conversation → most recently edited candidate → otherwise ask. Also should accept `/check-refs <path>`, which the Arguments section currently does not list at all.

**#6 — Verification steps assume fetch capability with no fallback (Impact: Medium, Confidence: High)**
Step 2 requires confirming each reference "points to the correct source," and step 6 requires searching for and swapping in primary sources. Both are network operations, but SKILL.md never names a tool (WebFetch/WebSearch), never says whether to fetch every reference or only suspicious ones, and gives no behavior for when the network is unavailable or a corporate proxy blocks it. On a 40-reference document, "verify each" without guidance is either extremely slow or quietly skipped. Should state the tool, a triage rule for which references warrant a fetch, and a documented degradation path (verify what is checkable, list the rest under UNRESOLVED).

### Edge Case Coverage

**#2 — Non-URL sources are unrepresentable (Impact: High, Confidence: High)**
The mandated format (`SKILL.md:40-41`) is `[Description or title](URL)` — every reference must have a URL. Real long-form writing cites books, print journals, interviews the author conducted, internal data, conference talks, and paywalled reports with no stable public link. Under the current rules such a reference has no valid form, and step 2's "read the reference link" and step 6's "search for the primary report" have no defined behavior. This is the single biggest generalization gap from the LinkedIn-article origin, where a URL per citation was a safe assumption. Needs a URL-less citation form and an explicit exemption from link validation for those entries.

**#4 — Destructive by default, with no report-only mode (Impact: High, Confidence: Medium)**
Steps 1, 3, 4, 5 and 6 are all imperatives that edit the user's document: "renumber them," "Add references where needed," "Remove orphaned references," "replace it." The Output Format is a past-tense record of edits already made ("DEAD REFERENCES REMOVED"). A general-purpose audit tool that rewrites someone's finished manuscript on first invocation — including *deleting* reference entries and *rewriting* citations — is a poor default for a new user who expected a check. Note the name and description both frame it as verification ("verifies," "catches"), which sets a read-only expectation the behavior does not meet. A `--dry-run`/report-only mode (arguably the default, with `--fix` to apply) would resolve the mismatch. Confidence is Medium only because the auto-fix behavior may be a deliberate design choice; the expectation mismatch is real either way.

**#5 — One rigid citation convention assumed (Impact: Medium, Confidence: High)**
The skill hardcodes `[1]` inline markers, a `## References` H2 heading, an ordered Markdown list, and Markdown link syntax. Documents in the wild use `[^1]` Markdown footnotes, superscript, parenthetical `(1)`, "Sources"/"Notes"/"Bibliography"/"Works Cited" headings, and non-Markdown formats entirely. `SKILL.md:35` even assumes the references sit "before any closing/engagement material," a LinkedIn-post-specific artifact that will confuse writers of other document types. The skill should detect and preserve the document's existing convention, and apply its own only when creating a section from scratch. Related: nothing says what to do when there is no references section at all — create one, or report and stop?

**#7 — No handling of URLs that no longer resolve (Impact: Medium, Confidence: Medium)**
The skill description advertises catching "dead" references, and users will read that as broken links. The pass has no step for it: step 4's "dead reference check" is about orphaned entries, not unreachable URLs. Given the skill runs at publish time, a 404 or redirect-to-homepage in a citation is exactly the failure a final check should catch. Also unaddressed: archive.org fallbacks, and URLs that resolve but whose content has changed.

### Clarity & Simplification

**#8 — Step order forces rework (Impact: Medium, Confidence: Medium)**
Step 1 renumbers everything into first-citation order, then step 3 adds new references, step 4 removes orphans, and step 5 merges duplicates — each invalidating the numbering step 1 just established. The reader is left to infer they should renumber again at the end. Reordering so that content changes (coverage, dead, duplicate, primary-source) precede a single final renumbering pass would remove the ambiguity and the wasted work. The current sequence likely reflects the order the checks were originally written, not the order they should execute.

**Note:** a clarity/token pass on the remaining always-loaded SKILL.md content is worth doing alongside any restructuring (the `plsfix` skill is suited to this).

### Accuracy & Consistency

**#3 — `--help` specified but not wired up (Impact: High, Confidence: High)**
`SKILL.md:13` promises `--help` "Prints this skill's summary and argument list." A `help.md` file exists containing exactly that text — but SKILL.md never mentions `help.md`, and `help.md` is not otherwise loaded. As written, the model will improvise a help summary from SKILL.md and `help.md` will drift out of sync with no signal. Either SKILL.md must instruct it to read and print `help.md` verbatim, or `help.md` should be deleted. Note also that `help.md:13` and `README.md:33` both refer the reader to "the full skill file" / `SKILL.md` — fine for a repo browser, but the skill file is already in context when the skill fires, making that pointer a no-op at runtime.

**#11 — "dead reference" overloaded (Impact: Low, Confidence: High)**
The frontmatter description says the skill "catches dead, duplicate, or missing references," and most readers parse "dead" as broken-link. Step 4 names the same word for orphaned-but-listed entries. Two distinct concepts, one term, and one of them (broken links) is not actually implemented — see #7. Renaming step 4 to "orphaned reference check" would fix the ambiguity cheaply.

### Redundancy

**#9 — Argument list stated three times (Impact: Low, Confidence: High)**
`SKILL.md:10-13`, `help.md:3-11`, and `README.md:26-31` each carry a version of the argument list, in three different phrasings. Three copies is three chances to drift — already visible in the fact that none of them mentions accepting a file path (#1). The SKILL.md copy is the one that must exist; help.md is the printed artifact; the README's should be the shortest possible pointer.

### Token Efficiency & Progressive Disclosure

**#10 — Output Format template always loaded (Impact: Low, Confidence: Medium)**
SKILL.md is small (68 lines / ~3.6KB), so this is a modest win and correctly deprioritized. That said, the Output Format block (`SKILL.md:44-68`) is ~25 lines, roughly a third of the file, and is a fill-in template needed only when composing the final report — a natural `references/report-format.md` candidate, read on demand at report time. Similarly, `help.md` is correctly already a separate file (it just needs to be wired up per #3). Installation instructions are already correctly in README.md rather than SKILL.md — that part of the progressive-disclosure split is right. Recommend deferring this until #1–#5 are addressed, since fixing those will grow SKILL.md and change the calculus.

### Navigability & Structure

**#12 — README lacks verification and a worked example (Impact: Low, Confidence: Medium)**
For a skill intended for any writer, the README gives install commands but no way to confirm the install worked (e.g. run `/check-refs --help`), and no before/after example showing what a reference integrity pass actually does to a document. A short worked example would also surface the destructive-by-default behavior (#4) that a new user cannot currently anticipate. The `git clone` snippet is also subtly wrong: it clones into `./check-refs` and then symlinks `"$(pwd)/check-refs"`, which only works if the user does not `cd` into the clone — worth making explicit.

---

## Summary

The pass procedure itself is sound and well-reasoned — the primary-source check in particular is a genuinely valuable step most citation tools skip. The gaps are almost all inherited assumptions from the `writeli` origin: a known target document, a URL for every source, a LinkedIn-shaped document structure, and a user who wanted edits rather than an audit. Addressing #1–#5 would complete the generalization the extraction set out to achieve.
