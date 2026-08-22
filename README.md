# check-refs

A Claude Code skill that runs a reference integrity pass on any article or document with numbered inline citations - verifies ordering, catches dead/duplicate references, and flags secondary sources that should point to the primary one.

## Highlights

- **Sequential ordering check** - references renumbered to match first-citation order in the body
- **Coverage check** - flags unattributed statistics, quotes, and claims
- **Dead and duplicate detection** - removes orphaned references, merges duplicate sources under one number
- **Primary source check** - flags secondary summaries that should be swapped for the original report, paper, or announcement

## Getting Started

### Installation

```bash
git clone https://github.com/keithmackay/check-refs.git

# Global install (available in all projects)
ln -s "$(pwd)/check-refs" ~/.claude/skills/check-refs

# Project-local install (available only in one project)
ln -s "$(pwd)/check-refs" /path/to/your/project/.claude/skills/check-refs
```

## Usage

```
/check-refs           # run the full reference integrity pass
/check-refs --help    # reads and displays help.md verbatim, takes no other action
```

See `SKILL.md` for the full pass procedure and reference format.

## Contributing

Pull requests are welcome - fork the repo, make your change, and open a PR describing what it does and why.

## License

[MIT](LICENSE)
