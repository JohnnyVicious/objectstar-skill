# TIBCO Object Service Broker - Programming in Rules Reference

Source: TIBCO® Object Service Broker Programming in Rules, Software Release 6.0, July 2012

## Document Structure

This reference is split into the following files for easier consumption:

| File | Contents |
|------|----------|
| [01-introduction.md](01-introduction.md) | Introduction to OSB Rules - what they are, how to define and execute them |
| [02-rule-composition.md](02-rule-composition.md) | Components of a rule: declaration, conditions, actions, exception handlers |
| [03-supported-characters.md](03-supported-characters.md) | Lexical elements, character sets, quotation marks, national language support |
| [04-action-statements.md](04-action-statements.md) | All action statements: CALL, COMMIT, DELETE, DISPLAY, EXECUTE, FORALL, GET, INSERT, PRINT, REPLACE, RETURN, ROLLBACK, SCHEDULE, TRANSFERCALL, WHERE |
| [05-exception-handling.md](05-exception-handling.md) | Exception handling: system exceptions, ON/SIGNAL/UNTIL statements |
| [06-expressions-operators.md](06-expressions-operators.md) | Expressions, data syntax, semantic types, operators, indirect referencing |
| [07-transaction-processing.md](07-transaction-processing.md) | Transactions, synchronization points, nesting, locks |
| [08-conditional-null-arithmetic.md](08-conditional-null-arithmetic.md) | Conditional processing, null handling, arithmetic operations |
| [09-rules-libraries-editor.md](09-rules-libraries-editor.md) | Rules libraries, Rule Editor usage, editing commands |
| [10-execution-debugging.md](10-execution-debugging.md) | Standard execution, batch processing, Rule Debugger |
| [11-syntax-reference.md](11-syntax-reference.md) | BNF notation and complete syntax specification |

## Quick Reference

### Rule Structure
```
RULE_NAME(ARG1, ARG2);
LOCAL VAR1, VAR2;
---------------------------------------------------------------------------
CONDITION1;                                               | Y N N
CONDITION2;                                               |   Y N
------------------------------------------------------------+---------------
ACTION1;                                                  | 1
ACTION2;                                                  |   1
ACTION3;                                                  |     1
---------------------------------------------------------------------------
ON EXCEPTION_NAME:
   HANDLER_ACTIONS;
```

### Key Concepts
- **Rules Language**: Programming language for creating OSB applications
- **Condition Quadrant**: Y/N columns that determine which actions execute based on conditions
- **Action Sequence Numbers**: Numbers determining execution order of actions
- **Exception Handlers**: ON statements that trap and handle exceptions
