# Chapter 9: Transaction Processing

## What is a Transaction?

A transaction moves the database from one consistent state to another. This includes transactions that:
- Do not attempt to update the database
- Attempt updates but determine they are invalid
- Successfully update data

### Example: Adding an Employee
Business rule: Employee must have valid department number.
1. Reference department table to validate
2. If invalid: reject addition
3. If valid: process addition

Result: Database remains in consistent state.

---

## Starting Transactions

| Statement | Effect on Current Transaction | New Transaction |
|-----------|-------------------------------|-----------------|
| EXECUTE | Suspended (not ended) | Child transaction started |
| TRANSFERCALL | Ended | New transaction started |
| DISPLAY & TRANSFERCALL | Ended (screen context preserved) | New transaction started |
| SCHEDULE | Continues | Separate batch transaction |

---

## Transaction Flow

### Basic Transaction Flow
1. Synchronization point established (implicit)
2. Transaction actions execute
3. Final synchronization point (implicit commit or rollback)
4. Locks released

### Synchronization Points

**Implicit**: Created at transaction start and end

**Explicit**: Created with COMMIT or ROLLBACK statements
- COMMIT: Saves changes since last sync point
- ROLLBACK: Discards changes since last sync point

### Automatic Behavior
- Normal termination: All changes since last sync point committed
- Abnormal termination: All changes since last sync point discarded

### Actions Within Sync Points
- Data retrieved
- Locks taken on definitions and data (if update mode)
- Updates made
- Explicit sync points processed

---

## Nesting Transactions

When you EXECUTE a rule from within a transaction:
- Current transaction becomes **parent** (suspended)
- New **child** transaction starts
- Child completion returns control to parent

### Parent-Child Handling
Parent can catch child failures with EXECUTEFAIL exception.

### Transaction Levels
- First transaction: Level 1
- TRANMAXNUM session parameter: Maximum nesting depth
- $RULENAME tool: Find rule name at any level

---

## Transaction Flow Diagrams

### Starting New Transaction (TRANSFERCALL)
```
Transaction 1                    Transaction 2
┌─────────────────┐             ┌─────────────────┐
│ CALL RULE1;     │             │ CALL RULE3;     │
│ TRANSFERCALL    │────────────>│ CALL RULE4;     │
│   RULE2;        │             │                 │
└─────────────────┘             └─────────────────┘
```

### Nested Transactions (EXECUTE)
```
Transaction 1         Transaction 2         Transaction 3
┌────────────┐       ┌────────────┐       ┌────────────┐
│            │  1    │            │  3    │            │
│ EXECUTE    │──────>│ EXECUTE    │──────>│ CALL RULE6;│
│   RULE2;   │       │   RULE3;   │       │            │
│ CALL RULE7;│<──────│            │<──────│            │
│            │  4    │            │  2    │            │
└────────────┘       └────────────┘       └────────────┘
```

### Nested with TRANSFERCALL Inside
```
Trans 1           Trans 2           Trans 3           Trans 4
┌─────────┐      ┌─────────┐      ┌─────────┐      ┌─────────┐
│         │  1   │         │  3   │TRANSFER │  4   │         │
│ EXECUTE │─────>│ EXECUTE │─────>│ CALL    │─────>│         │
│ RULE2;  │      │ RULE3;  │      │ RULE4;  │      │ CALL    │
│         │<─────│         │<─────│         │      │ RULE5;  │
│ CALL    │  5   │         │  2   │         │      │         │
│ RULE8;  │      │         │      │         │      │         │
└─────────┘      └─────────┘      └─────────┘      └─────────┘
```

### Batch Transaction (SCHEDULE)
```
Transaction 1                    Transaction 2
┌─────────────────┐             ┌─────────────────┐
│ CALL RULE1;     │             │ CALL RULEA;     │
│ SCHEDULE RULE2; │ - - - - - ->│ CALL RULEB;     │
│ CALL RULE3;     │   (async)   │                 │
└─────────────────┘             └─────────────────┘
Transaction 1 continues immediately; Transaction 2 runs independently.
```

---

## Transaction Mode

### Setting Mode
```
TRANSFERCALL IN BROWSE FIND_DEPT;
TRANSFERCALL IN UPDATE SAVE_DEPT;
EXECUTE IN BROWSE QUERY_DATA;
```

### Mode Inheritance
Without explicit mode, transaction inherits from parent.
First transaction mode: Set by BROWSE/NOBROWSE session parameters.

### Mode Effects

| Mode | TDS Data Locks | Updates Allowed |
|------|----------------|-----------------|
| BROWSE | Definition only | No (ACCESSFAIL if attempted) |
| UPDATE | Definition + Data | Yes |

### Browse Mode Updates Allowed
Even in browse mode, you CAN update:
- TEM (temporary) tables
- SCR (screen) tables
- RPT (report) tables
- SES (session) tables
- EXP files

---

## Locks on Data

### Lock Types

| Lock Type | Definition Modifiable | Other Transactions Can |
|-----------|----------------------|------------------------|
| Shared | No | Read data |
| Exclusive | No | Nothing (blocked) |

### Locking by Operation

| Operation | Mode | Lock Type | Level |
|-----------|------|-----------|-------|
| GET/FORALL | Browse | Shared | Definition only |
| GET/FORALL (unique PK) | Update | Shared | Definition + Occurrence |
| GET/FORALL (non-unique) | Update | Shared | Definition + Table/Instance |
| GET with MINLOCK | Update | Shared | Table during GET, then Occurrence only |
| DELETE/INSERT/REPLACE | Update | Shared→Exclusive | Definition + Occurrence |
| View definition | Any | Shared | Definition |
| Update definition | Update | Exclusive | Definition |
| Build secondary index | Update | Shared | Definition + Table/Instance |
| Access CLC table (read) | Browse | Shared | Source definition |
| Access CLC table | Update | Shared | Source data + Instance |
| Access SUB table (MODE=D) | Update | Shared | Multiple definitions + Instance + Occurrence |
| Access SUB table (MODE=B) | Browse | Shared | Source + Table definitions |

### Exception Handling
Use LOCKFAIL exception when lock cannot be obtained.

---

## Data Access and Uncommitted Changes

### GET Statement
- **Unique primary key specified**: Retrieves uncommitted data
- **Non-unique selection**: Retrieves committed data only

### FORALL Statement
- Always operates on committed data only
- Must COMMIT before FORALL can see your changes

### Example: Commit Before FORALL
```
INSERT EMPLOYEES WHERE REGION = 'MIDWEST';
COMMIT;
FORALL EMPLOYEES WHERE REGION = 'MIDWEST':
   -- Now sees the inserted occurrence
END;
```

---

## Fail Safe Processing

Ensures updates to multiple databases in a single transaction are synchronized at all times.
