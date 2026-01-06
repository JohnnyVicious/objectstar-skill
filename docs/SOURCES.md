# External Sources & References

This folder contains reference documentation gathered from external sources to support the Objectstar skills.

## Online References

See `CLAUDE.md` for the full list of URLs. Key sources:

### TIBCO Official Documentation
- Quick Reference (HTML) — Keywords, operators, statements
- Programming in Rules (PDF) — Comprehensive language guide
- Getting Started (PDF) — Tutorials and introduction
- Managing Data (PDF) — Table types, data operations

### What We've Verified

| Claim | Source | Status |
|-------|--------|--------|
| No IF/THEN/ELSE keywords | TIBCO Quick Reference | Verified |
| FORALL with UNTIL exception patterns | TIBCO Quick Reference | Verified |
| Support ends March 30, 2027 | TIBCO Support Article | Verified |
| Y/N condition quadrant format | Original skill files | Unverified from public sources |

## Adding New Documentation

If you download PDFs or capture important information from the TIBCO docs:

1. Save PDFs to this folder with descriptive names
2. For large PDFs, consider extracting relevant sections to markdown
3. Update `CLAUDE.md` External References section
4. Note the source URL and access date

## Objectstar History

Based on web research:

- **Original name**: Huron (created by Amdahl Corporation, early 1990s)
- **Renamed**: ObjectStar (mid 1990s, ported to Unix/Windows NT)
- **Current name**: TIBCO Object Service Broker (OSB)
- **End of extended support**: March 30, 2027

The language is notable for:
- Unusual Rules Language syntax
- Proprietary interpreted language and database
- Tight integration with query language
- LOCAL variables with no declared types
- Non-standard scoping (descendant visibility)
