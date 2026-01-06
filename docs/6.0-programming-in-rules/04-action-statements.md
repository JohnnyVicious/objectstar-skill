# Chapters 4-5: Action Statements

## Overview

Action statements are executable statements in the body of a rule. Each action ends with:
- Semicolon `;` for most statements
- Colon `:` for FORALL and UNTIL statements

## Categories of Action Statements

| Category | Statements |
|----------|------------|
| Assignment | `=` (simple), `.*` (by-name) |
| Database Synchronization | COMMIT, ROLLBACK |
| Looping | FORALL, UNTIL, UNTIL...DISPLAY |
| Output | DISPLAY, PRINT, DISPLAY & TRANSFERCALL |
| Rule Invocation | CALL, EXECUTE, RETURN, SCHEDULE, TRANSFERCALL |
| Table Access | DELETE, FORALL, GET, INSERT, REPLACE |

---

## CALL Statement

Invokes a rule within the current transaction scope.

### Syntax
```
CALL rule_name(arg1, arg2, ...);
CALL table.field;              -- indirect reference
```

### Behavior
- Control returns to next action after called rule completes
- All arguments must be specified
- Can invoke directly or indirectly

### Exceptions
| Exception | Condition |
|-----------|-----------|
| ROUTINEFAIL | Called routine fails, cause cannot be signaled otherwise |
| RULEFAIL | Table access from call is incorrect |

### Examples
```
CALL EMPLOYEE_COUNT(10);
CALL ENDMSG(COUNT || ' EMPLOYEES IN DEPARTMENT#' || DEPT);
CALL FCNKEYS.ROUTINE;          -- indirect call
```

---

## COMMIT Statement

Applies all changes made to TDS and external data since last synchronization point.

### Syntax
```
COMMIT;
```

### Key Points
- Not required at transaction end (implicit commit occurs)
- FORALL operates only on committed data
- GET without primary key operates only on committed data
- Does **not** release locks (released at transaction end)

### Exception
| Exception | Condition |
|-----------|-----------|
| COMMITLIMIT | Maximum updates between sync points reached |

### Example - Handling Commit Limit
```
INSERT EMPLOYEES('MIDWEST');
____________________________________
ON COMMITLIMIT:
   COMMIT;
```

### Example - Commit Per Table Instance
```
FORALL $EMPLOYEES:
   FORALL EMPLOYEES($EMPLOYEES.REGION):
      EMPLOYEES.SALARY = EMPLOYEES.SALARY + 100;
      REPLACE EMPLOYEES($EMPLOYEES.REGION);
   END;
   COMMIT;                      -- commit after each instance
END;
```

---

## DELETE Statement

Removes an occurrence from a table.

### Syntax
```
DELETE table_name WHERE primary_key = value;
DELETE table_name;             -- uses current buffer
```

### Requirements
Primary key must be available via:
- Previous GET or FORALL
- Explicit selection on primary key
- Assigned value to primary key

### Selection
Only equality (`=`) on primary key field is allowed.

### Exceptions
| Exception | Condition |
|-----------|-----------|
| DATAREFERENCE | Error in selection criteria |
| DELETEFAIL | Occurrence does not exist |
| INTEGRITYFAIL | Attempt to violate data integrity |

### Examples
```
DELETE STUDENT WHERE ID = '810883';

GET STUDENT;
DELETE STUDENT;                -- deletes first occurrence
```

---

## DISPLAY Statement

Shows a specified screen.

### Syntax
```
DISPLAY screen_name;
```

### Usage
- Use INSERT before DISPLAY to populate screen table
- Access entered data using GET/FORALL on screen tables

### Exceptions
| Exception | Condition |
|-----------|-----------|
| DISPLAYFAIL | Error displaying screen |
| DEFINITIONFAIL | Screen does not exist |

### Example
```
FORALL MANAGERS:
   MANAGER_DATA.* = MANAGERS.*;
   INSERT MANAGER_DATA('MANAGER_SCR');
END;
DISPLAY MANAGER_SCR;
```

---

## DISPLAY & TRANSFERCALL Statement

Enables pseudo-conversational processing by displaying screen after ending transaction.

### Syntax
```
DISPLAY screen_name & TRANSFERCALL rule_name;
```

### Benefits
- Improves concurrent access to resources
- Frees tables but preserves screen context
- Screen environment persists across transaction boundary

### Exceptions
| Exception | Condition |
|-----------|-----------|
| DISPLAYFAIL | Error displaying screen |
| DEFINITIONFAIL | Screen does not exist |

### Example
```
DISPLAY QUERY_SCREEN & TRANSFERCALL PROCESS_QUERY;
```
When user presses function key or Enter, PROCESS_QUERY begins as new transaction.

---

## EXECUTE Statement

