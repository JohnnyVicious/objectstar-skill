# Chapter 3: Supported Characters

## Lexical Elements

A rule is a sequence of lexical elements:
- Delimiters
- Hexadecimal literals (`X'...'`)
- Identifiers (including reserved words)
- Numeric literals
- Raw-data literals (`R'...'`)
- String literals (`'...'`)
- Unicode literals (`U'...'`)

## Delimiters

### Single Character Delimiters
```
| & ¬ * ( ) - + = : ; ' , / < >
```

### Compound Symbol Delimiters
```
** <= >= ¬= || R' r' U' u' X' x'
```

## Tokens

Each lexical element is called a **token**.

### Token Rules
- Adjacent tokens separated by spaces, delimiters, or new line
- Identifier or numeric literal must be separated from adjacent identifier/numeric
- Only string literals and Unicode literals can contain spaces
- String, hexadecimal, raw data, and Unicode literals can span multiple lines
- All other lexical elements must fit on a single line

### Literal Formats
| Format | Example | Description |
|--------|---------|-------------|
| Hexadecimal | `X'4A'` | Hex character values |
| Raw data | `R'4A4B'` | Raw byte data |
| Unicode | `U'text'` | Unicode characters |

## Reserved Words

The following are reserved keywords in the rules language:

| | | |
|---|---|---|
| AND | ASCENDING | BROWSE |
| CALL | COMMIT | CONTINUE* |
| DELETE | DESCENDING | DISPLAY |
| END | EXECUTE | FORALL |
| GET | IN | INSERT |
| LIKE | LOCAL | NOT |
| NULL | ON | OR |
| ORDERED | PRINT | REPLACE |
| RETURN | ROLLBACK | SCHEDULE |
| SIGNAL | TO | TRANSFERCALL |
| UNTIL | UPDATE | WHERE |

*CONTINUE is reserved but not currently used in the rules language.

## Character Set

### Uppercase and Lowercase Letters
```
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
a b c d e f g h i j k l m n o p q r s t u v w x y z
```

### Digits
```
0 1 2 3 4 5 6 7 8 9
```

### Special Characters
```
@ # $ | ¬ & * ( ) - _ = + ; : ' , . / < >
```

### Space Character
The space character is also part of the character set.

## National Character Set Support

### Allowable Usage

If hardware supports national character sets, they can be used in:
- Tables with field syntax C (fixed-length), V (variable-length), or W (double-byte)
- Screens and reports as literals
- Quoted strings

**Note**: Alphabetic characters in OSB names must be standard A-Z letters only.

### Behavior
- Characters sorted in EBCDIC order
- Control characters filtered when printing
- Uppercase/lowercase characters mapped to each other

### Available Character Sets

| Code | Language | English Name |
|------|----------|--------------|
| CDNB | Canadian Bilingual | Canadian Bilingual |
| DANS | Dansk | Danish/Norwegian |
| DEUT | Deutsch | Austrian/German |
| ENGB | English | English (Great Britain) |
| ENGL | English | English (US) |
| ESPA | Español | Spanish |
| FRAN | Français | French |
| ITAL | Italiano | Italian |
| NORS | Norsk | Danish/Norwegian |
| PORT | Português | Portuguese |
| SCHW | Schweiz | Swiss/French and Swiss/German |
| SUOM | Suomi | Finnish/Swedish |
| SVEN | Svenska | Finnish/Swedish |

## Double-byte Character String Support

### Allowable Usage (z/OS only)

Double-byte characters can be used in:
- Tables with syntax W fields
- Arguments to rules (if no specific syntax required)

**Cannot** be used as literals on report tables or screen tables.

### Behavior
- Converted to single-byte equivalent in assignment statements
- Sorted in EBCDIC order
- Comparison based on EBCDIC collating sequence
- Control characters SI/SO not printed to double-byte devices

## Use of Quotation Marks

### String Literals

String literals require single quotation marks:
```
GET EMPLOYEES WHERE REGION='MIDWEST';
```

### Quotation Marks Within Strings

Use two quotation marks to include one:
```
'I''M'  →  represents: I'M
```

### Slashes Within Unicode Literals

Use two slashes (since `/` is escape character):
```
U'I//O'  →  represents: I/O
```

### Unicode Escape Sequences
```
U'/20AC'  →  Euro sign (€)
```

### OSB Object Names

Object names, rule arguments, and local variables are **not** enclosed in quotes.

**Exception**: When passing as absolute value:
```
INSERT EMPLOYEE_INFO WHERE SCREEN='NEW_EMPLOYEE';
```

### Numeric Literals

Generally do **not** require quotation marks.

**Exception - Primary Keys**: When referring to C or V syntax primary key with leading zeros:
```
MONTH='01'   ← Correct (preserves leading zero)
MONTH=01     ← Incorrect (would match '1')
MONTH=1      ← Incorrect
```
