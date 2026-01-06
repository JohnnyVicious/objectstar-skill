---
name: objectstar
description: Analyze, refactor, and migrate TIBCO Objectstar (Object Service Broker) legacy mainframe applications. Use when working with Objectstar/OSB source code files, rules, screen definitions, or table definitions. Triggers on .osr, .osb file extensions, or code containing Objectstar patterns like condition quadrants (Y N columns), FORALL loops, TRANSFERCALL, parameterized tables, or rules with LOCAL declarations.
---

# Objectstar Language Skill

Objectstar is a rules-based 4GL mainframe platform using condition quadrants instead of IF/THEN/ELSE, tables instead of objects, and rules instead of programs. Extended support ends March 30, 2027.

## Core Workflow

1. **Identify Objectstar code** → Look for condition quadrants, FORALL loops, parameterized tables
2. **Understand rule structure** → Parse the four-section format (declaration, conditions, actions, handlers)
3. **Analyze data flow** → Track LOCAL variable scope (visible in descendant rules)
4. **For refactoring** → Apply patterns from [patterns.md](references/patterns.md)
5. **For migration** → Map constructs using [migration.md](references/migration.md)

## Rule Structure (Fundamental Unit)

Every Objectstar rule has four sections:

```
RULE_NAME(arg1, arg2);              -- 1. Declaration
LOCAL var1, var2;                   -- Local variables (untyped)
---------------------------------------------------------------------------
condition1;                         | Y N N    -- 2. Conditions (Y/N columns)
condition2;                         |   Y N
------------------------------------------------------------+---------------
action1;                            | 1        -- 3. Actions (numbered sequence)
action2;                            |   1
action3;                            |     1
---------------------------------------------------------------------------
ON exception_name:                             -- 4. Exception handlers
    handler_statement;
```

**Condition quadrant logic**: Conditions evaluate top-to-bottom. The Y/N pattern in each column determines which action column executes. Numbers indicate execution order within a column.

## Critical Language Rules

### Variable Scoping (Non-Standard)
LOCAL variables are visible in the declaring rule AND all descendant rules (rules called from within). This breaks encapsulation—track carefully during migration.

### No Traditional Loops
- **No WHILE/FOR** — Use `FORALL` for iteration over table occurrences
- **No IF/THEN/ELSE** — Use condition quadrant Y/N patterns
- Loop termination via `UNTIL exception:` pattern

### Data Types Are Dual-Classified
Semantic types (C=Count, D=Date, I=Identifier, L=Logical, Q=Quantity, S=String) define meaning. Syntax types (B=Binary, C=Char, P=Packed, V=Variable, F=Float, UN=Unicode) define storage. Variables inherit type from assigned values.

### Everything Is a Table
Screens, reports, data stores, even rule storage—all tables. Table types: TDS (permanent), SCR (screen), TEM (temporary), SES (session), MAP (memory mapping), SUB (subview).

## Essential Syntax Quick Reference

### Rule Invocation
```
CALL rule(args);                    -- Same transaction, returns
EXECUTE IN UPDATE rule(args);       -- Child transaction
SCHEDULE TO QUEUE rule(args);       -- Async batch
TRANSFERCALL rule(args);            -- Transfer control, no return
```

### Data Operations
```
GET TABLE WHERE field = value;
GET TABLE(param) ORDERED DESCENDING field WITH MINLOCK;
INSERT TABLE;
REPLACE TABLE;
DELETE TABLE WHERE field = value;
```

### Iteration
```
FORALL TABLE(param) WHERE condition
    ORDERED ASCENDING field
    UNTIL GETFAIL:
    -- process each occurrence
END;

UNTIL EXIT_DISPLAY DISPLAY SCREEN:
    -- process until exit
END;
```

### WHERE Clause Operators
```
=, >, <, >=, <=       -- Comparison
¬= or NOT =           -- Not equal
& or AND              -- Conjunction
| or OR               -- Disjunction
¬ or NOT              -- Negation
LIKE 'ABC*'           -- * matches any string
LIKE 'AB?D'           -- ? matches single char
```

### Exception Handling
```
ON GETFAIL:           -- Record not found
ON GETFAIL TABLE:     -- Table-specific
ON ACCESSFAIL:        -- Any access failure
ON ERROR:             -- Catch-all (use last)
SIGNAL CUSTOM_ERR;    -- Raise exception
```

### String Operations
```
str1 || str2                        -- Concatenation
LENGTH(s), SUBSTRING(s,start,len)
UPPERCASE(s), LOWERCASE(s)
MATCH(s, pattern), PAD(s,len,char,just)
```

## Reserved Keywords

```
AND        ASCENDING    BROWSE      CALL        COMMIT      CONTINUE
DELETE     DESCENDING   DISPLAY     END         EXECUTE     FORALL
GET        IN           INSERT      LIKE        LOCAL       NOT
NULL       ON           OR          ORDERED     PRINT       REPLACE
RETURN     ROLLBACK     SCHEDULE    SIGNAL      TO          TRANSFERCALL
UNTIL      UPDATE       WHERE
```

## Common Exception Hierarchy

```
ERROR (catch-all)
├── ACCESSFAIL
│   ├── GETFAIL, INSERTFAIL, REPLACEFAIL, DELETEFAIL
│   └── DISPLAYFAIL, SERVERFAIL
├── INTEGRITYFAIL
│   ├── COMMITLIMIT, LOCKFAIL, VALIDATEFAIL
└── RULEFAIL
    ├── ZERODIVIDE, OVERFLOW, CONVERSION
    └── NULLVALUE, STRINGSIZE, SECURITYFAIL
```

## Analysis Checklist

When analyzing Objectstar code:

- [ ] Identify all LOCAL variables and trace their scope chain
- [ ] Map condition quadrant logic to equivalent branching
- [ ] Document parameterized table usage (e.g., `TABLE(REGION)`)
- [ ] List all exception handlers and their scope
- [ ] Identify event rules (Trigger, Validation, Derived, Location)
- [ ] Note TRANSFERCALL usage (breaks return flow)
- [ ] Check for non-standard operators (¬, ||, &, |)

## Refactoring Guidelines

1. **Simplify condition quadrants** — Consolidate redundant Y/N columns
2. **Extract common patterns** — Repeated FORALL/GET sequences → separate rules
3. **Standardize exception handling** — Table-specific before generic
4. **Document scope dependencies** — LOCAL variables used across rule calls
5. **Add Browse hints** — Use Browse Mode for read-only operations

## Migration Priorities

For Java migration, address these semantic gaps (see [migration.md](references/migration.md) for details):

1. **Untyped LOCAL variables** → Type inference or wrapper classes
2. **Descendant scope visibility** → Custom scope chain management
3. **Parameterized tables** → Composite keys + filtered queries
4. **Condition quadrants** → Decision tables or switch expressions
5. **TRANSFERCALL** → Explicit control flow refactoring

## Reference Files

- [syntax.md](references/syntax.md) — Complete syntax reference with all data types
- [patterns.md](references/patterns.md) — Common code patterns with examples
- [migration.md](references/migration.md) — Java migration mappings and strategies
- [tools.md](references/tools.md) — Built-in shareable tools reference
