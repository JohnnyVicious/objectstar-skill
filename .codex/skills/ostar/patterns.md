# Objectstar Code Patterns

## Rule Patterns

### Basic CRUD Rule
```
GET_EMPLOYEE(EMP_ID);
LOCAL FOUND;
---------------------------------------------------------------------------
GET EMPLOYEES WHERE EMP# = EMP_ID;      | Y N
------------------------------------------------------------+---------------
FOUND = 'Y';                            | 1
FOUND = 'N';                            |   1
---------------------------------------------------------------------------
ON GETFAIL:
    FOUND = 'N';
```

### Conditional Processing with Quadrant
```
CALC_DISCOUNT(CUST_TYPE, ORDER_AMT);
LOCAL DISCOUNT;
---------------------------------------------------------------------------
CUST_TYPE = 'PREMIUM';                  | Y N N
ORDER_AMT > 1000;                       |   Y N
------------------------------------------------------------+---------------
DISCOUNT = 0.20;                        | 1
DISCOUNT = 0.10;                        |   1
DISCOUNT = 0.05;                        |     1
```

### Iteration with FORALL
```
PROCESS_DEPT_EMPLOYEES(DEPT_CODE);
LOCAL TOTAL_SAL, EMP_COUNT;
---------------------------------------------------------------------------
TOTAL_SAL = 0;
EMP_COUNT = 0;
FORALL EMPLOYEES WHERE DEPT = DEPT_CODE
    ORDERED ASCENDING LNAME
    UNTIL GETFAIL:
    TOTAL_SAL = TOTAL_SAL + EMPLOYEES.SALARY;
    EMP_COUNT = EMP_COUNT + 1;
    CALL PROCESS_SINGLE_EMP(EMPLOYEES.EMP#);
END;
RETURN(TOTAL_SAL / EMP_COUNT);
```

### Screen Display Loop
```
ORDER_ENTRY_SCREEN;
LOCAL DONE;
---------------------------------------------------------------------------
DONE = 'N';
UNTIL EXIT_DISPLAY DISPLAY ORDER_SCREEN:
    CALL VALIDATE_ORDER;
    CALL PROCESS_ORDER;
    CALL UPDATE_INVENTORY;
END;
---------------------------------------------------------------------------
ON EXIT_DISPLAY:
    DONE = 'Y';
ON VALIDATEFAIL:
    CALL SCREENMSG(ORDER_SCREEN, 'Invalid order data');
```

### Master-Detail Processing
```
PROCESS_ORDER_DETAILS(ORDER_ID);
LOCAL LINE_TOTAL;
---------------------------------------------------------------------------
GET ORDERS WHERE ORDER# = ORDER_ID;
LINE_TOTAL = 0;
FORALL ORDER_LINES(ORDER_ID) UNTIL GETFAIL:
    GET PRODUCTS WHERE PROD# = ORDER_LINES.PROD#;
    LINE_TOTAL = LINE_TOTAL + (ORDER_LINES.QTY * PRODUCTS.PRICE);
    CALL CALC_LINE_TAX(ORDER_LINES.LINE#);
END;
ORDERS.TOTAL = LINE_TOTAL;
REPLACE ORDERS;
---------------------------------------------------------------------------
ON GETFAIL ORDERS:
    SIGNAL ORDER_NOT_FOUND;
ON GETFAIL PRODUCTS:
    CALL MSGLOG('Unknown product: ' || ORDER_LINES.PROD#);
```

### Transaction with Rollback
```
TRANSFER_FUNDS(FROM_ACCT, TO_ACCT, AMOUNT);
LOCAL SUCCESS;
---------------------------------------------------------------------------
SUCCESS = 'Y';
GET ACCOUNTS WHERE ACCT# = FROM_ACCT WITH MINLOCK;
ACCOUNTS.BALANCE = ACCOUNTS.BALANCE - AMOUNT;
REPLACE ACCOUNTS;
GET ACCOUNTS WHERE ACCT# = TO_ACCT WITH MINLOCK;
ACCOUNTS.BALANCE = ACCOUNTS.BALANCE + AMOUNT;
REPLACE ACCOUNTS;
COMMIT;
---------------------------------------------------------------------------
ON GETFAIL:
    SUCCESS = 'N';
    ROLLBACK;
ON REPLACEFAIL:
    SUCCESS = 'N';
    ROLLBACK;
    CALL MSGLOG('Transfer failed - rolled back');
```

### Parameterized Table Access
```
REGIONAL_SALES_REPORT(REGION_CODE);
LOCAL REGION_TOTAL;
---------------------------------------------------------------------------
REGION_TOTAL = 0;
FORALL SALES(REGION_CODE) WHERE YEAR = $YEAR($SYSTEMDATE)
    ORDERED DESCENDING AMOUNT
    UNTIL GETFAIL:
    REGION_TOTAL = REGION_TOTAL + SALES.AMOUNT;
    CALL PRINT_SALE_LINE(SALES.SALE#);
END;
CALL PRINT_REGION_TOTAL(REGION_CODE, REGION_TOTAL);
```

