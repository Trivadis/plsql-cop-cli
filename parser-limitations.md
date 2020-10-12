# Parser Limitations

These document covers the known limitations of parsing SQL*Plus, SQL and PL/SQL code in PL/SQL Cop command line, PL/SQL Cop for SQL Developer and PL/SQL Analyzer.

## Maxim

If your SQL*Plus script runs successfully against an Oracle database but PL/SQL Cop or PL/SQL Analyzer reports an error, this is usually considered a bug. However, there are some known exceptions to this basic principle, which are documented below.

## SQL\*Plus Parser

The SQL\*Plus parser is a so-called shallow parser. It covers the bare minimum to identify SQL\*Plus commands, SQL statements and anonymous PL/SQL blocks. It is designed for basic metric calculation and for collaboration with the PL/SQL parser. It is specifically not designed for code validations.

## PL/SQL Parser

The PL/SQL parser treats SQL and PL/SQL as a single language. The goal is to parse the following statements completely for code validation purposes:

- Anonymous PL/SQL Block (PL/SQL Unit)
- Create Function
- Create Package
- Create Package Body
- Create Procedure
- Create Trigger
- Create Type
- Create Type Body
- Create View
- Call
- Commit
- Delete
- Explain Plan
- Insert
- Lock Table
- Merge
- Select
- Update
- Rollback
- Savepoint
- Set Constraint
- Set Transaction

Validator checks can be implemented only for these statements and their subcomponents.

## Block Terminator

Other block terminators than a dot (`.`) are not supported. This means the `SET BLOCKTERMINATOR` command is ignored.

## Command Separator

Other command separators than semicolon (`;`) are not supported. This means that the `SET CMDSEP` command is ignored.

## SQL Terminator

Other SQL terminators than semicolon (`;`) are not supported. This means that the `SET SQLTERMINATOR` command is ignored. Tailing whitespaces after a SQL terminator are not supported.

## Line Continuation Character

Tailing whitespaces after a line continuation character (`-`) are not supported.

## Slash Command

Tailing whitespaces after the slash command (`/`) are not supported.

## Execute Command

The execute command must end on semicolon (`;`) if the last token is an expression.

## Remark Command

The remark must not contain unterminated single (`'`) or double quotes (`"`).

## Prompt Command

The prompt must not contain unterminated single (`'`) or double quotes (`"`).

## Use of Keywords

The use of PL/SQL and SQL keywords as unquoted identifiers are generally not supported, due to the fact, that every single keyword needs to be treated as an exception. 

Oracle is quite gracious in that area and therefore we strive to support more and more keywords as unquoted identifiers with each release, but the following keywords are causing conflicts in certain parts of the grammar and the use as literals should therefore be avoided: 

`CROSS`, `CURSOR`, `END`, `EXCLUDE`, `FULL`, `FUNCTION`, `INSTANTIABLE`, `INNER`, `JOIN`, `LEFT`, `MODEL`, `OFFSET`, `OUTER`, `RIGHT`, `ROWTYPE`, `TYPE`.

Query the dictionary view `V$RESERVED_WORDS` for a list of keywords. Please note that a parsing error caused by using a keyword as an unquoted identifier is not considered a PL/SQL COP bug, regardless of the keyword categorization by columns such as `RESERVED`.

## Quote Delimiter Characters

Oracle supports various quote literal characters except space, tab and return. The following example uses the `[]` quote character pair:

```
nq'[that’s very cool!]'
```

The following quote characters-pairs are supported: 

`$$`, `##`, `@@`, `££`, `""`, `||`, `()`, `{}`, `[]`, `<>`, `!!`, `++`, `~~`, `//`. 

All other quote characters lead to parse errors.

## Conditional Compilation

Up until PL/SQL Cop version 1.0.16, PL/SQL Cop for SQL Developer 1.0.12, PL/SQL Analyzer 1.0.7 conditional compilation blocks have been fully analysed in the PL/SQL body, but were not supported in the PL/SQL DECLARE section.

