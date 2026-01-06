# Chapter 8: Using Expressions and Operators

## What is an Expression?

An expression is any combination of values and operators that evaluates to a single value.

### Expression Values Can Be
- Local variable
- Field of a table
- Rule argument name
- Function call
- Literal
- Indirect reference

### Expression Examples
| Expression | Result |
|------------|--------|
| `3+2*3` | 9 |
| `10*(1+4)**2` | 250 |
| `SUM + (TABLEREF).(FIELDREF)` | Sum + indirect field value |

---

## Data Syntax Types

Each data element has a **syntax** that describes how data is stored.

| Syntax | Name | Valid Lengths | Notes |
|--------|------|---------------|-------|
| B | Binary | 2-12 bytes | Signed integers. Length 4 supports ±2 billion |
| C | Fixed-length character | 1-127 (key), 1-254 (display) | Uppercase. Trailing blanks not significant |
| F | Floating point | 4, 8, 16 bytes | Not for keys/parameters. Approx range: 5.4×10⁻⁷⁹ to 7.2×10⁷⁵ |
| P | Packed decimal | 1-16 bytes (1-31 digits) | Supports display masks |
| RD | Raw data | 5+ bytes | 4-byte length + data. Not for keys |
| UN | Unicode | 2+ bytes (even) | UTF16 characters. Not for keys |
| V | Variable-length character | 1-127 (key), 1-254 (display) | Trailing blanks significant |
| W | Double-byte character | 4+ bytes | z/OS only. Not for parameters |

### Key Field Constraints
- Composite primary key: max 127 bytes total (up to 16 fields)
- All data parameters: max 240 bytes total

---

## Semantic Data Types

The **semantic data type** describes how an expression element can be used.

| Type | Code | Description | Valid Syntax |
|------|------|-------------|--------------|
| Typeless | (none) | Literals in rules, fields without type | B,C,F,P,RD,UN,V,W |
| Count | C | Integer numbers for counting | B,C,P,UN,V |
| Date | D | Dates (must include year). Range: 0000/01/01-9999/12/31 | B |
| Identifier | I | Unique identifier | B,C,P,UN,V |
| Logical | L | Yes/No conditions | C |
| Quantity | Q | Real numbers for measurement | B,C,F,P,UN,V |
| String | S | Character string data | C,RD,UN,V,W |

### Permitted Operations by Semantic Type

| Type | Relational | Concat | Assignment | Arithmetic | Logical |
|------|------------|--------|------------|------------|---------|
| Typeless | ✓ | ✓ | ✓ | ✓ | ✓ |
| Count | ✓ | ✓ | ✓ | ✓ | |
| Date | ✓ | ✓ | ✓ | +,- only | |
| Identifier | ✓ | ✓ | ✓ | | |
| Logical | ✓ | ✓ | ✓ | | ✓ |
| Quantity | ✓ | ✓ | ✓ | ✓ | |
| String | ✓ | ✓ | ✓ | | |

---

## Arithmetic Operators

| Operator | Operation | Precedence |
|----------|-----------|------------|
| `**` | Exponentiation | Highest |
| `*` | Multiplication | |
| `/` | Division | |
| `+` | Unary plus | |
| `-` | Unary minus | |
| `+` | Addition | Lowest |
| `-` | Subtraction | Lowest |

### Operand Types
- Count, quantity, date, and typeless allowed
- Unary +/- only for count, quantity, typeless

### Examples
```
CARS.PRICES = (PRICES.BASE + PRICES.SHIPPING) * TAXES.RETAIL;
AMOUNT = PRINCIPAL * (1 + INTEREST) ** YEARS;
```

**Note**: For consecutive exponentiation, use parentheses:
```
(A ** B) ** C    -- valid
A ** (B ** C)    -- valid
A ** B ** C      -- invalid
```

---

## Concatenation Operator

The `||` operator concatenates two values.

### Syntax Rules
| Operand Types | Result Syntax |
|---------------|---------------|
| UN + W | Forbidden |
| Either RD | RD |
| Either UN | UN |
| Either W | W |
| Otherwise | V |

Result semantic type is always **string**.

---

## Relational Operators

| Operator | Description |
|----------|-------------|
| `=` | Equality |
| `¬=` | Not equal (also `^=` on Open Systems) |
| `<` | Less than |
| `<=` | Less than or equal |
| `>` | Greater than |
| `>=` | Greater than or equal |
| `LIKE` | Pattern matching (WHERE clause only) |

### Equality Operators (= and ¬=)
Can be used between:

|  | Count | Date | ID | Logical | Quantity | String | Typeless |
|--|-------|------|-----|---------|----------|--------|----------|
| Count | ✓ | | ✓ | | | | ✓ |
| Date | | ✓ | | | | | ✓ |
| Identifier | ✓ | | ✓ | | | ✓ | ✓ |
| Logical | | | | ✓ | | | ✓ |
| Quantity | | | | | ✓ | | ✓ |
| String | | | ✓ | | | ✓ | ✓ |
| Typeless | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

