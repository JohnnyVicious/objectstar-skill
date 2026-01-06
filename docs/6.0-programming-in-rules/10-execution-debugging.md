# Chapters 16-18: Execution, Batch Processing, and Debugging

## Standard Execution Mode

### Invoking Execute Rule

**From workbench:**
```
EX execute rule ==> EMPLOYEES_RAISE
```

**From command line:**
```
COMMAND ==> EX EMPLOYEES_RAISE
```

**With arguments:**
```
COMMAND ==> EX EMPLOYEES_RAISE('ANALYST','MIDWEST')
```

**From Object Manager:** Use X line command

### Supplying Arguments

If not provided in command, argument screen appears:
```
---------------------------- RULE EXECUTION ----------------------------

ENTER ARGUMENTS FOR RULE EMPLOYEES_RAISE
JOBTITLE ===>
REGION ===>
```

**Note**: Maximum arguments via screen depends on terminal mode (12 for Model 5).

### Library Search Order
Default: Local → Installation → System

Can be modified via:
- User profile
- Session options
- Menu definition (DEFINE_MENU, DEFINE_OBJLIST)

### Execution Results

| Result | Display |
|--------|---------|
| ENDMSG | Message text at bottom |
| Success | `11:02:18 OK` |
| Failure | Error message at bottom |
| Message Log | Press PF2 for details |

---

## Message Logs

Two logs maintained:
- **User Log**: Messages from rules (MSGLOG, $PRINTLINE)
- **System Log**: Errors and debugging information

### Accessing Logs
- PF2 from workbench
- System log shown first
- PF2 again for user log (if system log empty, user log shown first)

### Log Commands

| Command | Action |
|---------|--------|
| PRINT | Print current log |
| FIND string | Find text (case insensitive) |
| F 'multi word' | Find phrase |
| F string C | Find with case sensitivity |

### Log PF Keys

| Key | Action |
|-----|--------|
| PF1 | Help |
| PF2 | Toggle to other log |
| PF3/PF12 | Exit |
| PF5 | Repeat search |
| PF7/PF8 | Scroll up/down |
| PF10/PF11 | Scroll left/right |
| PF13 | Print log |

---

## User Log

Contains messages from:
- MSGLOG tool
- $PRINTLINE tool
- Parent and descendant transactions

### Output Behavior

**EXECUTE**: Output from all parent transactions maintained.
Messages appear sequentially, then last child at each level.

**TRANSFERCALL**: Sibling transaction messages maintained.

### Example User Log
```
------------------------ INFORMATIONAL MESSAGE LOG ------------------------
COMMAND ===>                                                   SCROLL ===> P
HRODEK NOW EARNS 745.50
CANNON NOW EARNS 735.00
BOIVIN NOW EARNS 745.00

PFKEYS: 2=NEXT LOG 3=EXIT 5=REPEAT 12=EXIT 13=PRINT
```

---

## System Log

Contains error information when rule fails and exception not handled.

### Information Provided
1. Location of error
2. Reason rule stopped
3. Traceback of rules called
4. Active local variables
5. Buffers for active tables
6. Last 16 rules invoked

### Example System Log
```
------------------------ INFORMATIONAL MESSAGE LOG ------------------------
COMMAND ===>                                                   SCROLL ===> P
Error detected in rule "NEWMANAGER" at action 3 FORALL statement 2
Access error on TABLE "EMPLOYEES"
REPLACE EMPLOYEES
Invalid number of parameters supplied for table "EMPLOYEES"
Traceback of rules called at time of error
NEWMANAGER(96,10)
End of traceback
-------------------------------------------------------------------------------
No local variables
Dump of active tables
Table EMPLOYEES
EMPNO = 44385
LNAME = 'SOUZA'
POSITION = 'SALES'
...
```

### Reading Error Location
```
Error detected in rule "NEWMANAGER" at action 3 FORALL statement 2
```
- Rule: NEWMANAGER
- Action: 3rd statement
- FORALL statement: 2nd line in loop

**Note**: Action sequence numbers not referenced; statement count from top.

### Event Logging
For trigger, validation, or derivation rule errors:
```
=========== Trigger for table "DEPARTMENTS" ==== level 1 ==========
Trigger rule "INSERTDEPTNO" for table "DEPARTMENTS" failed at action 3
...
```

---

## Batch Processing

### Options

| Method | Description |
|--------|-------------|
| SCHEDULE (immediate) | Submit JCL/batch/script for immediate processing |
| SCHEDULE TO 'queue' | Send to batch queue (z/OS only) |

### @SCHEDULEMODEL Table
Parameterized by OPERATING_SYSTEM and MODELNAME.

Default search:
1. @SCHEDULEMODEL(os, rule_name)
2. @SCHEDULEMODEL(os, *DEFAULT*)

### Default Instances
| Instance | Platform |
|----------|----------|
| @SCHEDULEMODEL(MVS,*DEFAULT*) | z/OS |
| @SCHEDULEMODEL(NT,*DEFAULT*) | Windows |
| @SCHEDULEMODEL(UNIX,*DEFAULT*) | Solaris |

### Variable Substitution
Variables in braces: `{LIBRARY}`, `{RULE}`, `{SEARCH}`

Delimiters set by: VARLDELIMITER, VARRDELIMITER parameters

### Example JCL
```
//HURON EXEC PGM=S6BBATCH,REGION=4096K,
//PARM=('TDS={TDS},RULE=INCOME_TAX')
//HRNIN DD *
{TEST},
SEA={SEARCH},
C={CHARSET},
L=REPORT
/*
```

