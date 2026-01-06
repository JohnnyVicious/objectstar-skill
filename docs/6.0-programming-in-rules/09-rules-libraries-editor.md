# Chapters 13-15: Rules Libraries and Rule Editor

## Rules Libraries Organization

### Three-Tier Library System

| Library | Description | Typical Name |
|---------|-------------|--------------|
| Local | Rules you created, copied, or modified | User-defined |
| Installation | Rules available to all users at site | SITE |
| System | Rules shipped with OSB | COMMON |

### Default Search Order
1. Local library
2. Installation library
3. System library

Can be modified via user profile, session options, or menu definition.

---

## Managing Libraries

### View Library Listing
From workbench: `DL define library ==>`

### Library List Commands
| Command | Action |
|---------|--------|
| D | Delete library (with confirmation) |
| S | Display all rules in library |

### Define New Library
1. Position cursor on `DL define library`
2. Type new library name, press Enter
3. Enter description information
4. Press PF3 to save

### Library Description Fields
| Field | Description |
|-------|-------------|
| KEYWORDS | Brief words for Keyword Search (comma/space separated) |
| SUMMARY | One-line description |
| DESCRIPTION | Detailed information (SCRIPT format) |

---

## Changing Libraries

### Default Login Library
Set via:
- User profile
- Command line arguments to osBatch/S6BBATCH
- Session options

### Access Different Library
Overtype library name in Library field of workbench.
Change persists until modified or session ends.

### Copy Rule to Different Library
**Method 1**: `CD copy defn` menu option (COPYDEFN tool)
**Method 2**: C line command from Object Manager
**Method 3**: CALL or EXECUTE COPY_DEFN tool

---

## Rule Editor

### Invoking the Editor

**From workbench:**
```
ER edit rule ==> rulename
```

**From execute option:**
```
EX execute rule ==> EDITRULE(rulename)
```

**From command line:**
```
COMMAND==> ER rulename
```

Without rule name: Object Manager screen appears.