Since it is possible to store non-PL/SQL code within conditional compilation blocks, e.g. generation templates as used in FTLDB or tePSQL, the full-fletched analysis support of directive `IF` statements has been dropped. The `$IF … $END` and the `$ERROR … $END` code blocks are still recognised as statements/expressions including conditions, but the rest of the code is just parsed as a series of tokens. As a side effect, metrics such as the number of statements might change.

The current PL/SQL parser supports conditional compilation within the `DECLARE` section as `ITEM_LIST_1` or `ITEM_LIST_2`.

Example of a supported directive `IF` in the `DECLARE` section:

```sql
CREATE OR REPLACE PACKAGE my_pkg AS 
   $IF DBMS_DB_VERSION.VERSION < 10 $THEN 
      SUBTYPE my_real IS NUMBER;
   $ELSE 
      SUBTYPE my_real IS BINARY_DOUBLE;
   $END
   my_pi my_real;
   my_e my_real;
END my_pkg;
/
```

Example of an unsupported directive IF in the DECLARE section

```sql
CREATE OR REPLACE PACKAGE my_pkg AS 
   SUBTYPE my_real IS
      $IF DBMS_DB_VERSION.VERSION < 10 $THEN 
         NUMBER;
      $ELSE 
         BINARY_DOUBLE;
      $END
   my_pi my_real;
   my_e my_real;
END my_pkg;
/
```

Empty branches are not supported. The following code leads to parse errors:

```sql
CREATE OR REPLACE PROCEDURE p IS
BEGIN
   NULL;
   $IF $$something = 1 $THEN
   $ELSE
      dbms_output.put_line('enabled');
   $END
END p;
```

Instead use the following, without empty branches:

```sql
CREATE OR REPLACE PROCEDURE p IS
BEGIN
   NULL;
   $IF $$something != 1 $THEN
      dbms_output.put_line('enabled');
   $END
END p;
```

## FOR LOOP Statement

The `lower_bound` and `upper_bound` are separated with a `..`. Whitspaces before and after `..` are not required according the PL/SQL grammar. 

However, if the `lower_bound` expression ends on a number, then a one of the following is required:

- a whitespace before the `..`
- a whitespace after the `..`
- `lower_bound` in parenthesis

Example of supported `FOR LOOP`:

```sql
BEGIN
   FOR i IN length(l_value)+1 .. 500 LOOP
      dbms_output.put_line('supported');
   END LOOP;
END;
```

Example of unsupported `FOR LOOP`:

```sql
BEGIN
   FOR i IN length(l_value)+1..500 LOOP
      dbms_output.put_line('unsupported');
   END LOOP;
END;
```

## Error Logging Clause
The keyword `log` is supported as table name and table alias. As a side effect `DELETE` and `INSERT` statements with an `error_logging_clause` but without a `where_clause` and without table alias cannot be supported.

Example of supported `INSERT` statement with an `error_logging_clause`

```sql
INSERT INTO deptsal (dept_no, dept_name, salary)
     SELECT dept_no, dept_name, salary
       FROM source_syn s
 LOG ERRORS INTO deptsal_err REJECT LIMIT 10;
```

Example of unsupported `INSERT` statement with an `error_logging_clause` (`source_syn` has no alias)

```sql
INSERT INTO deptsal (dept_no, dept_name, salary)
     SELECT dept_no, dept_name, salary
       FROM source_syn
 LOG ERRORS INTO deptsal_err REJECT LIMIT 10;
```

## PL/SQL Source Text Wrapping

Since PL/SQL Cop and PL/SQL Analyzer do not include a PL/SQL unwrap utility, the use of wrapped PL/SQL code is not supported.

## Supported Oracle Versions

The PL/SQL and SQL grammars from Oracle version 7.0 until version 12.2 are supported. The language is based on the following documentation:

- [Oracle SQL\*Plus User’s Guide and Reference, 12c Release 2 (12.2)](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sqpug/index.html), E50028-08, January 2017
- [Oracle SQL Language Reference, 12c Release 2 (12.2)](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sqlrf/index.html), E49448-12, January 2017
- [Oracle PL/SQL Language Reference, 12c Release 2 (12.2)](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/lnpls/index.html), E49633-15, January 2017

Grammar changes and enhancements made in newer versions are not yet covered.
