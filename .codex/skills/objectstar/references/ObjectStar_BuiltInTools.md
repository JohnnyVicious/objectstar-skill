# ObjectStar Built-in Tools Reference

Built-in functions (shareable tools) available in all rules. Call directly without CALL statement.

## String Functions

| Function | Description | Example |
|----------|-------------|---------|
| `LENGTH(s)` | String length | `LENGTH('hello')` → 5 |
| `SUBSTRING(s,start,len)` | Extract substring | `SUBSTRING('hello',2,3)` → 'ell' |
| `HEADSTRING(s,len)` | First n chars | `HEADSTRING('hello',3)` → 'hel' |
| `TAILSTRING(s,len)` | Last n chars | `TAILSTRING('hello',3)` → 'llo' |
| `UPPERCASE(s)` | Convert to upper | `UPPERCASE('hello')` → 'HELLO' |
| `LOWERCASE(s)` | Convert to lower | `LOWERCASE('HELLO')` → 'hello' |
| `PAD(s,len,char,just)` | Pad string | `PAD('hi',5,' ','L')` → 'hi   ' |
| `MATCH(s,pattern)` | Pattern match | `MATCH('abc','a*')` → 'Y' |
| `QUOTE(s)` | Add quotes | `QUOTE('text')` → "'text'" |
| `TRIM(s)` | Remove whitespace | `TRIM('  hi  ')` → 'hi' |
| `REPLACE(s,old,new)` | Replace substring | `REPLACE('abc','b','x')` → 'axc' |

## Date/Time Functions

| Function | Description | Example |
|----------|-------------|---------|
| `$SYSTEMDATE` | Current date | Returns system date |
| `$ADD_DATE(date,comp,amt)` | Add to date | `$ADD_DATE(date,'M',1)` → +1 month |
| `$CREATE_DATE(pic,str)` | Parse date | `$CREATE_DATE('YYYYMMDD','20240115')` |
| `$DATE_PIC(pic,date)` | Format date | `$DATE_PIC('MM/DD/YYYY',date)` |
| `REALTIME` | Current time | Returns HH:MM:SS |
| `HOUR` / `MINUTE` / `SECOND` | Time components | Returns 0-23, 0-59, 0-59 |
| `$YEAR(date)` | Extract year | `$YEAR($SYSTEMDATE)` |
| `$MONTH(date)` | Extract month | `$MONTH($SYSTEMDATE)` |
| `$DAY(date)` | Extract day | `$DAY($SYSTEMDATE)` |

**$ADD_DATE components**: Y (year), M (month), D (day), H (hour), N (minute), S (second)

## Math Functions

| Function | Description | Example |
|----------|-------------|---------|
| `ABS(v)` | Absolute value | `ABS(-5)` → 5 |
| `MAX(x,y)` | Maximum | `MAX(3,7)` → 7 |
| `MIN(x,y)` | Minimum | `MIN(3,7)` → 3 |
| `MOD(div,divisor)` | Modulo | `MOD(10,3)` → 1 |
| `ROUND(v)` | Round to integer | `ROUND(3.7)` → 4 |
| `RANDOM(limit)` | Random 0 to limit-1 | `RANDOM(100)` → 0-99 |
| `TRUNCATE(v)` | Truncate decimals | `TRUNCATE(3.7)` → 3 |

## Type Inspection Functions

| Function | Description | Returns |
|----------|-------------|---------|
| `$GET_TYPE(v)` | Get semantic type | C/D/I/L/Q/S |
| `$GET_SYNTAX(v)` | Get syntax type | B/C/P/V/F/UN |
| `$GET_SIZE(v)` | Get byte size | Integer |
| `$TYPECAST(type,syn,size,dec,val)` | Type conversion | Cast value |
| `$ISNULL(v)` | Check if null | Y/N |

## System Functions