Starts a new transaction to invoke a rule.

### Syntax
```
EXECUTE rule_name(args);
EXECUTE IN BROWSE rule_name;   -- browse mode
EXECUTE IN UPDATE rule_name;   -- update mode
EXECUTE table.field;           -- indirect
```

### Behavior
- Creates nested transaction
- Control returns to original transaction after completion
- Can specify browse or update mode

### Exception
| Exception | Condition |
|-----------|-----------|
| EXECUTEFAIL | Error in child transaction |

### Examples
```
EXECUTE EMPLOYEE_COUNT(10);
EXECUTE FCNKEYS.ROUTINE;       -- indirect
```

---

## FORALL Statement

Looping construct that processes a set of occurrences.

### Syntax
```
FORALL table_name [WHERE conditions] [ORDERED field] [UNTIL exception]:
   actions;
END;
```

### Structure
1. Table name with optional WHERE, ORDERED, UNTIL clauses
2. Colon `:` after clauses
3. Actions (no action sequence numbers inside loop)
4. END statement

### Ordering
Without ORDERED clause, occurrences selected in storage order or by:
- ORD field in table definition
- ORDERED clause(s)

### Exceptions
- No exceptions if no occurrences found (loop body skipped)
- SIGNAL not allowed inside FORALL
- DATAREFERENCE if selection criteria error

### Termination
- All matching occurrences processed, OR
- Unhandled exception inside loop

**Note**: Table buffer undefined after all occurrences processed.

### Examples
```
-- Basic loop
FORALL MANAGER:
   CALL PRINT_MANAGER;
END;

-- With WHERE clause
FORALL EMPLOYEES WHERE REGION = 'MIDWEST'
   & HIREDATE > *.BIRTHDATE + 40:
   ...
END;

-- With LIKE pattern matching
FORALL MANAGER WHERE MANAGER_NAME LIKE '*SON':
   ...
END;

-- With UNTIL exception
FORALL MANAGER UNTIL GETFAIL:
   ...
END;

-- With ORDERED (multiple fields)
FORALL CARS WHERE CITY = INVOICE.CITY 
   ORDERED DESCENDING PRICE 
   & ORDERED ASCENDING MODEL:
   CALL $PRINTLINE('CAR ID ' || CARS.LICENSE# ||
      ' MODEL ' || CARS.MODEL);
END;
```

---

## GET Statement

Retrieves first occurrence matching selection criteria.

### Syntax
```
GET table_name [WHERE conditions] [ORDERED field] [WITH MINLOCK];
```

### Data Retrieval
- With unique primary key: uncommitted data retrieved
- Otherwise: committed data retrieved

### Locking
- Unique primary key: share lock on row
- Non-unique: share lock on table/instance
- WITH MINLOCK: temporary table lock, then row lock only

### Exceptions
| Exception | Condition |
|-----------|-----------|
| DATAREFERENCE | Error in selection criteria |
| GETFAIL | No occurrences meet criteria |
| INTEGRITYFAIL | Attempt to violate data integrity |

### Examples
```
GET STUDENTS;                  -- first occurrence
GET STUDENTS WHERE STUDENT# = '810883';
GET MONTHS WHERE MONTH = MM & DAYS >= DD;
GET EMPLOYEES WHERE REGION='MIDWEST' & DEPTNO = INPUT.DEPT
   ORDERED DESCENDING LNAME;
```

---

## INSERT Statement

Adds a new occurrence to a table.

### Syntax
```
INSERT table_name;
INSERT table_name WHERE parameter = value;
```

### Requirements
- Occurrences must have unique primary keys
- No field selection allowed
- WHERE clause specifies only parameter values

### Exceptions
| Exception | Condition |
|-----------|-----------|
| DATAREFERENCE | Error in selection criteria |
| INSERTFAIL | Primary key already exists |
| INTEGRITYFAIL | Attempt to violate data integrity |

### Examples
```
INSERT STUDENT;
INSERT CARS WHERE CITY = INPUT.CITY;
INSERT EXPENSE_DATA WHERE SCREEN = 'EMPLOYEE_EXPENSE';
```

---

## ORDERED Clause

Specifies retrieval order for GET or FORALL.

### Syntax
```
ORDERED [ASCENDING|DESCENDING] field_name
```

### Behavior
- Default order: ASCENDING by primary key
- Multiple ORDERED clauses joined with `&`
- 7+ ORDERED clauses triggers external sort

### Examples
```
GET EMPLOYEES('MIDWEST') & DEPTNO = INPUT.DEPT
   ORDERED DESCENDING LNAME;

FORALL CARS WHERE CITY = INVOICE.CITY
   ORDERED DESCENDING PRICE
   & ORDERED ASCENDING MODEL:
```

---

## PRINT Statement

