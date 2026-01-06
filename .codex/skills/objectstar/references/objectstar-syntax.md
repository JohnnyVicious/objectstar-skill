# Objectstar Syntax Reference

This document provides key syntax patterns for TIBCO Objectstar rules language (Object Service Broker), applicable to both OTP and batch contexts.

## Structure of a Rule

Objectstar rules use a four-section format with condition quadrants (Y/N matrix), **not** traditional IF/THEN/ELSE:

```
RULE_NAME(param1, param2);          -- Declaration
LOCAL var1, var2;                   -- Local variables (untyped)
---------------------------------------------------------------------------
condition1;                         | Y N N    -- Conditions (Y/N columns)
condition2;                         |   Y N
------------------------------------------------------------+---------------
action1;                            | 1        -- Actions (numbered sequence)
action2;                            |   1
action3;                            |     1
---------------------------------------------------------------------------
ON exception_name:                             -- Exception handlers
    handler_statement;
```

## Condition Quadrant Patterns

### Basic Two-Way Branch
```
record_found;                       | Y N
----------------------------------------+-----
CALL PROCESS_RECORD;                | 1
CALL HANDLE_NOT_FOUND;              |   1
```

### Multiple Conditions
```
CUST_TYPE = 'PREMIUM';              | Y N N
ORDER_AMT > 1000;                   |   Y N
----------------------------------------+-----
DISCOUNT = 0.20;                    | 1        -- Premium customer
DISCOUNT = 0.10;                    |   1      -- Large order
DISCOUNT = 0.05;                    |     1    -- Default
```

### Sequential Actions in One Column
```
valid_input;                        | Y N
----------------------------------------+-----
CALL VALIDATE;                      | 1
CALL PROCESS;                       | 2        -- Runs after VALIDATE
CALL SAVE;                          | 3        -- Runs after PROCESS
SIGNAL INVALID_INPUT;               |   1
```

## Data Access

### GET (Retrieve)
```
GET TABLE WHERE KEY = value;
GET TABLE(param) ORDERED ASCENDING field;
GET TABLE WITH MINLOCK;
GET TABLE WITH NOLOCK;
```

### FORALL (Iteration)
```
FORALL TABLE WHERE condition
    ORDERED ASCENDING field
    UNTIL GETFAIL:
    -- process each occurrence
END;
```

### REPLACE / INSERT / DELETE
```
INSERT TABLE;
REPLACE TABLE;
DELETE TABLE WHERE KEY = value;
```

## Control Flow
- **CALL / EXECUTE / TRANSFERCALL**
```plaintext
CALL RuleName;
EXECUTE RuleName;
TRANSFERCALL RuleName;
```

- **UNTIL Exception Loop**
```plaintext
CALL FORALLA('TABLE', ...);
UNTIL ENDFILE:
  CALL FORALLB('TABLE');
END;
CALL FORALLE('TABLE');
```

## Transactions
```plaintext
COMMIT;
ROLLBACK;
```

## Exception Handling
```
ON GETFAIL:              -- Record not found
    handler_statement;

ON GETFAIL TABLE:        -- Table-specific handler
    SIGNAL ERROR;

ON ACCESSFAIL:           -- Any access failure
    ROLLBACK;

ON ERROR:                -- Catch-all (use last)
    ROLLBACK;
    CALL MSGLOG('Error occurred');
```

### Common Exception Hierarchy
```
ERROR (catch-all)
├── ACCESSFAIL
│   ├── GETFAIL, INSERTFAIL, REPLACEFAIL, DELETEFAIL
│   └── DISPLAYFAIL, SERVERFAIL
├── INTEGRITYFAIL
│   ├── COMMITLIMIT, LOCKFAIL, VALIDATEFAIL
└── RULEFAIL
    ├── ZERODIVIDE, OVERFLOW, CONVERSION
    └── NULLVALUE, STRINGSIZE
```

## Messaging and Screens
```plaintext
CALL MSGLOG("Message");
CALL SCREENMSG('SCR', "Field missing");
DISPLAY SCREEN_NAME;
```

## Special Functions
```plaintext
$USER()       -- current user
$DATE()       -- current date
$GETOPT(name) -- retrieve system option
```

## Notes
- Semicolons (`;`) terminate statements
- Comments use `/* ... */`
- Variables are typeless, scoped to the rule unless global tables are used
- BROWSE mode restricts writes; UPDATE mode required for DML

