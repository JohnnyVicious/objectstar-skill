# Objectstar Syntax Reference

## Data Types

### Semantic Types (Business Meaning)

| Type | Code | Description | Valid Operations |
|------|------|-------------|------------------|
| Count | C | Integers for integral counting | Arithmetic, relational, assignment |
| Date | D | Date values (must include year) | Arithmetic (+/-), relational |
| Identifier | I | Unique identifiers | Relational, assignment only |
| Logical | L | Boolean Yes/No | Logical operators, assignment |
| Quantity | Q | Real numbers for measurement | Full arithmetic, relational |
| String | S | Character string data | Concatenation, relational |

### Syntax Types (Storage Representation)

| Syntax | Description | Valid Lengths |
|--------|-------------|---------------|
| B | Binary (signed) | 2-12 bytes |
| C | Fixed length character | 1-254 bytes |
| P | Packed decimal (signed) | 1-16 bytes (1-31 digits) |
| V | Variable length character | 1-256 bytes |
| F | Floating point | 4, 8, or 16 bytes |
| UN | Unicode | 2+ bytes (must be even) |

### Default Uninitialized Values
- Character context: empty string
- Numeric context: zero
- Logical context: 'N'

## Table Types

| Type | Code | Purpose | Persistence |
|------|------|---------|-------------|
| TDS | Native data store | Primary storage | Permanent |
| SCR | Screen table | UI data | Transaction duration |
| TEM | Temporary table | Working storage | Transaction duration |
| SES | Session table | Session-level | Session duration |
| EES | Execution Environment | EE-level | EE duration |
| MAP | Memory mapping | DSECT/STRUCT | As mapped |
| SUB | Subview | Filtered view | Non-persistent |

## External Database Gateways

| Code | Database |
|------|----------|
| DB2 | IBM DB2 |
| IMS | IMS/DB |
| IDM | CA-IDMS |
| ADA | Adabas |
| VSM | VSAM |
| SLK | ODBC/Oracle |
| DAT | CA-Datacom |
| 204 | Model 204 |

## Statement Reference

### Data Access

```
-- GET (retrieve)
GET table;
GET table WHERE field = value;
GET table(param) WHERE field = value;
GET table ORDERED ASCENDING field;
GET table ORDERED DESCENDING field;
GET table WITH MINLOCK;
GET table WITH NOLOCK;

-- INSERT (create)
INSERT table;
INSERT table(param);
INSERT table WHERE field = value;

-- REPLACE (update)
REPLACE table;
REPLACE table WHERE field = value;

-- DELETE (remove)
DELETE table;
DELETE table WHERE field = value;
```

### Rule Invocation

```
-- CALL (synchronous, same transaction)
CALL rule_name;
CALL rule_name(arg1, arg2);
CALL rule_name WHERE parm1 = 'value1' & parm2 = 'value2';

-- EXECUTE (child transaction)
EXECUTE rule_name;
EXECUTE IN UPDATE rule_name(args);
EXECUTE IN BROWSE rule_name;

-- SCHEDULE (async batch)
SCHEDULE rule_name;
SCHEDULE TO queue_name rule_name(args);

-- TRANSFERCALL (no return)
TRANSFERCALL rule_name(args);

-- RETURN (from functional rule)
RETURN;
RETURN(expression);
```

### Control Flow

```
-- FORALL iteration
FORALL table WHERE condition:
    statements;
END;

FORALL table(param) WHERE condition
    ORDERED ASCENDING field
    UNTIL exception_name:
    statements;
END;

-- UNTIL loop
UNTIL exception_name:
    statements;
END;

UNTIL exc1 | exc2:
    statements;
END;

-- DISPLAY loop
DISPLAY screen;

UNTIL EXIT_DISPLAY DISPLAY screen:
    statements;
END;

UNTIL EXIT_DISPLAY | ERROR DISPLAY screen:
    statements;
END;
```

### Transaction Control

```
COMMIT;           -- Commit current transaction
ROLLBACK;         -- Rollback current transaction
```

### Exception Handling

```
-- Handler syntax
ON exception_name:
    handler_statements;

ON exception_name table_name:
    table_specific_handler;

-- Raise exception
SIGNAL exception_name;
```

## Operators

### Arithmetic
```
+    Addition
-    Subtraction
*    Multiplication
/    Division
```

### Relational
```
=    Equal
¬=   Not equal (also: NOT =)
>    Greater than
<    Less than
>=   Greater or equal
<=   Less or equal
```

### Logical
```
&    AND (also: AND)
|    OR (also: OR)
¬    NOT (also: NOT)
```

### String
```
||   Concatenation
```

### Pattern Matching (in WHERE LIKE)
```
*    Matches any string (zero or more chars)
?    Matches single character
```

## Condition Quadrant Patterns

### Basic Pattern
```
condition1;              | Y N
condition2;              |   Y
--------------------------+-----
action_for_Y_only;       | 1
action_for_Y_Y;          |   1
```

### Multiple Columns
```
condition1;              | Y Y N N
condition2;              | Y N Y N
--------------------------+--------
action_YY;               | 1
action_YN;               |   1
action_NY;               |     1
action_NN;               |       1
```

### Sequenced Actions
```
condition1;              | Y N
--------------------------+-----
first_action;            | 1
second_action;           | 2
third_action;            | 2 1
```
Numbers indicate execution order: column 1 runs actions 1,2,2; column 2 runs action 1.

## Field References

```
table.field              -- Qualified field reference
field                    -- Unqualified (current table context)
table(param).field       -- Parameterized table field
```

## Literals

```
'string value'           -- String literal
123                      -- Numeric literal
123.45                   -- Decimal literal
'Y' or 'N'               -- Logical literals
```

## Comments

```
-- Single line comment (double dash)
/* Multi-line
   comment */
```

## Screen Handling

### Display Statements
```
DISPLAY screen_name;
UNTIL EXIT_DISPLAY DISPLAY screen_name:
    process_statements;
END;
```

### Screen Functions
```
ENTERKEY(screen)              -- Last function key pressed
CURSORFIELD(screen)           -- Field with cursor
SETCURSOR(screen, table, field)  -- Position cursor
SCREENMSG(screen, message)    -- Display message
DELETESCREENDATA(screen)      -- Clear screen data
$SETATTRIBUTE(screen, table, field, attr, flag)
```

### Standard Function Keys
```
PF1  - Help
PF3  - Save/Exit
PF7  - Scroll Up/Back
PF8  - Scroll Down/Forward
PF12 - Cancel
```

## Event Rules

Event rules fire automatically on data operations.

| Type | Code | Purpose |
|------|------|---------|
| Trigger | T | Execute actions on access |
| Validation | V | Validate before update |
| Derived | D | Calculate field values |
| Location | L | Determine data location |

Access type specifiers: I (Insert), R (Replace), D (Delete), G (Get), A (All)
