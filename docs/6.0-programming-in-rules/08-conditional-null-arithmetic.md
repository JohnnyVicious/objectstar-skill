# Chapters 10-12: Conditional Processing, Null Handling, and Arithmetic

## Conditional Processing

### What is Conditional Processing?

Determines whether an expression is true or false, then controls which actions execute.

### Uses
- Relational operator comparisons
- Y/N values passed from calling rule
- Logical field values (table.field)
- Functional rules returning logical values

### Maximum Conditions
Up to **6 conditions** per rule.

---

## Condition Examples

### Expression as Condition
```
EMPLOYEE_COUNT(DEPT);
LOCAL COUNT;
---------------------------------------------------------------------------
DEPT > 0;                                                   | Y N
------------------------------------------------------------+--------------
COUNT = 0;                                                  | 1 1
FORALL EMPLOYEES WHERE REGION = 'MIDWEST' & DEPTNO = DEPT   | 2
   ORDERED DESCENDING LNAME:                                |
   CALL MSGLOG(EMPLOYEES.LNAME);                            |
   COUNT = COUNT + 1;                                       |
END;                                                        |
CALL ENDMSG(COUNT || ' EMPLOYEES IN DEPARTMENT# ' || DEPT); | 3 2
---------------------------------------------------------------------------
ON CONVERSION:
   CALL ENDMSG('Value for DEPT must be numeric');
```

### Argument as Condition
```
EMPLOYEE_UPDATE(NEW);

---------------------------------------------------------------------------
NEW;                                                        | Y N
------------------------------------------------------------+--------------
CALL ENDMSG('NEW EMPLOYEE');                                | 1
CALL ENDMSG('EXISTING EMPLOYEE');                           |   1
---------------------------------------------------------------------------
```

**Note**: Any character value other than 'Y' evaluates as 'N'. Numeric values cause errors.

### Table.Field as Condition

**Rule 1: Get the data**
```
GET_EXEMPT1(NAME);

---------------------------------------------------------------------------
-----------------------------------------------------------+--------------
GET MANAGER WHERE MANAGER_NAME = NAME;                      | 1
CALL GET_EXEMPT2(NAME);                                     | 2
---------------------------------------------------------------------------
ON GETFAIL:
   CALL ENDMSG(NAME || ' IS AN INVALID NAME');
```

**Rule 2: Use the field value**
```
GET_EXEMPT2(NAME);

---------------------------------------------------------------------------
MANAGER.EXEMPT;                                             | Y N
-----------------------------------------------------------+---------------
CALL ENDMSG(NAME || ' IS EXEMPT');                          | 1
CALL ENDMSG(NAME || ' IS NOT EXEMPT');                      |   1
---------------------------------------------------------------------------
```

### Functional Rule as Condition
```
PROCESS_CMD(USER_COMMAND);

---------------------------------------------------------------------------
PATTERN_MATCH(COMMAND, 's*');                               | Y N N
PATTERN_MATCH(COMMAND, 'd*');                               |   Y N
------------------------------------------------------------+--------------
CALL SELECT_COMMAND;                                        | 1
CALL DELETE_COMMAND;                                        |   1
CALL SCREENMSG(USERCOMMAND || ' IS AN INVALID COMMAND');    |     1
---------------------------------------------------------------------------
```

**Note**: Non-'Y' return values evaluate as 'N'. Numeric returns cause errors.

---

## Null Processing

### What is a Null Value?

A null represents data of no value—either no data exists or explicitly assigned null.

### Null Ordering
Null is **less than** any other value. Appears first in ASCENDING order.

### How Fields Get Null Values
1. INSERT without assignment and no default value
2. Assign NULL or empty string ('') without default
3. New field added to existing table

### Null Syntax Types

| Syntax | Null Behavior |
|--------|---------------|
| C, UN, V, W | Character null = empty string ('') |
| B, P, F | Numeric null (special behavior) |
| RD | Acts as either depending on context |

The keyword `NULL` represents syntax B (numeric) null.

---

## Null Manipulation

### Allowed Operations
- Logical manipulation only
- Arithmetic operations raise NULLVALUE exception

### Using NULL Keyword

| Context | Syntax |
|---------|--------|
| Rule condition | `expression = NULL` |
| Assignment | `target = NULL` |
| Passing by name | `WHERE arg = NULL` |
| Function return | `RETURN(NULL)` |
| Selection | `field = NULL` |
| Passing by position | `(NULL, value)` |
| Initialize fields | `table.* = NULL` |

### Restrictions
- Cannot use in expressions: `NULL + 1` ✗
- Cannot use to replace empty string when executing from workbench

---

## Null Behavior by Context

### Conversion

| From | To | Result |
|------|-----|--------|
| Numeric null | C, UN, V, W | Empty string |
| Character null / '' | B, P, F | Zero |
| RD null | C, UN, V, W | Empty string |
| RD null | B, P, F | Numeric null |
| Char/'' or numeric null | RD | RD null |

### Assignment

