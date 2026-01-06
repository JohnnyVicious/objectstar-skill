---
name: analyzing-objectstar
description: >
  Skill for understanding, editing, analyzing, and migrating TIBCO ObjectStar (Object Service Broker) code used in mainframe OTP and batch applications. Activate when user is working with ObjectStar rules, asks about mainframe modernization, or legacy 4GL code involving GET, FORALL, or EXCEPTION blocks.
license: Proprietary
metadata:
  author: Legacy Modernization Team
  version: "1.0"
---

# ObjectStar Programming Skill

This skill enables AI agents to interpret, refactor, and migrate ObjectStar legacy code. ObjectStar (also known as TIBCO Object Service Broker) is a mainframe-focused 4GL used for screen-based OTP applications and batch processing. It operates with structured rules language, embedded exception handling, and integrated data access.

## When to Use This Skill
- User provides ObjectStar source code or mentions `.OSB` / MetaStore / screen rules
- User asks about migrating a legacy mainframe system using TIBCO OSB
- User asks about understanding code using `GET`, `FORALL`, `ON GETFAIL`, `DISPLAY`, `REPLACE`, etc.
- Static analysis, refactoring, or modernization of ObjectStar code

## Language Overview

ObjectStar rules are declarative procedures with four sections:
- **DECLARATION**: Rule name, parameters, LOCAL variables
- **CONDITIONS**: Y/N matrix logic (no IF/THEN/ELSE exists)
- **ACTIONS**: Numbered execution sequence per condition column
- **EXCEPTIONS**: Structured handlers (GETFAIL, LOCKFAIL, etc.)

Used in both interactive OTP (3270 screens) and batch (job stream) contexts.

## Condition Quadrants (Critical Concept)

Objectstar has **no IF/THEN/ELSE**. All conditional logic uses condition quadrants — a Y/N matrix that determines which actions execute.

### Rule Structure
```
RULE_NAME(param1, param2);
LOCAL var1, var2;                   -- Untyped local variables
---------------------------------------------------------------------------
condition1;                         | Y N N    -- Condition rows
condition2;                         |   Y N
------------------------------------------------------------+---------------
action1;                            | 1        -- Action rows (numbered)
action2;                            |   1
action3;                            |     1
---------------------------------------------------------------------------
ON GETFAIL:                                    -- Exception handlers
    handler_action;
```

### How It Works
1. Conditions evaluate top-to-bottom
2. Each column represents a unique combination (like switch cases)
3. Y/N pattern determines which column matches
4. Numbers indicate execution order within that column

### Example: Discount Calculation
```
CALC_DISCOUNT(CUST_TYPE, ORDER_AMT);
LOCAL DISCOUNT;
---------------------------------------------------------------------------
CUST_TYPE = 'PREMIUM';              | Y N N    -- Column 1: Premium
ORDER_AMT > 1000;                   |   Y N    -- Column 2: Large order
------------------------------------------------------------+---------------
DISCOUNT = 0.20;                    | 1        -- 20% for premium
DISCOUNT = 0.10;                    |   1      -- 10% for large orders
DISCOUNT = 0.05;                    |     1    -- 5% default
```

Equivalent pseudo-code:
```
if (CUST_TYPE == 'PREMIUM') { DISCOUNT = 0.20; }
else if (ORDER_AMT > 1000)  { DISCOUNT = 0.10; }
else                        { DISCOUNT = 0.05; }
```

### Key Points
- **No IF/ENDIF** — Always use condition quadrants
- **Columns are mutually exclusive** — Only one column executes
- **Numbers set order** — Multiple actions in a column run in numbered sequence
- **Blank cells** — Condition not evaluated for that column

## Syntax Reference
See [ObjectStar_Syntax.md](references/ObjectStar_Syntax.md) for language keywords and examples.

## Best Practices
✅ Always trap expected exceptions (GETFAIL, INSERTFAIL) locally  
✅ Use primary key WHERE clauses for GET and REPLACE to ensure intent list consistency  
✅ Use EXECUTE for sub-transactions that should commit independently  
✅ Commit periodically in batch jobs to avoid COMMITLIMIT errors  
✅ Use screen tables for passing data in OTP rules and session tables in batch  
✅ Favor BROWSE mode when writing read-only rules  

## Common Pitfalls
❌ Using `ON ERROR` as a general flow control — leads to masked bugs  
❌ FORALL with nested loops over large tables — performance killer  
❌ No COMMIT in long batch loops — causes memory and lock issues  
❌ Hardcoding dataset names and magic values — hinders migration  
❌ Reliance on implicit global state instead of parameter passing  

See [ObjectStar_Pitfalls.md](references/ObjectStar_Pitfalls.md) for deeper explanations.

## Idioms and Patterns
- **Exception loop idiom**:
  ```
  CALL FORALLA('TABLE', ...);
  UNTIL ENDFILE:
    CALL FORALLB('TABLE');
  END;
  CALL FORALLE('TABLE');
  ```

- **Screen data validation rule**:
  ```
  GET CUSTOMER WHERE ID = SCR.ID;
  ON GETFAIL:
    CALL SCREENMSG('SCR', 'Customer not found');
    SIGNAL ERROR;
  ```

- **Batch processing with commit window** (using condition quadrant):
  ```
  BATCH_PROCESS;
  LOCAL COUNT;
  ---------------------------------------------------------------------------
  COUNT = 0;
  FORALL INVOICES UNTIL GETFAIL:
      CALL PROCESS_INVOICE;
      COUNT = COUNT + 1;
      COUNT >= 1000;                      | Y N
      ----------------------------------------+-----
      COMMIT;                             | 1
      COUNT = 0;                          | 2
  END;
  COMMIT;  -- Final commit for remaining
  ```
  Note: Objectstar has no IF/THEN/ELSE — use condition quadrants (Y/N columns) for conditional logic.

## Refactoring Instructions
1. Identify use of outdated patterns (e.g. ON ERROR without handler logic)
2. Wrap database access with clear ON handlers
3. Isolate screen logic from core business logic
4. Convert monolithic rules into modular subrules
5. Externalize config (e.g. file names, limits) into control tables

## Static Analysis Checklist
- [ ] Any unhandled exceptions (GETFAIL, LOCKFAIL, etc.)?
- [ ] Large FORALLs without WHERE filters?
- [ ] Broad ON ERROR blocks without rollback or logging?
- [ ] Hardcoded screen transitions or filenames?
- [ ] Rule too long (>200 lines)? Consider modularizing.

## Migration Strategy
- Separate screen vs. data logic into callable modules
- Identify and replicate exception model in target platform (e.g., Java try/catch)
- Translate GET/REPLACE to SQL SELECT/UPDATE with transaction control
- Replace screen tables with data transfer objects (DTOs) or form models
- Use a translation matrix (see reference) to map idioms to modern equivalents

See [ObjectStar_MigrationGuide.md](references/ObjectStar_MigrationGuide.md).

## Resources
- [ObjectStar Syntax](references/ObjectStar_Syntax.md)
- [Built-in Tools](references/ObjectStar_BuiltInTools.md)
- [Known Pitfalls](references/ObjectStar_Pitfalls.md)
- [Migration Guide](references/ObjectStar_MigrationGuide.md)

---

Use this skill to safely read, update, analyze, or migrate ObjectStar code. Treat rules as structured procedures with explicit error handling. Preserve transactional semantics. Avoid global side-effects and large in-memory transactions when modernizing.

