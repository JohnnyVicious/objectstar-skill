# Chapter 2: Composition of a Rule

## Components of a Rule

A rule is divided into four components:

1. **Rules Declaration**: Rule name, optional arguments, and local variables
2. **Conditions**: Logical expressions with Y/N quadrant
3. **Actions**: Executable statements with action sequence numbers
4. **Exception Handlers**: ON statements for error handling

## Visual Structure

```
RULE EDITOR ===>                                                    SCROLL: P
RULE_NAME(ARG1, ARG2);                              ← Declaration
LOCAL VAR1, VAR2;                                   ← Local Variables
---------------------------------------------------------------------------
CONDITION1;                                         | Y N N    ← Conditions
CONDITION2;                                         |   Y N       with Y/N
------------------------------------------------------------+-------------- Quadrant
ACTION1;                                            | 1       ← Actions with
ACTION2;                                            |   1        Sequence
ACTION3;                                            |     1      Numbers
---------------------------------------------------------------------------
ON EXCEPTION_NAME:                                  ← Exception Handlers
   HANDLER_STATEMENT;
```

## Rules Declaration

### Rule Name and Arguments

```
EMPLOYEES_RAISE(JOBTITLE, REGION);
```

- Rule name must be unique within a rules library
- Arguments are enclosed in parentheses `()`
- Multiple arguments separated by commas `,`
- Declaration ends with semicolon `;`

### Argument Behavior

| Property | Description |
|----------|-------------|
| Scope | The rule where the argument is declared |
| Data representation | Dynamic - conforms to semantic type and syntax of passed value |
| Passing method | By value - no rule can alter an argument's value |

## Local Variables

### Declaration Syntax

```
LOCAL RAISE, RATE;
```

- Begins with reserved word `LOCAL`
- Ends with semicolon `;`
- Multiple variables separated by commas `,`

### Local Variable Properties

| Property | Description |
|----------|-------------|
| Scope | The declaring rule and any descendant rules |
| Initial value | Empty string (character), zero (numeric), N (logical) |
| Maximum value/length | Determined by session attributes |

### Data Representation Rules

| Assignment Type | Behavior |
|-----------------|----------|
| From table field | Conforms to semantic type and syntax of source |
| Literal value | No semantic type assigned, only syntax |
| With concatenation (||) | String (S) semantic type assigned |

## Conditions Section

### What Comprises a Condition?

A condition is one of the following that evaluates to Y/N:
- An expression with a comparison operator
- An argument name or local name
- A functional rule and its argument values
- A table.field reference

Each condition ends with a semicolon `;`

### Y/N Quadrant

The Y/N quadrant regulates action processing:
- **Y** = Yes, execute actions in this column when condition is true
- **N** = No, condition must be false for this column
- Blank = Condition not evaluated for this column

### Example

```
JOBTITLE = 'SENIOR ANALYST';                        | Y N N
JOBTITLE = 'ANALYST';                               |   Y N
----------------------------------------------------+---------
RATE = 0.1;                                         | 1
RATE = 0.05;                                        |   1
RATE = 0.02;                                        |     1
```

**Reading this:**
- Column 1: JOBTITLE='SENIOR ANALYST' is Y → RATE=0.1 executes
- Column 2: First condition is N, second is Y → RATE=0.05 executes
- Column 3: Both conditions are N → RATE=0.02 executes

## Actions Section

### Action Statements

- Each action is an executable statement
- A rule must contain at least one action
- Each action ends with semicolon `;` (except FORALL/UNTIL which end with `:`)

### Action Sequence Numbers

- Determine which actions execute for each condition column
- Same action can execute for different conditions
- Actions spanning multiple lines: sequence number on first line only

### Restrictions

- No action sequence numbers within FORALL or UNTIL loops
- All actions within a loop execute when the loop executes

### Editing Behavior

- In rules without conditions: Rule Editor supplies consecutive numbers
- In rules with conditions: You must supply the numbers
- Alphanumeric characters converted to sequential numbers on save
- Deleting a sequence number prevents that action from executing

## Exception Handlers

### Structure

```
ON exception_name:
   statement1;
   statement2;
```

- Begins with `ON` keyword
- Exception name followed by colon `:`
- Optional statements executed when exception is trapped

### Behavior

- No action sequence numbers for exception handlers
- All statements in handler execute when exception is trapped

### Example

```
ON GETFAIL:
   CALL ENDMSG('POSITION IS INVALID');
```

This traps the GETFAIL exception and displays an error message.
