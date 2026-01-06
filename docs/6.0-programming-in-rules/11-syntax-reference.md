# Appendix A: Syntax of the Rules Language (BNF Notation)

## BNF Notation Conventions

| Convention | Meaning |
|------------|---------|
| `< >` | Syntactic category (e.g., `<identifier>`) |
| `::=` | "is defined to be" |
| `{ }` | Repeated item (zero or more times) |
| `[ ]` | Optional item |
| New line | Alternative in a list |

---

## Identifiers

Identifiers cannot exceed 16 characters.

```bnf
<identifier> ::=
    <start character> {<follow character>}

<start character> ::=
    A B C D E F G H I J K L M N O P Q R S T U V W X Y Z @ # $

<follow character> ::=
    <start character>
    <digit>
    _

<digit> ::=
    0 1 2 3 4 5 6 7 8 9
```

---

## Numeric Literals

```bnf
<numeric literal> ::=
    <digits> [ . <digits> ] [ <exponent> ]
    [<digits> ] . <digits> [ <exponent> ]

<digits> ::=
    <digit> { <digit> }

<digit> ::=
    0 1 2 3 4 5 6 7 8 9

<exponent> ::=
    E [ <sign> ] <digits>
    e [ <sign> ] <digits>

<sign> ::=
    +
    -
```

---

## String Literals

```bnf
<string literal> ::=
    <plain character string>
    <hexadecimal character string>
    <Unicode character string>
    <hexadecimal byte string>

<plain character string> ::=
    '{<character>}'

<hexadecimal character string> ::=
    x'{<hexadecimal digit><hexadecimal digit>}'
    X'{<hexadecimal digit><hexadecimal digit>}'

<Unicode character string> ::=
    u'{<Unicode character>}'
    U'{<Unicode character>}'

<hexadecimal byte string> ::=
    r'{<hexadecimal digit><hexadecimal digit>}'
    R'{<hexadecimal digit><hexadecimal digit>}'

<hexadecimal digit> ::=
    0 1 2 3 4 5 6 7 8 9 a A b B c C d D e E f F

<Unicode character> ::=
    <character other than />
    /<hexadecimal digit><hexadecimal digit><hexadecimal digit><hexadecimal digit>
    //
```

### String Literal Types

| Type | Example | Description |
|------|---------|-------------|
| Plain | `'text'` | Typeless, syntax V |
| Hexadecimal | `X'4A4B'` | Characters by hex value, converts to plain if printable |
| Unicode | `U'text/20AC'` | UTF-16 characters, syntax UN |
| Raw data | `R'4A4B'` | Byte values, syntax RD |

---

## Rule Syntax

```bnf
<rule> ::=
    <rule declaration> <condition list> <action list> <exception list>

<rule declaration> ::=
    <rule header> [<local name declaration>]

<rule header> ::=
    <rule name> [<rule header argument list>] ;

<rule header argument list> ::=
    (<rule argument name> {,<rule argument name>})

<local name declaration> ::=
    LOCAL <local name> {,<local name>} ;

<condition list> ::=
    {<condition>;}

<condition> ::=
    [<not>] <logical value>
    <expression> <relational operator in condition> <expression>

<logical value> ::=
    <field of a table>
    <rule argument name>
    <function call>

<action list> ::=
    <action> {<action>}

<exception list> ::=
    {<on exception>}

<on exception> ::=
    ON <exception designation> : {<action>}

<action> ::=
    <statement> ;
```

---

## Statements

```bnf
<statement> ::=
    <assignment>
    <rule invocation>
    <function return>
    <table access statement>
    <synchronous processing>
    <output processing>
    <signal exception>
    <asynchronous call>
    <iterative processing>
```

---

## Assignment

```bnf
<assignment> ::=
    <assignment target> = <expression>
    <assignment-by-name>

<assignment target> ::=
    <field of a table>
    <local name>

<assignment-by-name> ::=
    <table reference> .* = <table reference> .*
    <table reference> .* = NULL
```

---

## Rule Invocation

```bnf
<rule invocation> ::=
    <invocation> <invocation specification> [<invocation arguments>]

<invocation> ::=
    CALL
    EXECUTE [IN <browse specification>]
    TRANSFERCALL [IN <browse specification>]

<invocation specification> ::=
    <rule name>
    <rule argument name>
    <table name> . <field name>

<invocation arguments> ::=
    <argument list>
    WHERE <where argument list>

<where argument list> ::=
    <where argument item> {<and> <where argument item>}

<where argument item> ::=
    <identifier> = <expression>

<function return> ::=
    RETURN ( <expression> )

<browse specification> ::=
    BROWSE
    UPDATE
```

---

## Table Access Statements

