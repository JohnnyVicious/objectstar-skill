# To Verify

Items in the skills documentation that need verification against official TIBCO sources or practitioners with Objectstar experience.

## Partially Verified: Condition Quadrant Visual Format

**Status**: Core concepts verified, exact visual formatting is reconstructed

The Y/N column format used throughout the skills:

```
condition1;                         | Y N N
condition2;                         |   Y N
------------------------------------------------------------+---------------
action1;                            | 1
action2;                            |   1
action3;                            |     1
```

**Verified from TIBCO Glossary** ([source](https://docs.tibco.com/pub/object_service_broker/5.2.0_august_2010/html/tib_osb_getting_started/glossary.htm)):

| Concept | Official Definition | Status |
|---------|---------------------|--------|
| Y/N evaluation | "Condition: An expression evaluated for its truth or logical value **(Y or N)**" | ✅ Verified |
| Action Sequence Numbers | "An identifier that determines which rules statements are executed when a given condition is satisfied" | ✅ Verified |
| Action definition | "A statement within a rule. An action can occupy more than one line within a rule." | ✅ Verified |

**Still reconstructed** (not found in public docs):
- The exact visual format with pipe separators (`|`)
- The dashed line separators (`-----------+-----`)
- The column alignment and spacing
- Whether numbers appear as `| 1` or in some other format

**Conclusion**: The Y/N logic and action sequence numbering are officially documented. The visual representation in the skills is a reasonable interpretation but the exact character-by-character layout would need confirmation from an actual OSB rule editor.

---

## Verified Items

For reference, these items have been verified:

| Item | Source | Verification |
|------|--------|--------------|
| Conditions evaluate to Y or N | [TIBCO Glossary](https://docs.tibco.com/pub/object_service_broker/5.2.0_august_2010/html/tib_osb_getting_started/glossary.htm) | "An expression evaluated for its truth or logical value (Y or N)" |
| Action Sequence Numbers exist | [TIBCO Glossary](https://docs.tibco.com/pub/object_service_broker/5.2.0_august_2010/html/tib_osb_getting_started/glossary.htm) | "An identifier that determines which rules statements are executed" |
| No IF/THEN/ELSE keywords | [TIBCO Quick Reference](https://docs.tibco.com/pub/object_service_broker/5.2.0_august_2010/html/tib_osb_quick_reference/qref.htm) | Searched for IF, THEN, ELSE, ENDIF — not found |
| FORALL with UNTIL pattern | TIBCO Quick Reference | Listed in control flow statements |
| ON exception handling | TIBCO Quick Reference | Listed syntax: `ON exception_name:` |
| Support ends March 2027 | [TIBCO Support Article](https://support.tibco.com/s/article/Tibco-KnowledgeArticle-Article-44626) | Official announcement |
| Originally named Huron | Multiple web sources | Created by Amdahl Corporation |
| LOCAL variables untyped | FreeSoft migration docs | "ObjectStar's LOCAL data has no declared type" |

---

## How to Update This File

When verifying an item:
1. Move it from "Unverified" to "Verified Items" table
2. Add the source URL or reference
3. Describe how verification was done
4. Commit with `docs: verify [item name]`

When adding new unverified items:
1. Add a section with **Status**, **What we know**, **What is reconstructed**, **How to verify**
2. Be specific about what needs checking