Sends a report to predetermined destination.

### Syntax
```
PRINT report_name;
PRINT(argument_name);          -- indirect
PRINT table.field;             -- indirect
```

### Destination
Determined by session options or `$SETRPTMEDIUM` tool.

### Exception
| Exception | Condition |
|-----------|-----------|
| DEFINITIONFAIL | Report does not exist |

### Examples
```
PRINT DEPT_SALARY;
PRINT(REPORTNAME);             -- indirect via argument
```

---

## REPLACE Statement

Updates an occurrence in the database.

### Syntax
```
REPLACE table_name;
REPLACE table_name WHERE parameter = value;
```

### Requirements
- First retrieve data with GET or FORALL
- To change primary key: DELETE old, INSERT new
- WHERE specifies only parameter values

### Exceptions
| Exception | Condition |
|-----------|-----------|
| DATAREFERENCE | Error in selection criteria |
| INTEGRITYFAIL | Attempt to violate data integrity |
| REPLACEFAIL | Primary key doesn't exist, or browse mode |

### Examples
```
REPLACE STUDENTS;
REPLACE CARS(INPUT.CITY);
```

---

## RETURN Statement

Returns a specified result from a function.

### Syntax
```
RETURN(expression);
RETURN(constant);
```

A rule with RETURN is a **function**.

### Examples
```
RETURN('Y');
RETURN(EMPLOYEE_SALARY.SALARY + INCREASE);
```

---

## ROLLBACK Statement

Discards all changes since last synchronization point.

### Syntax
```
ROLLBACK;
```

### Behavior
- Does not release locks (released at transaction end)

### Example
```
FORALL HIREDATE:
   GET EMPLOYEES('MIDWEST') WHERE EMPNO=HIREDATE.EMPNO;
   EMPLOYEES.HIREDATE = HIREDATE.DATE;
   REPLACE EMPLOYEES('MIDWEST');
END;
---------------------------------------------------------
ON GETFAIL:
   ROLLBACK;
```

---

## SCHEDULE Statement

Enables asynchronous batch processing.

### Syntax
```
SCHEDULE rule_name(args);                    -- immediate
SCHEDULE IN BROWSE rule_name;                -- browse mode
SCHEDULE TO 'queue_name' rule_name;          -- to queue (z/OS)
```

### Behavior
- Uses @SCHEDULEMODEL table instance
- TO clause sends to batch queue (z/OS only)
- Values with spaces automatically wrapped in quotes (Open Systems)

### Exceptions
| Exception | Condition |
|-----------|-----------|
| LOCKFAIL | Lock held on @SCHEDULEMODEL (z/OS) |
| SECURITYFAIL | Security violation on @SCHEDULEMODEL (z/OS) |

### Examples
```
SCHEDULE PRINT_INVOICE(INPUT.INVOICE#);
SCHEDULE TO 'WEEKEND' CLEANUP WHERE LOCATION = INPUT.CITY;
```

---

## TRANSFERCALL Statement

Terminates current transaction and starts a new one.

### Syntax
```
TRANSFERCALL rule_name(args);
TRANSFERCALL IN BROWSE rule_name;
TRANSFERCALL IN UPDATE rule_name;
TRANSFERCALL table.field;                    -- indirect
```

### Behavior
- Ends current transaction completely
- Called rule runs as new transaction
- From nested transaction: control returns to parent after completion

### Examples
```
TRANSFERCALL SELECT_RENTAL(NEAREST_CAR(LOCATION));
TRANSFERCALL IN BROWSE SELECT_RENTAL WHERE RENTAL_LOC = NEAREST_CAR(LOCATION);
TRANSFERCALL FCNKEYS.ROUTINE;
```

---

## WHERE Clause

Selects occurrences, table instances, or rule arguments.

### Syntax
```
WHERE field = value
WHERE field LIKE 'pattern'
WHERE condition1 & condition2
WHERE condition1 | condition2
WHERE NOT(condition)
```

### Pattern Matching with LIKE
| Pattern | Meaning |
|---------|---------|
| `?` | One character of any kind |
| `*` | Zero or more characters |
| Other | Must be present |

### Current Table Reference
Use `*` to refer to current occurrence:
```
GET MANAGER WHERE MANAGER_NUM = *.USERID;
```

### Operators
- `&` (AND) and `|` (OR) for combining conditions
- `¬` (NOT) for negation: `WHERE NOT(fieldname LIKE 'value')`

### Examples
```
GET EMPLOYEES WHERE REGION = 'MIDWEST'
   & NOT(STATE_PROV='ONT' | STATE_PROV='MINN');

DELETE MANAGER WHERE MANAGER_NUM = 80002;
DELETE EMPLOYEES WHERE REGION = 'MIDWEST';
```