### Rule Name Requirements
- Up to 16 characters
- Start with letter (A-Z) or special character ($ #)
- Continue with letters, specials, digits (0-9), or underscore
- No spaces
- Unique within library

**Note**: `@` is used by OSB-supplied objects.

---

## Rule Editor Screen Layout

```
RULE EDITOR ===>                                            SCROLL: P
RULE_NAME(ARGS);                                 ← Declaration
LOCAL VARS;                                      ← Local Variables
------------------------------------------------------
                                                | Y/N    ← Conditions
Conditions                                      | Quadrant
------------------------------------+-----------------
                                    |          ← Actions with
Actions                             | Numbers     Sequence Numbers
------------------------------------------------------
                                                ← Exception
Exception Handlers                                 Handlers

PFKEYS: 1=HELP 3=END 12=CANCEL 13=PRINT 14=EXPAND 2=DOCUMENT 22=DELETE
```

### Scrolling
| Key | Action |
|-----|--------|
| PF7 | Scroll up |
| PF8 | Scroll down |

### Scroll Values
| Value | Meaning |
|-------|---------|
| H | Half page |
| P | Full page (default) |
| M | Maximum |
| C | From cursor line |
| nnn | Specific number of lines |

---

## Line Commands

| Command | PF Key | Action |
|---------|--------|--------|
| I | PF4 | Insert line below |
| R | – | Replicate line below |
| D | PF16 | Delete line |
| S | – | Split line at cursor |
| M | – | Move line (use A/B for destination) |
| A | – | Destination: after this line |
| B | – | Destination: before this line |
| C | – | Copy line (use A/B for destination) |

### Copy Lines
```
c JOBTITLE = 'SENIOR ANALYST';                  | Y N N
c JOBTITLE = 'ANALYST';                         |   Y N
b --------------------------------------------+---------
```
Lines with 'c' copied before line with 'b'.

### Move Lines
```
b RATE = 0.1;                                   | 1
m RATE = 0.05;                                  |   1
m RATE = 0.02;                                  |     1
```
Lines with 'm' moved before line with 'b'.

### Split Line
1. Type S in line command field
2. Position cursor where to split
3. Press Enter

Token-sensitive: splits at token boundary.

### Combining Commands
Multiple line commands processed top to bottom.

---

## Primary Commands

### File Operations
| Command | PF Key | Action |
|---------|--------|--------|
| SAVE | – | Save rule, remain in editor |
| END | PF3 | Save rule, exit editor |
| CANCEL | PF12 | Discard changes, exit |
| EDIT | – | Save current, edit different rule |
| XEDIT | – | Discard current, edit different rule |
| DELETE | PF22 | Delete rule (requires CONFIRM) |

### Search/Replace
| Command | Action |
|---------|--------|
| `FIND token` | Find token (PF5 for next) |
| `CHANGE old new` | Change first occurrence |
| `CHANGE old new ALL` | Change all occurrences |

After FIND: PF6 changes current to null.
Search starts at cursor, wraps to beginning.

### Content Operations
| Command | PF Key | Action |
|---------|--------|--------|
| `COPY rulename` | – | Replace content with copy of rule |
| `APPEND rulename` | – | Append rule (actions & handlers) |
| DOCUMENT | PF2 | Edit rule documentation |
| PRINT | PF13 | Print rule with documentation |

### Case Control
| Command | Action |
|---------|--------|
| LOWER | Allow lowercase in strings |
| UPPER | Revert to uppercase |

### Other
| Command | PF Key | Action |
|---------|--------|--------|
| HELP | PF1 | Display online help |
| – | PF9 | Re-display last primary command |

---

## Expanding Token Information

### Expand Types
| Token Type | Information Displayed |
|------------|----------------------|
| Rule | Complete contents |
| Table | Name, parameters, fields with attributes |
| Field | Attributes, global usage, reference values |
| Screen | Name, screen tables (expandable to fields) |
| Report | Name, report tables (expandable to fields) |
| Routine | Name, arguments, keywords, summary |

### Using Expand

**PF14 Method:**
Position cursor on token name, press PF14.

**EXPAND Command:**
```
EXPAND token_name [token_type]
```
Token types: RULE, REPORT/RPT, SCREEN/SCR, TABLE/TBL

Without type: searches reports, screens, tables, rules, routines.

### Expanded Display
Screen splits horizontally:
- Upper: Rule being edited
- Lower: Browsable token information

Scroll in either portion with PF7/PF8.

### Nesting Expansions
From within expanded display:
- Position cursor on object
- Press PF14 to expand further

Allowed for: rules, screens, reports, screen tables, report tables.
Not allowed from: routines, tables, fields.

### Closing Expanded Display
| Action | Effect |
|--------|--------|
| PF15 | Close current, return to previous |
| PF15 outside display | Exit expand facility |
| CLOSE command | Close all open displays |

---

## Object Manager Screen

When invoking Rule Editor without rule name:

```
List of rules to edit                           LIBRARY: DOCEXMPL
Command ==>                                              Scroll P

NAME             DATE       TIME UNIT     DESCRIPTION*
---------------- ---------- ---- -------- ---------------------------
_ ABC            2000-03-20 1154 USR      Sample rule
_ EMPLOYEE_COUNT 1999-11-02 1146 USR40    Count employees in dept
...

C-Copy D-Delete G-Debug P-Print S-Select X-eXecute

PFKEYS: 12=EXIT 13=PRINT 3=END 5=FIND NEXT 9=RECALL
```

### Object Manager Commands
| Command | Action |
|---------|--------|
| C | Copy rule to another library |
| D | Delete rule (with confirmation) |
| G | Invoke Rule Debugger |
| P | Print rule |
| S | Select rule for editing |
| X | Execute rule |

### Primary Commands
- SELECT: Create shorter list with criteria
- APPLY: Apply selection
- ORDERED: Order list
- FIND: Search for rule

---

## Syntax Checking

The syntax checker:
- Runs when saving (PF3, END, SAVE)
- Checks for accuracy
- Displays error messages
- Positions cursor at first error
- Rule cannot be saved until correct

**Note**: Syntactically correct rules may still have:
- Misspelled names
- Incorrect logic
- Runtime errors

These are only detected during execution.