### Batch Queue (z/OS)
```
SCHEDULE TO 'MONTH_END' MONTHLY_REPORT(MONTH);
```

Use BATCH tool to:
- View available queues
- Check job status
- Remove jobs from queue
- Create JCL

### Editing @SCHEDULEMODEL

**Important Notes:**
- Use Model 5 terminal if possible
- Data is case sensitive
- Card field limited to 72 characters
- Continuation character: `\`
- Windows commands: max 2046 characters
- Solaris: add `&` at end of last line

---

## Rule Debugger

### Processing in Debug Mode

Allows you to:
- View execution state
- Modify field values, parameters, local variables
- Set break events
- Step through execution

### Invoking Debugger

**From workbench:**
```
DB debug rule ==> rulename
```

**From command line:**
```
COMMAND ==> DB rulename
```

**From Execute Rule:**
```
EX execute rule ==> Debug(rulename)
```

**From Object Manager:** Use G line command

### Debugger Screen Layout

```
COMMAND ===>

-------------------------Debug Information-----------------------------
Starting TRANSFERCALL to rule EMPLOYEES_RAISE
------------------------Breakevent Control-----------------------------
Break Event           Object name



PFKEYS: 1=HELP 3=SAVE BREAKS 6=DBGCOM 12=EXIT 14=EXPAND 15=GO 22=REMOVE
```

### Debugger PF Keys

| Key | Command | Action |
|-----|---------|--------|
| PF2 | – | Message logs |
| PF3 | – | Save breaks, exit one level |
| PF4 | – | Save breaks, remain |
| PF5/PF15 | GO | Continue executing |
| PF6 | DBGCOM | Command Editor screen |
| PF9 | – | Re-display last command |
| PF12 | EXIT | Exit debugger |
| PF13 | – | Print screen |
| PF14 | EXPAND | Expand current rule |
| PF17 | DUMP CALLSTACK | Show execution stack |
| PF18 | DUMP LASTCALLED | Show last 16 rules |
| PF19 | DUMP LOCALS | Show local variables |
| PF20 | DUMP TABLES | Show active tables |
| PF22 | OFF | Delete break event at cursor |

### Debugger Commands

| Command | Action |
|---------|--------|
| AT event qualifier | Set break event |
| RUN | Ignore break for current transaction |
| STOPAPPLICATION | Exit application to starting point |
| STOPDEBUGGER | Stop debugging, let app finish |

---

## Break Events

### Specifying Break Events

**Command method:**
```
AT event qualifier
```

**Direct entry:** Type in Breakevent Control section

### Break Event Reference

| Event | Description | Qualifiers |
|-------|-------------|------------|
| ACCESS | Any table access | *, table_name |
| CALL | CALL or function reference | *, rule_name |
| COMMIT | COMMIT statement | (none) |
| DELETE | DELETE statement | *, table_name |
| DISPLAY | DISPLAY statement | *, screen_name |
| DISPXCALL | DISPLAY & TRANSFERCALL | *, screen_name |
| EXECUTE | EXECUTE statement | *, rule_name |
| FORALL | FORALL statement | *, table_name |
| GET | GET statement | *, table_name |
| INSERT | INSERT statement | *, table_name |
| ON | Exception handling | *, exception, exception table |
| REFFIELD | Reference to table field | *, table.*, table.field |
| REFLOCAL | Reference to local variable | *, variable_name |
| REPLACE | REPLACE statement | *, table_name |
| RETURN | Return from rule | *, rule_name |
| ROLLBACK | ROLLBACK statement | (none) |
| SCHEDULE | SCHEDULE statement | *, instance_name |
| SETFIELD | Assign to table field | *, table.*, table.field |
| SETLOCAL | Assign to local variable | *, variable_name |
| SIGNAL | Raise exception | *, exception, exception table |
| TRANSACTIONEND | End of transaction | (none) |
| TRANSFERCALL | TRANSFERCALL statement | *, rule_name |
| UNRECERR | Unrecoverable error | (none) |

`*` = All events of this type

**Note**: Break events not processed for trigger, validation, or derivation rules.

**Warning**: Do not use COMMIT or ROLLBACK breaks with external database open for update.

---

## Examining Execution State

### Commands

| Command | Syntax | Action |
|---------|--------|--------|
| LIST | `LIST local_var` | Display variable value |
| LIST | `LIST table.field` | Display field value |
| LIST | `LIST table.*` | Display all fields |
| SET | `SET local=value` | Change local variable |
| SET | `SET table.field=value` | Change field value |
| SET | `SET table.*=value` | Change all fields |
| DUMP CALLSTACK | – | Show execution stack |
| DUMP LASTCALLED | – | Show last 16 rules |
| DUMP LOCALS | – | Show all locals with values |
| DUMP TABLES | – | Show all tables with values |

### Command Series for Break Events

Create automated command sequences:
1. Position cursor on break event
2. Press PF6 (Debugger Command Editor)
3. Enter commands using TED editor
4. PF3 to save

### Command Editor Line Commands

| Command | Action |
|---------|--------|
| I | Insert line after |
| D | Delete line |
| M | Move line (use A/B) |
| C | Copy line (use A/B) |
| A | After destination |
| B | Before destination |

### Command Editor PF Keys

| Key | Action |
|-----|--------|
| PF1 | Help |
| PF3 | Save and exit |
| PF4 | Validate commands |
| PF12 | Cancel and exit |
| PF13 | Print commands |
| PF22 | Delete all commands |