| Value | Assigned To | Result |
|-------|-------------|--------|
| Char null / '' | Character field | Empty string |
| Char null / '' | Date field | Numeric null |
| Char null / '' | Local variable | Empty string |
| Char null / '' | Numeric field | Zero |
| Numeric null | Character field | Empty string |
| Numeric null | Date field | Numeric null |
| Numeric null | Local variable | Numeric null |
| Numeric null | Numeric field | Numeric null |
| RD null | Character field | Empty string |
| RD null | Date field | Numeric null |
| RD null | Local variable | RD null |
| RD null | Numeric field | Numeric null |

### In Expressions

| Evaluation Point | Operation | Result |
|------------------|-----------|--------|
| Selection (WHERE) | Arithmetic on numeric/RD null | Occurrence discarded, continue |
| Execution | Arithmetic on numeric/RD null | NULLVALUE exception |
| Any | Arithmetic on character null | Zero |
| Any | Concatenation on numeric/RD null | Empty string |

### Relational Expressions (True Conditions)

| Left | Operator | Right |
|------|----------|-------|
| Numeric null | = | Numeric null |
| Numeric null | = | '' |
| Numeric null | < | Any non-null |
| Character null | = | '' |
| Character null | = | 0 |
| RD null | = | RD null |
| RD null | = | Character null |
| RD null | = | Numeric null |
| RD null | < | Any non-null |

### Special Cases

| Situation | Behavior |
|-----------|----------|
| Primary key | Cannot be null (INSERTFAIL/REPLACEFAIL) |
| IDgen table | Null primary key allowed |
| Secondary key | Numeric null not allowed |
| Required field | Null not allowed (INSERTFAIL/REPLACEFAIL) |
| Default value | Replaces null before INSERT/REPLACE |
| Unassigned field | Can reference by assigning null first |
| Indirect reference | Null not allowed (ERROR raised) |
| Local variable init | Empty string ('') |

---

## Arithmetic Processing

### Arithmetic Operators

| Operator | Operation |
|----------|-----------|
| `**` | Exponentiation |
| `*` | Multiplication |
| `/` | Division |
| `+` | Addition |
| `-` | Subtraction |
| `-` | Unary minus |
| `+` | Unary plus |

### Permitted Syntaxes
- Binary (B)
- Floating point (F)
- Packed decimal (P)

String syntaxes (C, V, W, UN, RD) are converted to numeric before operation.

### String to Numeric Conversion

| String Content | Converted To |
|----------------|--------------|
| All digits | Binary |
| Contains period (.) | Packed decimal |
| Has E/e exponent | Floating point |

If B or P cannot hold value, F is used.

### Number to String Conversion
Non-significant zeros removed:
- `00010` → `10`
- `040.0170` → `40.017`
- `00500.0200E-013` → `500.02E-13`

---

## Permissible Arithmetic Operations

### Addition (+)

|  | Count | Date | Quantity | Typeless |
|--|-------|------|----------|----------|
| Count | Count | Date* | – | Count |
| Date | Date* | – | – | Date |
| Quantity | – | – | Quantity | Quantity |
| Typeless | Count | Date | Quantity | Typeless |

*Date + Count only valid if Count is binary syntax

### Subtraction (-)

|  | Count | Date | Quantity | Typeless |
|--|-------|------|----------|----------|
| Count | Count | – | – | Count |
| Date | Date | Count | – | Date |
| Quantity | – | – | Quantity | Quantity |
| Typeless | Count | – | Quantity | Typeless |

### Multiplication, Division, Exponentiation (*, /, **)

|  | Count | Quantity | Typeless |
|--|-------|----------|----------|
| Count | Count | Quantity | Count |
| Quantity | Quantity | Quantity | Quantity |
| Typeless | Count | Quantity | Typeless |

---

## Resultant Syntax from Arithmetic

| Operand 1 | Operand 2 | Result |
|-----------|-----------|--------|
| B(L1) | B(L2) | B(4) |
| B(L1) | P(L2,D2) | P(16,D) |
| B(L1) | F(L2) | F(L2) |
| P(L1,D1) | B(L2) | P(16,D) |
| P(L1,D1) | P(L2,D2) | P(16,D) |
| P(L1,D1) | F(L2) | F(L2) |
| F(L1) | B(L2) | F(L1) |
| F(L1) | P(L2,D2) | F(L1) |
| F(L1) | F(L2) | F(MIN(L1,L2)) |

**Note**: Overflow causes computation to be attempted in floating point.

### Decimal Places for Packed Results

| Operation | Decimal Places |
|-----------|----------------|
| + or - (B,P) | D2 |
| + or - (P,B) | D1 |
| + or - (P,P) | MIN(MAX(D1,D2), DMAX) |
| * (P,P) | MIN(D1+D2, DMAX) |
| / or ** (any P) | DMAX |

Where DMAX = Maximum decimals without integer overflow.

### Formatting Results
Use `$PIC` tool to format numeric results with display masks.
