---
name: objectstar-language
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
ObjectStar rules are declarative procedures with sections for:
- **CONDITION**: optional trigger logic (for screen events or inputs)
- **ACTION**: primary code (data logic, rule calls, control)
- **EXCEPTION**: structured handlers for recoverable errors (GETFAIL, LOCKFAIL, etc.)

It is used in both interactive OTP (3270 screens) and batch (job stream) contexts.

## Syntax Reference
See [references/ObjectStar_Syntax.md](references/ObjectStar_Syntax.md) for language keywords and examples.

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

See [references/ObjectStar_Pitfalls.md](references/ObjectStar_Pitfalls.md) for deeper explanations.

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

- **Batch processing with commit window**:
  ```
  FORALL INVOICES:
    PROCESS;
    IF $COUNTER % 1000 = 0:
      COMMIT;
    ENDIF;
  END;
  COMMIT;
  ```

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

See [references/ObjectStar_MigrationGuide.md](references/ObjectStar_MigrationGuide.md).

## Resources
- [ObjectStar Syntax](references/ObjectStar_Syntax.md)
- [Known Pitfalls](references/ObjectStar_Pitfalls.md)
- [Migration Guide](references/ObjectStar_MigrationGuide.md)

---

Use this skill to safely read, update, analyze, or migrate ObjectStar code. Treat rules as structured procedures with explicit error handling. Preserve transactional semantics. Avoid global side-effects and large in-memory transactions when modernizing.

