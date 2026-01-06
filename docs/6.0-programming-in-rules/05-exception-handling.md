# Chapters 6-7: Exception Handling

## What is an Exception?

An exception indicates an exceptional or unusual circumstance during rules processing. When encountered:
- TIBCO Object Service Broker can raise a **system exception**
- User code can raise a **user-defined exception** via SIGNAL

## Exception Hierarchy

```
                           ERROR
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   ACCESSFAIL          ROUTINEFAIL            RULEFAIL
        │
        ├─── DATAREFERENCE
        ├─── DEFINITIONFAIL
        ├─── DISPLAYFAIL
        └─── INTEGRITYFAIL
                  │
                  ├─── COMMITLIMIT
                  ├─── CONVERSION
                  ├─── DELETEFAIL
                  ├─── GETFAIL
                  ├─── INSERTFAIL
                  ├─── LOCKFAIL
                  ├─── OVERFLOW
                  ├─── RANGERROR
                  ├─── REPLACEFAIL
                  ├─── SECURITYFAIL
                  ├─── SELECTIONFAIL
                  ├─── SERVERBUSY
                  ├─── SERVERERROR
                  ├─── SERVERFAIL
                  ├─── STRINGSIZE
                  ├─── UNASSIGNED
                  ├─── UNDERFLOW
                  ├─── VALIDATEFAIL
                  └─── ZERODIVIDE
        
   Also: EXECUTEFAIL, NULLVALUE, JAVAFAIL (standalone)
```

An exception handler traps any exception below it in the hierarchy.
- `ON ERROR:` traps all detectable errors
- `ON GETFAIL:` traps only GET failures

## System Exceptions Reference

| Exception | Condition |
|-----------|-----------|
| ACCESSFAIL | Table access error or browse mode update attempt |
| COMMITLIMIT | Maximum updates between sync points reached |
| CONVERSION | Invalid syntax or conversion failure |
| DATAREFERENCE | Error in selection criteria |
| DEFINITIONFAIL | Error in table definition |
| DELETEFAIL | Primary key doesn't exist or browse mode |
| DISPLAYFAIL | Error displaying screen |
| ERROR | Any error detected |
| EXECUTEFAIL | Error in child transaction |
| GETFAIL | No occurrence satisfies selection criteria |
| INSERTFAIL | Primary key exists or browse mode |
| INTEGRITYFAIL | Attempt to violate data integrity |
| JAVAFAIL | Java exception from external routine |
| LOCKFAIL | Another transaction prevents data access |
| NULLVALUE | Arithmetic on numeric null, or null passed as numeric arg |
| OVERFLOW | Value too large for target syntax (max value inserted) |
| RANGERROR | Routine argument out of allowable range |
| REPLACEFAIL | Primary key doesn't exist or browse mode |
| ROUTINEFAIL | Called routine fails, no specific exception available |
| RULEFAIL | Incorrect rules coding given correct dictionary |
| SECURITYFAIL | Permission denied |
| SELECTIONFAIL | Conversion error during WHERE clause evaluation |
| SERVERBUSY | External database server unavailable |
| SERVERERROR | External database server error |
| SERVERFAIL | Connection to external server broke |
| STRINGSIZE | Receiving string too short (value truncated) |
| UNASSIGNED | Reference to unassigned table field |
| UNDERFLOW | Value too small for target syntax (min value inserted) |
| VALIDATEFAIL | Invalid data in screen or table update |
| ZERODIVIDE | Division by zero attempted |

## Untrappable Exceptions

These cannot be caught with exception handlers:
- Rule called from unlisted library
- Rule does not exist
- Wrong number of arguments
- Memory/storage limits exceeded

---

## ON Statement

Defines an exception handler in the exception handler section.

### Syntax
```
ON exception_name:
   statement1;
   statement2;
```

### With Table Qualification
Limits scope to specific table:
```
ON GETFAIL table_name:
   handler_statements;
```

### Behavior
- No action sequence numbers in ON statements
- All statements execute when exception trapped

### Data Access Exceptions (Can Be Table-Qualified)
- ACCESSFAIL, DEFINITIONFAIL, DELETEFAIL, GETFAIL
- INSERTFAIL, LOCKFAIL, REPLACEFAIL, SECURITYFAIL
- SERVERBUSY, SERVERERROR, SERVERFAIL, VALIDATEFAIL

### Examples
```
ON GETFAIL MANAGER:
   CALL ENTER_MANAGER;

ON DEFINITIONFAIL:
   CALL ENDMSG('The report does not exist.');
```

---

## SIGNAL Statement

Raises a user-defined exception.

### Syntax
```
SIGNAL exception_name;
```

### Rules
- Can be caught by ON or UNTIL statements
- Must use literal exception name (no indirect reference)
- For indirect reference, use `$SIGNAL` tool
- Cannot be used inside FORALL loop
- In trigger/validation rules: must handle within calling hierarchy

