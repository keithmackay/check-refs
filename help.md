check-refs - Verify reference/citation integrity in any document with numbered footnotes.

This pass EDITS the target document directly unless --dry-run is passed.

USAGE
  /check-refs          Resolves the target document (confirms it if already named,
                        asks if not), confirms that the document will be edited,
                        then runs the full reference integrity pass and applies
                        the fixes: detect and preserve the document's existing
                        citation convention, coverage check for unattributed
                        claims, number-to-link validation, orphaned-reference
                        removal, duplicate merging, a primary-source check that
                        flags secondary summaries which should point to the
                        original source, a broken-link check with archive.org
                        fallback, a completeness check for sources with no URL,
                        and finally sequential renumbering.

  /check-refs --dry-run
                       Runs the same pass and produces the same report, but makes
                       no changes. Every action that would have been applied is
                       listed under a WOULD DO: heading.

  /check-refs --help   Prints this summary. Takes no other action.

See the full skill file for the complete pass procedure and reference format.