### Lock Retry Pattern
```
UPDATE_WITH_RETRY(RECORD_ID);
LOCAL DELAY, MAX_TRIES, ATTEMPT;
---------------------------------------------------------------------------
DELAY = 1;
MAX_TRIES = 5;
ATTEMPT = 0;
UNTIL DONE:
    ATTEMPT = ATTEMPT + 1;
    CALL DO_UPDATE(RECORD_ID);
END;
---------------------------------------------------------------------------
ON LOCKFAIL:
    DELAY = DELAY + 1;
    $SLEEP(DELAY * 1000);
ON DONE:
    -- Success exit
ON ERROR:
    ATTEMPT > MAX_TRIES;                | Y N
    ----------------------------------------+-----
    SIGNAL LOCK_TIMEOUT;                | 1
    CONTINUE;                           |   1
```

### Validation Rule (Event Rule)
```
-- Event rule type: V (Validation), Access: IR (Insert/Replace)
VALIDATE_EMPLOYEE;
---------------------------------------------------------------------------
EMPLOYEES.SALARY > 0;                   | Y N
EMPLOYEES.DEPT ¬= '';                   |   Y
------------------------------------------------------------+---------------
GET DEPARTMENTS WHERE DEPTNO = EMPLOYEES.DEPT; | 1 1
RETURN('Y');                            | 2
RETURN('N');                            |     1
---------------------------------------------------------------------------
ON GETFAIL:
    CALL MSGLOG('Invalid department');
    RETURN('N');
```

### Derived Field Rule (Event Rule)
```
-- Event rule type: D (Derived), Access: G (Get)
CALC_FULL_NAME;
---------------------------------------------------------------------------
EMPLOYEES.FULL_NAME = EMPLOYEES.FNAME || ' ' || EMPLOYEES.LNAME;
```

## Anti-Patterns to Refactor

### Deeply Nested FORALL (Refactor)
```
-- BAD: Deep nesting, poor performance
FORALL A:
    FORALL B:
        FORALL C:
            FORALL D:
                -- processing
            END;
        END;
    END;
END;

-- BETTER: Extract inner loops to separate rules
FORALL A:
    CALL PROCESS_B_LEVEL(A.KEY);
END;
```

### Missing Exception Handlers (Refactor)
```
-- BAD: No handlers, fails silently
GET CUSTOMERS WHERE CUST# = INPUT_ID;
CALL PROCESS_CUSTOMER;

-- BETTER: Handle expected exceptions
GET CUSTOMERS WHERE CUST# = INPUT_ID;
CALL PROCESS_CUSTOMER;
ON GETFAIL:
    CALL MSGLOG('Customer not found: ' || INPUT_ID);
    SIGNAL CUSTOMER_NOT_FOUND;
```

### Hardcoded Values (Refactor)
```
-- BAD: Magic numbers
DISCOUNT = ORDER_AMT * 0.15;

-- BETTER: Use configuration table
GET CONFIG WHERE CONFIG_KEY = 'DISCOUNT_RATE';
DISCOUNT = ORDER_AMT * CONFIG.CONFIG_VALUE;
```

### Overly Complex Condition Quadrant (Refactor)
```
-- BAD: Too many columns, hard to read
cond1;  | Y Y Y Y N N N N
cond2;  | Y Y N N Y Y N N
cond3;  | Y N Y N Y N Y N
--------+----------------
act1;   | 1
act2;   |   1
...

-- BETTER: Split into multiple rules with clear names
CALL PROCESS_TYPE_A;    -- When cond1=Y, cond2=Y, cond3=Y
CALL PROCESS_TYPE_B;    -- When cond1=Y, cond2=Y, cond3=N
-- etc.
```

## Performance Patterns

### Use Browse Mode for Reads
```
-- Explicit browse for read-only
EXECUTE IN BROWSE report_query;

-- Or use SUB table (inherently browse)
GET EMPLOYEE_VIEW WHERE DEPT = CURRENT_DEPT;
```

### Specify First Primary Key
```
-- GOOD: Uses index
GET ORDERS WHERE ORDER# = target_id;

-- BAD: Table sweep
GET ORDERS WHERE CUST_NAME = target_name;
```

### Batch Commits for Large Updates
```
UPDATE_ALL_SALARIES(PCT_INCREASE);
LOCAL COUNT;
---------------------------------------------------------------------------
COUNT = 0;
FORALL EMPLOYEES UNTIL GETFAIL:
    EMPLOYEES.SALARY = EMPLOYEES.SALARY * (1 + PCT_INCREASE);
    REPLACE EMPLOYEES;
    COUNT = COUNT + 1;
    COUNT >= 100;                       | Y N
    ----------------------------------------+-----
    COMMIT;                             | 1
    COUNT = 0;                          | 2
END;
COMMIT;  -- Final commit for remaining
```

## Integration Patterns

### External DB Access (DB2)
```
GET_DB2_CUSTOMER(CUST_ID);
---------------------------------------------------------------------------
GET DB2_CUSTOMERS WHERE CUSTOMER_ID = CUST_ID;
RETURN(DB2_CUSTOMERS.CUSTOMER_NAME);
---------------------------------------------------------------------------
ON GETFAIL:
    RETURN('');
ON SERVERFAIL:
    CALL MSGLOG('DB2 connection failed');
    SIGNAL DB2_ERROR;
```

### Async Processing via SCHEDULE
```
SUBMIT_BATCH_REPORT(REPORT_TYPE, PARAMS);
---------------------------------------------------------------------------
SCHEDULE TO BATCH_QUEUE generate_report(REPORT_TYPE, PARAMS);
CALL MSGLOG('Report scheduled for batch processing');
```
