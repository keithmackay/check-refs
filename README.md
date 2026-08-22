# check-refs

A Claude Code skill that runs a reference integrity pass on any article or document with numbered inline citations and applies the fixes directly unless `--dry-run` is passed - verifies ordering, catches orphaned, duplicate, and broken references, and flags secondary sources that should point to the primary one.

## Highlights

- **Convention detection** - preserves the document's existing citation style (`[N]`, `[^N]`, superscript, `(N)`) and reference-list heading rather than imposing its own
- **Coverage check** - flags unattributed statistics, quotes, and claims
- **Orphaned and duplicate detection** - removes references never cited in the body, merges duplicate sources under one number
- **Broken-link check** - catches 404s and unrelated redirects, with an archive.org fallback
- **Non-URL sources** - books, interviews, and internal data are checked for completeness instead of being forced to have a link
- **Primary source check** - swaps secondary summaries for the original report, paper, or announcement, and never fabricates one when none can be found
- **Sequential renumbering, done last** - after every other change is settled

## Getting Started

### Installation

Run these commands from the directory you want the clone to live in - do not `cd` into the clone first; the `$(pwd)` in the symlink commands must still be the parent directory.

```bash
git clone https://github.com/keithmackay/check-refs.git

# Global install (available in all projects)
ln -s "$(pwd)/check-refs" ~/.claude/skills/check-refs

# Project-local install (available only in one project)
ln -s "$(pwd)/check-refs" /path/to/your/project/.claude/skills/check-refs
```

Verify the install by running `/check-refs --help` - it should print the usage summary.

## Usage

Run `/check-refs --help` for the full argument list.

### Example

Before - references out of order, one never cited, one duplicated:

```markdown
Cloud spend hit $300 billion [2], up 40% year over year [3].
Gartner attributes most of the growth to AI workloads [4].

## References
1. [Unused industry overview](https://example.com/overview)
2. [Gartner cloud spend release](https://gartner.com/cloud-spend)
3. [Blog summarizing Gartner's numbers](https://blog.example.com/gartner-recap)
4. [Gartner cloud spend release](https://gartner.com/cloud-spend)
```

After - orphan removed, duplicate merged, secondary swapped for primary, renumbered:

```markdown
Cloud spend hit $300 billion [1], up 40% year over year [1].
Gartner attributes most of the growth to AI workloads [1].

## References
1. [Gartner cloud spend release](https://gartner.com/cloud-spend)
```

## Contributing

Pull requests are welcome - fork the repo, make your change, and open a PR describing what it does and why.

## License

[MIT](LICENSE)
