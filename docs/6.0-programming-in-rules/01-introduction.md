# Chapter 1: Introduction to TIBCO Object Service Broker Rules

## What is the Rules Language?

The rules language is the programming language used within TIBCO Object Service Broker to create and modify applications. It consists of:

- **Database access commands**: GET, DELETE, INSERT, REPLACE statements
- **High-level constructs**: Loops (FORALL, UNTIL), exception handling (SIGNAL, ON)
- **Pseudo-conversational constructs**: DISPLAY & TRANSFERCALL
- **Procedural statements**: CALL, EXECUTE

## What is a Rule?

A rule is a series of one or more rules language statements that contains a procedure or returns a value.

### Minimum Requirements
A rule must have:
- A unique name within a rules library
- One or more action statements

### Optional Components
A rule can also contain:
- Variables (LOCAL declarations)
- Conditional processing (Y/N quadrant)
- Calls to other rules
- Transference to another transaction
- Error handling (exception handlers)

## How are Rules Defined?

Rules are defined using the **Rule Editor**. This tool displays either:
- An empty rule template for creating a new rule
- The source code for an existing rule

## How Do You Execute a Rule?

Once rules pass syntax checking, they can be executed in several ways:

| Mode | Method | Description |
|------|--------|-------------|
| Standard | Rule Executor from workbench | Normal interactive execution |
| Batch | Asynchronous processing | Background/scheduled execution |
| Debug | Rule Debugger | Step-through debugging with breakpoints |

## Sample Rule: EMPLOYEES_RAISE

```
EMPLOYEES_RAISE(JOBTITLE, REGION);
LOCAL RAISE, RATE;
---------------------------------------------------------------------------
JOBTITLE = 'SENIOR ANALYST';                              | Y N N
JOBTITLE = 'ANALYST';                                     |   Y N
------------------------------------------------------------+---------------
RATE = 0.1;                                               | 1
RATE = 0.05;                                              |   1
RATE = 0.02;                                              |     1
GET EMPLOYEES(REGION) WHERE POSITION = JOBTITLE;          | 2
FORALL EMPLOYEES(REGION) WHERE POSITION = JOBTITLE:       | 2 2 3
   RAISE = EMPLOYEES.SALARY * RATE;                       |
   EMPLOYEES.SALARY = EMPLOYEES.SALARY + RAISE;           |
   CALL REPLACE_SALARY(REGION);                           |
   CALL MSGLOG(EMPLOYEES.LNAME || ' NOW EARNS ' ||        |
      EMPLOYEES.SALARY);                                  |
END;                                                      |
---------------------------------------------------------------------------
ON GETFAIL:
   CALL ENDMSG('POSITION IS INVALID');
```

### How This Rule Works
1. Takes JOBTITLE and REGION as arguments
2. Evaluates conditions to determine raise rate (10%, 5%, or 2%)
3. Gets employees matching the job title and region
4. Loops through employees, calculates raise, updates salary
5. If no employees found (GETFAIL), displays error message

## Sample Rule: REPLACE_SALARY

```
REPLACE_SALARY(REGION);

---------------------------------------------------------------------------
-------------------------------------------------------------+-------------
REPLACE EMPLOYEES(REGION);                                   | 1
---------------------------------------------------------------------------
ON COMMITLIMIT:
   COMMIT;
```

This helper rule handles the actual database update and manages commit limits.

## Available Tools for Rules Development

| Tool | Purpose |
|------|---------|
| Rule Editor | Primary tool for writing/editing rules |
| Rule Executor | Run rules in standard mode |
| Rule Debugger | Debug rules with breakpoints |
| Rule Printer (RULEPRINTER) | Produce tree of rules and associated objects |
| Search Utility (CROSSREFSEARCH) | Search for rules, exceptions, handlers |
| Copy Definition (COPYDEFN, COPY_DEFN) | Copy rule definitions between libraries |
| Compare Definition (DIFFDEFN, DIFF_DEFN) | Compare rule definitions |
| Library Definer | Define and manage rules libraries |