### Ordering Operators (<, <=, >, >=)
Cannot be used with **Logical** type. Otherwise same as equality.

### LIKE Operator
- Pattern matching with `?` (one char) and `*` (zero or more)
- Case insensitive for syntax C
- NOT syntax: `WHERE NOT(fieldname LIKE 'value')`

### Comparison Notes
- Trailing blanks significant for V syntax, not for C syntax
- Different syntax operands converted to common syntax
- Result is always logical (Y or N)

---

## Logical Operators

| Word | Symbol | Description |
|------|--------|-------------|
| AND | `&` | True if both operands true (WHERE clause only) |
| OR | `\|` | True if either operand true (WHERE clause only) |
| NOT | `¬` | True if operand false (conditions and WHERE) |

**Precedence**: OR takes precedence over AND

**Note**: On Open Systems, NOT may display as `^`

---

## Assignment Operator

The `=` operator assigns values.

### Valid Assignments

|  | Count | Date | ID | Logical | Quantity | String | Typeless |
|--|-------|------|-----|---------|----------|--------|----------|
| Count | ✓ | | ✓ | | | ✓ | ✓ |
| Date | | ✓ | | | | ✓ | ✓ |
| Identifier | ✓ | | ✓ | | | ✓ | ✓ |
| Logical | | | | ✓ | | | ✓ |
| Quantity | | | | | ✓ | ✓ | ✓ |
| String | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Typeless | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

Result semantic type = left operand's type (unless local variable, then right operand's type).

### Simple Assignment
```
CARS.PRICE = (PRICES.BASE + PRICES.SHIPPING) * TAXES.RETAIL;
AMOUNT = PRINCIPAL * (1 + INTEREST) ** YEARS;
```

### Assignment-by-Name
Assigns all identically-named fields:
```
INPUTORDERS.* = ORDERS.*;
```

### Initialize All Fields to Null
```
ORDERS.* = NULL;
```

---

## Indirect Referencing

Allows generic rules by referencing tables, fields, screens, reports, or rules indirectly.

### Where to Use
- Conditions or actions
- NOT in ON statement itself (but OK in handler body)

### Value Sources
Values must be semantic type Identifier or typeless, from:
- Rule argument
- Table.field reference

**Cannot use**: Local variable or function return value

### Using Argument for Indirect Reference

```
COUNT(TABLEREF, FIELDREF);
LOCAL SUM;
---------------------------------------------------------------------------
------------------------------------------------------------+--------------
SUM = 0;                                                    | 1
FORALL TABLEREF:                                            | 2
   SUM = SUM + (TABLEREF).(FIELDREF);                       |
END;                                                        |
CALL MSGLOG('THE SUM OF ' || TABLEREF || '.' ||             | 3
   FIELDREF || ' IS ' || SUM);                              |
---------------------------------------------------------------------------
```

Call as: `CALL COUNT('EMPLOYEE_VIEW', 'SALARY');`

### When Parentheses Required
Use parentheses around indirect reference when:
- Data element referenced in expression
- Used on left side of assignment
- Used as field name in WHERE or ORDERED clause

### Using Table.Field for Indirect Reference

```
PROCESS_FCNKEY(SCREEN);

---------------------------------------------------------------------------
------------------------------------------------------------+--------------
GET FCNKEYS(SCREEN) WHERE PF_KEY = ENTERKEY(SCREEN);        | 1
CALL FCNKEYS.ROUTINE;                                       | 2
---------------------------------------------------------------------------
ON GETFAIL FCNKEYS:
   CALL SCREENMSG(SCREEN, MESSAGE('GENERAL', 3, ENTERKEY(SCREEN)));
```

The field `FCNKEYS.ROUTINE` contains the rule name to call.

### Parameterized Table with Indirect Reference

```
COUNT1(TABLEREF, FIELDREF, PARM1, PARM2);
LOCAL SUM;
---------------------------------------------------------------------------
PARM1 = NULL;                                               | Y N N
PARM2 = NULL;                                               |   Y N
------------------------------------------------------------+--------------
FORALL TABLEREF:                                            | 1
   SUM = SUM + (TABLEREF).(FIELDREF);                       |
END;                                                        |
FORALL TABLEREF(PARM1):                                     |   1
   SUM = SUM + (TABLEREF).(FIELDREF);                       |
END;                                                        |
FORALL TABLEREF(PARM1, PARM2):                              |     1
   SUM = SUM + (TABLEREF).(FIELDREF);                       |
END;                                                        |
CALL MSGLOG('THE SUM OF ' || TABLEREF || '.' ||             | 2 2 2
   FIELDREF || ' IS ' || SUM);                              |
---------------------------------------------------------------------------
```

---

## The NULL Keyword

`NULL` represents a null value. Can be used in:
- Rule conditions: `expression = NULL`
- Assignment: `field = NULL`
- Passing arguments: `WHERE arg = NULL` or `(NULL, value)`
- Function return: `RETURN(NULL)`
- Initialize fields: `table.* = NULL`

**Cannot** be used as operand in expression (e.g., `NULL + 1` is invalid).
