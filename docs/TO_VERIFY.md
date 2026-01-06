# To Verify

Items in the skills documentation that need verification against official TIBCO sources or practitioners with Objectstar experience.

## ✅ Verified: Condition Quadrant Visual Format

**Status**: Fully verified from official TIBCO documentation

**Source**: [TIBCO® Object Service Broker Programming in Rules](https://docs.tibco.com/pub/object_service_broker/6.0.0_july_2012/doc/pdf/tib_osb_processing.pdf?id=3), Software Release 6.0, July 2012

The official documentation in `docs/6.0-programming-in-rules/02-rule-composition.md` shows the exact visual format:

```
RULE EDITOR ===>                                                    SCROLL: P
RULE_NAME(ARG1, ARG2);                              ← Declaration
LOCAL VAR1, VAR2;                                   ← Local Variables
---------------------------------------------------------------------------
CONDITION1;                                         | Y N N    ← Conditions
CONDITION2;                                         |   Y N       with Y/N
------------------------------------------------------------+-------------- Quadrant
ACTION1;                                            | 1       ← Actions with
ACTION2;                                            |   1        Sequence
ACTION3;                                            |     1      Numbers
---------------------------------------------------------------------------
ON EXCEPTION_NAME:                                  ← Exception Handlers
   HANDLER_STATEMENT;
```

| Concept | Official Definition | Status |
|---------|---------------------|--------|
| Y/N evaluation | "Condition: An expression evaluated for its truth or logical value **(Y or N)**" | ✅ Verified |
| Action Sequence Numbers | "An identifier that determines which rules statements are executed when a given condition is satisfied" | ✅ Verified |
| Visual format with pipe separators | Shown in Rule Editor examples throughout Chapter 2 | ✅ Verified |
| Dashed line separators | Shown in official documentation | ✅ Verified |
| Maximum 6 conditions per rule | "Up to **6 conditions** per rule" (Chapter 10) | ✅ Verified |

**Conclusion**: The Y/N column format, pipe separators, dashed lines, and action sequence numbers are all confirmed by official TIBCO documentation.

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
| Locks non-existent keys | [FreeSoft](https://freesoftus.com/services/application-code-conversion/objectstar-conversion/) | "when a table is queried for a specific primary key value and no record is found, the specified primary key will be 'locked'" |
| Breaks encapsulation | [FreeSoft](https://freesoftus.com/services/application-code-conversion/objectstar-conversion/) | "ObjectStar's semantics breaks the principle of Encapsulation" |

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
