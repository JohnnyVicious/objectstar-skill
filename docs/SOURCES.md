# External Sources & References

This folder contains reference documentation gathered from external sources to support the Objectstar skills.

## Local Documentation

### 6.0-programming-in-rules/

**Source**: [TIBCO® Object Service Broker Programming in Rules](https://docs.tibco.com/pub/object_service_broker/6.0.0_july_2012/doc/pdf/tib_osb_processing.pdf?id=3), Software Release 6.0, July 2012

Comprehensive markdown reference extracted from official TIBCO documentation:

| File | Contents |
|------|----------|
| `00-INDEX.md` | Document structure and quick reference |
| `01-introduction.md` | Introduction to OSB Rules |
| `02-rule-composition.md` | Rule components: declaration, conditions, actions, handlers |
| `03-supported-characters.md` | Lexical elements, character sets |
| `04-action-statements.md` | All action statements (CALL, GET, FORALL, etc.) |
| `05-exception-handling.md` | ON/SIGNAL/UNTIL exception handling |
| `06-expressions-operators.md` | Expressions, data types, operators |
| `07-transaction-processing.md` | Transactions, locks, synchronization |
| `08-conditional-null-arithmetic.md` | Conditional processing, null handling |
| `09-rules-libraries-editor.md` | Rules libraries, Rule Editor |
| `10-execution-debugging.md` | Execution and Rule Debugger |
| `11-syntax-reference.md` | BNF notation and syntax specification |

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