| Function | Description |
|----------|-------------|
| `USERID` | Current user ID |
| `LIBID` | Current library ID |
| `$SYSTEMNAME` | System name |
| `$ENVIRONMENT` | Environment name |
| `$TASKID` | Current task ID |
| `$TRANSACTIONID` | Current transaction ID |

## Messaging Functions

| Function | Description | Example |
|----------|-------------|---------|
| `MSGLOG(msg)` | Log message | `MSGLOG('Processing started')` |
| `ENDMSG(msg)` | End with message | `ENDMSG('Process complete')` |
| `SCREENMSG(screen,msg)` | Screen message | `SCREENMSG(MYSCREEN,'Saved')` |

## Option Functions

| Function | Description |
|----------|-------------|
| `$GETOPT(option)` | Get system option value |
| `$SETOPT(option,value)` | Set system option |

**Common options**: `COMMITUSED`, `COMMITSIZE`, `MAXLOCK`, `TRACE`

## Conversion Functions

| Function | Description | Example |
|----------|-------------|---------|
| `NUMERIC(s)` | String to number | `NUMERIC('123')` → 123 |
| `STRING(n)` | Number to string | `STRING(123)` → '123' |
| `HEX(v)` | To hexadecimal | `HEX(255)` → 'FF' |
| `UNHEX(s)` | From hexadecimal | `UNHEX('FF')` → 255 |

## Screen Functions

| Function | Description |
|----------|-------------|
| `ENTERKEY(screen)` | Last function key pressed |
| `CURSORFIELD(screen)` | Field containing cursor |
| `SETCURSOR(screen,table,field)` | Position cursor |
| `DELETESCREENDATA(screen)` | Clear all screen data |
| `$SETATTRIBUTE(screen,table,field,attr,flag)` | Set field attribute |
| `$GETATTRIBUTE(screen,table,field,attr)` | Get field attribute |

**Attributes**: PROTECTED, HIDDEN, BRIGHT, REVERSE, UNDERSCORE

## Table Functions

| Function | Description |
|----------|-------------|
| `$ROWCOUNT(table)` | Rows in result set |
| `$TABLEEXISTS(table)` | Check table exists |
| `$FIELDEXISTS(table,field)` | Check field exists |

## Control Functions

| Function | Description | Example |
|----------|-------------|---------|
| `$SLEEP(ms)` | Pause execution | `$SLEEP(1000)` → 1 sec |
| `CONTINUE` | Continue loop | Skip to next iteration |
| `RETURN(v)` | Return value | `RETURN('Y')` |

## File Functions

| Function | Description |
|----------|-------------|
| `$OPEN(file,mode)` | Open file |
| `$CLOSE(file)` | Close file |
| `$READ(file,buffer)` | Read from file |
| `$WRITE(file,buffer)` | Write to file |
| `$EOF(file)` | Check end of file |

**Modes**: 'R' (read), 'W' (write), 'A' (append)

## Usage Examples

### String Manipulation
```
LOCAL FULL_NAME, INITIALS;
FULL_NAME = EMPLOYEES.FNAME || ' ' || EMPLOYEES.LNAME;
INITIALS = HEADSTRING(EMPLOYEES.FNAME,1) || HEADSTRING(EMPLOYEES.LNAME,1);
```

### Date Calculation
```
LOCAL NEXT_REVIEW, FORMATTED;
NEXT_REVIEW = $ADD_DATE($SYSTEMDATE, 'M', 6);
FORMATTED = $DATE_PIC('MM/DD/YYYY', NEXT_REVIEW);
CALL MSGLOG('Next review: ' || FORMATTED);
```

### Commit Threshold Check
```
LOCAL USED, SIZE;
USED = $GETOPT('COMMITUSED');
SIZE = $GETOPT('COMMITSIZE');
(USED / SIZE) > 0.8;                 | Y N
------------------------------------+-----
COMMIT;                             | 1
CALL MSGLOG('Commit threshold');    | 2
```
