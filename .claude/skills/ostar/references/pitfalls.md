# Objectstar Anti-Patterns and Pitfalls

Common mistakes and code smells to avoid when working with Objectstar code. Each anti-pattern includes a BAD example and a BETTER refactoring approach.

## Deeply Nested FORALL

**Impact**: Poor performance, hard to debug, memory pressure

```
-- BAD: Deep nesting, poor performance
FORALL A:
    FORALL B:
        FORALL C:
            FORALL D:
                -- processing
            END;
        END;
    END;
END;

-- BETTER: Extract inner loops to separate rules
FORALL A:
    CALL PROCESS_B_LEVEL(A.KEY);
END;
```

## Missing Exception Handlers

**Impact**: Silent failures, data corruption, difficult debugging

```
-- BAD: No handlers, fails silently
GET CUSTOMERS WHERE CUST# = INPUT_ID;
CALL PROCESS_CUSTOMER;

-- BETTER: Handle expected exceptions
GET CUSTOMERS WHERE CUST# = INPUT_ID;
CALL PROCESS_CUSTOMER;
ON GETFAIL:
    CALL MSGLOG('Customer not found: ' || INPUT_ID);
    SIGNAL CUSTOMER_NOT_FOUND;
```

## Hardcoded Values

**Impact**: Maintenance nightmare, environment-specific bugs, migration difficulties

```
-- BAD: Magic numbers
DISCOUNT = ORDER_AMT * 0.15;

-- BETTER: Use configuration table
GET CONFIG WHERE CONFIG_KEY = 'DISCOUNT_RATE';
DISCOUNT = ORDER_AMT * CONFIG.CONFIG_VALUE;
```

## Overly Complex Condition Quadrant

**Impact**: Unreadable code, error-prone maintenance, testing difficulties

```
-- BAD: Too many columns, hard to read
cond1;  | Y Y Y Y N N N N
cond2;  | Y Y N N Y Y N N
cond3;  | Y N Y N Y N Y N
--------+----------------
act1;   | 1
act2;   |   1
...

-- BETTER: Split into multiple rules with clear names
CALL PROCESS_TYPE_A;    -- When cond1=Y, cond2=Y, cond3=Y
CALL PROCESS_TYPE_B;    -- When cond1=Y, cond2=Y, cond3=N
-- etc.
```

## Implicit Global State

**Impact**: Hidden dependencies, testing difficulties, migration bugs

```
-- BAD: Using screen table as implicit data passing
-- Rule A writes to SCRATCH_SCREEN, Rule B reads from it
-- Hidden dependency, testing difficulties

CALL PROCESS_STEP_1;    -- Writes to SCRATCH_SCREEN.VALUE1
CALL PROCESS_STEP_2;    -- Expects SCRATCH_SCREEN.VALUE1 to be set

-- BETTER: Pass values explicitly via parameters
CALL PROCESS_STEP_1(INPUT_VAL, RESULT_VAL);
CALL PROCESS_STEP_2(RESULT_VAL, FINAL_VAL);

-- Or use session tables with clear naming
GET STEP1_RESULTS(SESSION_ID);
```

**Why**: ObjectStar's semantics breaks the principle of encapsulation. Implicit state creates hidden dependencies between rules, making testing and migration difficult. ([Verified](https://freesoftus.com/services/application-code-conversion/Objectstar-conversion/))

## Reliance on Locking Side Effects

**Impact**: Migration bugs, non-portable code, unexpected behavior in RDBMS

```
-- BAD: Using GET to lock a non-existent key
GET LOCKS_TABLE WHERE KEY = 'PROCESS_XYZ';
-- In ObjectStar, this locks 'PROCESS_XYZ' even if no record exists
-- Used as a semaphore/mutex pattern

-- BETTER: Use explicit locking table with actual records
GET SEMAPHORES WHERE SEM_NAME = 'PROCESS_XYZ' WITH MINLOCK;
SEMAPHORES.IN_USE = 'Y';
REPLACE SEMAPHORES;
-- Release lock
SEMAPHORES.IN_USE = 'N';
REPLACE SEMAPHORES;
```

**Why**: ObjectStar locks non-existent primary keys — a behavior difficult to replicate in RDBMS systems. This anti-pattern causes migration bugs. ([Verified](https://freesoftus.com/services/application-code-conversion/Objectstar-conversion/))

## Repeated Code Blocks

**Impact**: Inconsistent behavior, maintenance burden, bug propagation

```
-- BAD: Same tax calculation in multiple rules
-- In CALC_ORDER:
TAX = SUBTOTAL * 0.08;
-- In CALC_INVOICE:
TAX = SUBTOTAL * 0.08;
-- In CALC_ESTIMATE:
TAX = SUBTOTAL * 0.08;

-- BETTER: Extract to shared utility rule
CALC_TAX(AMOUNT);
LOCAL TAX_RATE;
---------------------------------------------------------------------------
GET CONFIG WHERE KEY = 'TAX_RATE';
RETURN(AMOUNT * CONFIG.VALUE);
---------------------------------------------------------------------------
ON GETFAIL:
    RETURN(AMOUNT * 0.08);  -- Default fallback
```

**Why**: Duplicated business logic leads to inconsistent behavior when one copy is updated but others are forgotten. Extract to shared rules invoked via CALL.

## Catch-All ON ERROR Without Logic

**Impact**: Masked bugs, silent failures, difficult debugging

```
-- BAD: Swallowing all errors
CALL PROCESS_DATA;
ON ERROR:
    -- Nothing here, or just CONTINUE

-- BETTER: Log and handle appropriately
CALL PROCESS_DATA;
ON ERROR:
    CALL MSGLOG('Unexpected error in PROCESS_DATA');
    ROLLBACK;
    SIGNAL PROCESS_FAILED;
```

## No COMMIT in Long Batch Loops

**Impact**: COMMITLIMIT errors, memory exhaustion, lock contention

```
-- BAD: No commits in large batch
FORALL INVOICES UNTIL GETFAIL:
    CALL PROCESS_INVOICE;
    REPLACE INVOICES;
END;
-- May hit COMMITLIMIT with thousands of records

-- BETTER: Periodic commits with counter
LOCAL COUNT;
COUNT = 0;
FORALL INVOICES UNTIL GETFAIL:
    CALL PROCESS_INVOICE;
    REPLACE INVOICES;
    COUNT = COUNT + 1;
    COUNT >= 1000;                      | Y N
    ----------------------------------------+-----
    COMMIT;                             | 1
    COUNT = 0;                          | 2
END;
COMMIT;  -- Final commit for remaining
```

## Quick Reference: Pitfall Detection

| Code Smell | Look For | Risk |
|------------|----------|------|
| Nested FORALL | 3+ levels of FORALL/END | Performance |
| Missing handlers | GET/REPLACE without ON | Silent failures |
| Magic numbers | Hardcoded values in logic | Maintenance |
| Complex quadrant | 5+ condition columns | Readability |
| Global state | SCRATCH tables, unmarked dependencies | Testing |
| Lock tricks | GET on non-existent keys | Migration |
| Duplicated logic | Same calculation in multiple rules | Consistency |
| Empty ON ERROR | ON ERROR with no statements | Debugging |
| No batch COMMIT | FORALL without periodic COMMIT | Resources |