### Examples
```
SIGNAL MISSING_INVOICE;

ON GETFAIL MANAGER:
   SIGNAL UNKNOWN_NAME;
```

---

## UNTIL Statement

Specifies exceptions that terminate a loop.

### Syntax
```
UNTIL exception_name [OR exception_name ...]:
   actions;
END;
```

### Structure
1. UNTIL keyword
2. Exception name(s) separated by OR
3. Colon `:`
4. Actions (no sequence numbers)
5. END statement

### Behavior When Exception Occurs
1. If in UNTIL's exception list: continue after END
2. If in ON statement: execute ON handler
3. If neither: bubble up hierarchy or terminate transaction

### Examples
```
-- Single exception
UNTIL NO_MORE_NUMBERS:
   MULTIPLIER = MULTIPLIER + 1;
   CALL CHECKDIGIT(LENGTH($NUMBER));
END;

-- With FORALL
FORALL SOURCETAB1 UNTIL GETFAIL:
   GET SOURCETAB2 WHERE LINE_NUM = SOURCETAB1.LINE_NUM;
   CALL COMPARELINES(SOURCETAB1.TEXT, SOURCETAB2.TEXT);
END;
```

---

## UNTIL ... DISPLAY Statement

Displays a screen repetitively until exception occurs.

### Syntax
```
UNTIL exception_name [OR ...] DISPLAY screen_name:
   actions;
END;
```

### Termination
Loop terminates when listed exception is detected and not handled inside loop.

### Example
```
UNTIL DONE DISPLAY QUERY_SCREEN:
   CALL PROCESS_FCNKEYS('QUERY_SCREEN');
END;
```

---

## Exception Handler Scope

### Basic Scope Rules
- Handler effective only during rule execution
- Traps exceptions in the rule AND descendant rules
- If not trapped in hierarchy: transaction terminates with error

### Multiple Handlers
When multiple handlers can catch same exception:
1. Handler in lowest rule (closest to error) wins
2. Within same rule: most specific handler wins

**Example**: Rule has both GETFAIL and ACCESSFAIL handlers
- GETFAIL occurs → GETFAIL handler invoked
- No GETFAIL handler → ACCESSFAIL handler invoked

### Limiting Data Access Exception Scope

Specify table name to limit scope:
```
ON GETFAIL MANAGER:      -- only traps GETFAIL on MANAGER table
   CALL ENTER_MANAGER;
```

Without table name:
```
ON GETFAIL:              -- traps GETFAIL on ANY table
   CALL HANDLE_ERROR;
```

---

## VALIDATEFAIL Exception for Screens

### When Issued
- User exits screen with Validation Exit key while data is invalid
- Data inserted into screen table fails reference checking

### Handling Invalid Data
- VALIDATEFAIL can manage the exception
- Invalid occurrences still present and readable
- Completely incompatible data converted to null

### Example
```
EMPADD(EMPNO);

---------------------------------------------------------------------------
------------------------------------------------------------+--------------
EMPLOYEE_INFO.EMPNO = EMPNO;                                | 1
INSERT EMPLOYEE_INFO('NEW_EMPLOYEE');                       | 2
UNTIL EXIT_DISPLAY DISPLAY NEW_EMPLOYEE:                    | 3
   CALL PROCESS_PFKEY('NEW_EMPLOYEE');                      |
END;                                                        |
---------------------------------------------------------------------------
ON VALIDATEFAIL:
   GET EMPLOYEE_INFO('NEW_EMPLOYEE') WHERE EMPNO = EMPNO;
   CALL ENDMSG('SEE AUDIT TRAIL FOR INVALID DATA.');
   CALL @OPENDSN('AUDIT.TRAIL');
   CALL @WRITEDSN(EMPLOYEE_INFO.EMPNO);
   CALL @WRITEDSN(EMPLOYEE_INFO.LNAME);
   CALL @CLOSEDSN;
```

---

## Complete Exception Handling Example

```
MGRINFO(DATE, ID);

---------------------------------------------------------------------------
VALID_DATE(DATE);                                           | Y N
------------------------------------------------------------+--------------
GET MANAGER WHERE MANAGER_NUM = ID;                         | 1
SIGNAL DATE_INVALID;                                        |   1
---------------------------------------------------------------------------
ON GETFAIL MANAGER:
   CALL ENTER_MANAGER;
ON DATE_INVALID:
   CALL ENDMSG('Invalid date entered.');
```

This rule:
1. Validates the date using a functional rule
2. If valid: gets manager record (traps GETFAIL)
3. If invalid: signals user-defined DATE_INVALID exception

## Maximum Exception Handlers

A rule can have up to **32** exception handlers total, including:
- ON statements
- UNTIL statements (including UNTIL...DISPLAY and FORALL...UNTIL)
