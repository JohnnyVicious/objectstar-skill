# ObjectStar Syntax Reference

This document provides key syntax patterns for TIBCO ObjectStar rules language (Object Service Broker), applicable to both OTP and batch contexts.

## Structure of a Rule
```plaintext
RULE RuleName
CONDITION:
    /* optional condition */
ACTION:
    /* main logic */
EXCEPTION:
    /* exception handling */
END RULE
```

## Data Access
- **GET**
```plaintext
GET TABLE WHERE KEY = value;
ON GETFAIL:
  /* handle not found */
ENDON;
```

- **FORALL**
```plaintext
FORALL TABLE WHERE condition ORDERED BY field:
  /* logic */
END;
```

- **REPLACE / INSERT / DELETE**
```plaintext
REPLACE TABLE WHERE KEY = value;
ON REPLACEFAIL:
  /* record not found */
ENDON;

INSERT TABLE;
ON INSERTFAIL:
  /* duplicate key */
ENDON;

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
```plaintext
ON GETFAIL TABLE:
  SIGNAL ERROR;
ENDON;

ON ERROR:
  ROLLBACK;
  DISPLAY ERROR_SCREEN;
ENDON;
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