```bnf
<table access statement> ::=
    <get statement>
    <insert statement>
    <replace statement>
    <delete statement>
    <forall statement>

<get statement> ::=
    GET <occurrence specification> [<table order>] [WITH MINLOCK]

<occurrence specification> ::=
    <table specification> [WHERE <where predicate>]

<table specification> ::=
    <table name> [<argument list>]
    <rule argument name> [<argument list>]
    <table name> . <field name> [<argument list>]

<insert statement> ::=
    INSERT <table specification> [WHERE <where parameter list>]

<replace statement> ::=
    REPLACE <table specification> [WHERE <where parameter list>]

<delete statement> ::=
    DELETE <table specification> [WHERE <where primary key>]

<where primary key> ::=
    <primary key> = <expression>

<forall statement> ::=
    FORALL <occurrence specification> [<table order>] [<until specification>] :
        <for action list>
    END

<until specification> ::=
    UNTIL <exceptions>

<for action list> ::=
    {<for action> ;}

<for action> ::=
    <assignment>
    <rule invocation>
    <table access statement>
    <output processing>
    <asynchronous call>
    <iterative processing>
    COMMIT
```

---

## WHERE Clause

```bnf
<where predicate> ::=
    <where not expression> {<logical operator> <where not expression>}

<where not expression> ::=
    [<not>] <where expression>

<where expression> ::=
    <where relation>
    (<where predicate>)

<where relation> ::=
    <field reference> <relational operator> <where expression>

<where expression> ::=
    [<unary operator>] <where expression term> {<add operator> <where expression term>}

<where expression term> ::=
    <where expression factor> {<multiplication operator> <where expression factor>}

<where expression factor> ::=
    <where expression primary> [<exponent operator> <where expression primary>]

<where expression primary> ::=
    (<where expression>)
    <where field of a table>
    <rule argument name>
    <local name>
    <function call>
    <literal>

<where field of a table> ::=
    <where table reference> . <field reference>

<where table reference> ::=
    *
    <table name>
    (<rule argument name>)
    (<table name> . <field name>)

<where parameter list> ::=
    <where parameter item> {<and> <where parameter item>}

<where parameter item> ::=
    <table parameter name> = <expression>
```

**Note**: `<where table reference>` allows `*` for current table.

---

## Ordering

```bnf
<table order> ::=
    <table order item> {AND <table order item>}

<table order item> ::=
    ORDERED [<ordering>] <field reference>

<ordering> ::=
    ASCENDING
    DESCENDING
```

---

## Synchronization and Output

```bnf
<synchronous processing> ::=
    COMMIT
    ROLLBACK

<output processing> ::=
    PRINT <report reference>
    DISPLAY <screen reference> [<and option>]

<report reference> ::=
    <report name>
    <table name> . <field name>
    <rule argument name>

<screen reference> ::=
    <screen name>
    <table name> . <field name>
    <rule argument name>

<and option> ::=
    <and> TRANSFERCALL [IN <browse specification>] <invocation specification>
        [<invocation arguments>]
```

---

## Exception Handling

```bnf
<signal exception> ::=
    SIGNAL <exception name>

<exceptions> ::=
    <exception designation> {<or> <exception designation>}

<exception designation> ::=
    <exception name> [<table name>]

<exception name> ::=
    <identifier>
```

---

## Asynchronous Processing

```bnf
<asynchronous call> ::=
    SCHEDULE [IN <browse specification>] [<queue specification>]
        <rule name> [<call arguments>]

<queue specification> ::=
    TO <expression>
```

---

## Iterative Processing

```bnf
<iterative processing> ::=
    <until statement>

<until statement> ::=
    UNTIL <exceptions> [DISPLAY <screen reference>] :
        {<action>}
    END
```

---

## Expressions

```bnf
<expression> ::=
    [<unary operator>] <expression term> {<add operator> <expression term>}

<expression term> ::=
    <expression factor> {<multiplication operator> <expression factor>}

<expression factor> ::=
    <expression primary> [<exponent operator> <expression primary>]

<expression primary> ::=
    (<expression>)
    <field of a table>
    <rule argument name>
    <local name>
    <function call>
    <literal>

<field of a table> ::=
    <table reference> . <field reference>

<table reference> ::=
    <table name>
    (<rule argument name>)
    (<table name> . <field name>)

<field reference> ::=
    <field name>
    (<rule argument name>)
    (<table name> . <field name>)

<function call> ::=
    <function name> [<argument list>]

<argument list> ::=
    (<expression> {,<expression>})
```

---

## Operators

```bnf
<unary operator> ::=
    -
    +

<add operator> ::=
    +
    -
    ||

<multiplication operator> ::=
    *
    /

<exponent operator> ::=
    **

<logical operator> ::=
    <and>
    <or>

<and> ::=
    AND
    &

<or> ::=
    OR
    |

<not> ::=
    NOT
    ¬

<relational operator in condition> ::=
    =
    ¬=
    >
    >=
    <
    <=

<relational operator> ::=
    <relational operator in condition>
    LIKE
```

---

## Literals and Names

```bnf
<literal> ::=
    <string literal>
    <numeric literal>
    NULL

<rule name> ::=
    <identifier>

<function name> ::=
    <identifier>

<rule argument name> ::=
    <identifier>

<table name> ::=
    <identifier>

<field name> ::=
    <identifier>

<local name> ::=
    <identifier>

<report name> ::=
    <identifier>

<screen name> ::=
    <identifier>

<table parameter name> ::=
    <identifier>

<primary key> ::=
    <field name>
```

**Note**: The keyword `NULL` can only appear by itself, not as an operand in an expression. `NULL + 1` is syntactically incorrect.
