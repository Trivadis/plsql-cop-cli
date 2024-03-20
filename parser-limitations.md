# Parser Limitations

These document covers the known limitations of parsing SQL*Plus, SQL and PL/SQL code in db\* CODECOP command line, db\* CODECOP for SQL Developer and PL/SQL Analyzer.

## Maxim

If your SQL*Plus script runs successfully against an Oracle database but db\* CODECOP reports an error, this is usually considered a bug. However, there are some known exceptions to this basic principle, which are documented below.

## SQL\*Plus Parser

The SQL\*Plus parser is a so-called shallow parser. It covers the bare minimum to identify SQL\*Plus and SQLcl commands, SQL statements and anonymous PL/SQL blocks. It is designed for basic metric calculation and for collaboration with the PL/SQL parser. It is specifically not designed for code validations.

This parser needs lines without trailing spaces as input. Otherwise, the metrics regarding the number of commands can be wrong and parse errors can occur when the SQL\*Plus parser passes SQL\*Plus commands to the PL/SQL parser. We could remove trailing spaces before parsing, but this could lead to positional differences in the file or editor when reporting validation issues. Therefore, we strongly recommend that you ensure that your input does not contain trailing spaces, e.g. by configuring your editors accordingly.

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

Other block terminators than a dot (`.`) are not supported. This means the `set blockterminator` command is ignored.

## Command Separator

Other command separators than semicolon (`;`) are not supported. This means that the `set cmdsep` command is ignored.

## SQL Terminator

Other SQL terminators than semicolon (`;`) are not supported. This means that the `set sqlterminator` command is ignored. Tailing whitespace after a SQL terminator are not supported.

## Line Continuation Character

Tailing whitespace after a line continuation character (`-`) are not supported.

## Slash Command

Tailing whitespace after the slash command (`/`) are not supported.

## Execute Command

The execute command must end on semicolon (`;`) if the last token is an expression.

## Remark Command

The remark must not contain unterminated single (`'`) or double quotes (`"`).

## Prompt Command

The prompt must not contain unterminated single (`'`) or double quotes (`"`).

## Use of Keywords

The use of PL/SQL and SQL keywords as unquoted identifiers are generally not supported, due to the fact, that every single keyword needs to be treated as an exception. 

Oracle is quite liberal in this area. We try to support existing keywords and keywords introduced in new Oracle database versions as identifiers as long as they do not cause conflicts in our grammar.

For example, the following keywords cannot be used unquoted as table alias since the are conflicting with the SQL grammar:

`cross`, `full`, `inner`, `join`, `json`, `left`, `model`, `natural`, `offset`, `outer`, `right`.

Beginning with version 4.1 function names with non-standard parameters are not supported as unquoted identifiers anymore. Standard parameters follow the notation defined in the [Database PL/SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/21/lnpls/plsql-subprograms.html#GUID-A5DA8CF5-1BCC-4ABE-9B68-DB593FF1D2CC). Examples are [`add_months`](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/ADD_MONTHS.html#GUID-B8C74443-DF32-4B7C-857F-28D557381543), [`greatest`](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/GREATEST.html#GUID-06B88B22-8466-44B6-93C7-50B222122ECE) or [`instr`](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/INSTR.html#GUID-47E3A7C4-ED72-458D-A1FA-25A9AD3BE113).

The following functions use non-standard parameters and therefore cannot be used as unquoted identifiers:

`cast`, `collect`, `feature_compare`, `json_array`, `json_arrayagg`, `json_mergepatch`, `json_object`, `json_objectagg`, `json_query`, `json_scalar`, `json_serialize`, `json_transform`, `json_value`, `listagg`, `to_binary_double`, `to_binary_float`, `to_date`, `to_dsinterval`, `to_number`, `to_timestamp`, `to_timestamp_tz`, `to_yminterval`, `treat`, `validate_conversion`, `xmlagg`, `xmlcast`, `xmlcolattval`, `xmlelement`, `xmlparse`, `xmlpi`.

You can query the dictionary view `v$reserved_words` for a complete list of keywords. Please note that a parsing error caused by using a keyword as an unquoted identifier is not considered a db\* CODECOP bug, regardless of the keyword categorization by columns such as `reserved`.

## Quote Delimiter Characters

Oracle supports various quote literal characters except space, tab and return. The following example uses the `[]` quote character pair:

```
nq'[that’s very cool!]'
```

The following quote characters-pairs are supported: 

`$$`, `##`, `@@`, `££`, `""`, `||`, `()`, `{}`, `[]`, `<>`, `!!`, `++`, `~~`, `//`, `§§`. 

All other quote characters lead to parse errors.

## Identifiers

The parsers supports only the following characters in unquoted identifiers:

First position:

- `a` to `z`
- `A` to `Z`

Position 2ff:

- `a` to `z`
- `A` to `Z`
- `0` to `9` 
- `_`
- `$`
- `#`
- `ä`/`Ä`
- `ö`/`Ö`
- `ü`/`Ü`

The following error is thrown when other letters are used in an identifier: `E-0002: Syntax error. Please check the limitations and contact the author if the code can be compiled successfully in your environment.`

As a workaround you can use [quoted identifiers](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/plsql-language-fundamentals.html#GUID-C4B3F788-770E-48F8-9A44-ACD7977B1545).

## Conditional Compilation

Up until db\* CODECOP version 1.0.16, db\* CODECOP for SQL Developer 1.0.12, PL/SQL Analyzer 1.0.7 conditional compilation blocks have been fully analysed in the PL/SQL body, but were not supported in the PL/SQL DECLARE section.

Since it is possible to store non-PL/SQL code within conditional compilation blocks, e.g. generation templates as used in FTLDB or tePSQL, the full-fletched analysis support of directive `if` statements has been dropped. The `$if … $end` and the `$error … $end` code blocks are still recognised as statements/expressions including conditions, but the rest of the code is just parsed as a series of tokens. As a side effect, metrics such as the number of statements might change.

The current PL/SQL parser supports conditional compilation within the `declare` section as `item_list_1` or `item_list_2`.

Example of a supported directive `if` in the `declare` section:

```sql
create or replace package my_pkg as 
   $if dbms_db_version.version < 10 $then 
      subtype my_real is number;
   $else 
      subtype my_real is binary_double;
   $end
   my_pi my_real;
   my_e my_real;
end my_pkg;
```

Example of an unsupported directive IF in the DECLARE section

```sql
create or replace package my_pkg as
   subtype my_real is
      $if dbms_db_version.version < 10 $then  
         number;
      $else 
         binary_double;
      $end
   my_pi my_real;
   my_e my_real;
end my_pkg;
```

Empty branches are not supported. The following code leads to parse errors:

```sql
create or replace procedure p is
begin
   null;
   $if $$something = 1 $then
   $else
      dbms_output.put_line('enabled');
   $end
end p;
```

Instead use the following, without empty branches:

```sql
create or replace procedure p is
begin
   null;
   $if $$something != 1 $then
      dbms_output.put_line('enabled');
   $end
end p;
```

## `for loop` Statement

The `lower_bound` and `upper_bound` are separated with a `..`. Whitspaces before and after `..` are not required according the PL/SQL grammar. 

However, if the `lower_bound` expression ends on a number, then a one of the following is required:

- a whitespace before the `..`
- a whitespace after the `..`
- `lower_bound` in parenthesis

Example of supported `FOR LOOP`:

```sql
begin
   for i in length(l_value)+1 .. 500 loop
      dbms_output.put_line('supported');
   end loop;
end;
```

Example of unsupported `FOR LOOP`:

```sql
begin
   for i in length(l_value)+1..500 loop
      dbms_output.put_line('unsupported');
   end loop;
end;
```

## Error Logging Clause
The keyword `log` is supported as table name and table alias. As a side effect `delete` and `insert` statements with an `error_logging_clause` but without a `where_clause` and without table alias cannot be supported.

Example of supported `insert` statement with an `error_logging_clause`

```sql
insert into deptsal (dept_no, dept_name, salary)
     select dept_no, dept_name, salary
       from source_syn s
 log errors into deptsal_err reject limit 10;
```

Example of unsupported `insert` statement with an `error_logging_clause` (`source_syn` has no alias)

```sql
insert into deptsal (dept_no, dept_name, salary)
     select dept_no, dept_name, salary
       from source_syn
 log errors into deptsal_err reject limit 10;
```

## Access Parameters of Inline External Tables

The `obpaque_format_spec` clause has to be provided as a string literal or as a subquery returning a CLOB. Embedding the driver specific parameters directly is not supported. 

Here are examples of a supported and an unsupported statement. These examples are based on [Tim Hall's article "Inline External Tables in Oracle Database 18c"](https://oracle-base.com/articles/18c/inline-external-tables-18c).

Supported example:

```sql
select country_code, count(*) as amount
  from external (
          (
             country_code  varchar2(3),
             object_id     number,
             owner         varchar2(128),
             object_name   varchar2(128)
          )
          type oracle_loader
          default directory tmp_dir1
          access parameters (q'[
             records delimited by newline
             badfile tmp_dir1
             logfile tmp_dir1:'inline_ext_tab_%a_%p.log'
             discardfile tmp_dir1
             fields csv with embedded terminated by ',' optionally enclosed by '"'
             missing field values are null (
                country_code,
                object_id,
                owner,
                object_name 
             )]'
          )
          location ('gbr1.txt', 'gbr2.txt', 'ire1.txt', 'ire2.txt')
          reject limit unlimited
       ) sample (99) inline_ext_tab 
 group by country_code
 order by 1;
```

Unsupported example:

```sql
select country_code, count(*) as amount
  from external (
          (
             country_code  varchar2(3),
             object_id     number,
             owner         varchar2(128),
             object_name   varchar2(128)
          )
          type oracle_loader
          default directory tmp_dir1
          access parameters (
             records delimited by newline
             badfile tmp_dir1
             logfile tmp_dir1:'inline_ext_tab_%a_%p.log'
             discardfile tmp_dir1
             fields csv with embedded terminated by ',' optionally enclosed by '"'
             missing field values are null (
                country_code,
                object_id,
                owner,
                object_name 
             )
          )
          location ('gbr1.txt', 'gbr2.txt', 'ire1.txt', 'ire2.txt')
          reject limit unlimited
       ) inline_ext_tab
 group by country_code
 order by 1;
```

Please note that the supported statement provides the access parameters as a string using `q'[...]'`.

## PL/SQL Source Text Wrapping

Since db\* CODECOP and PL/SQL Analyzer do not include a PL/SQL unwrap utility, the use of wrapped PL/SQL code is not supported.

## SQL\*Plus Substitution Variables

Substitution variables are supported in the SQL\*Plus grammar. This means they work when used in in SQL\*Plus commands such as `connect`. 

When a substitution variable is used in commands supported by the PL/SQL grammar (e.g. the `SELECT` statement), then it can be processed only when it can be replaced by a SQL expression or a condition. 

Here's an example of a supported use of substitution variables:

```sql
select &&column_name 
from emp
where &&where_condition;
```

However, the following example is not supported supported and leads to a parse error:

```sql
select empno, ename 
from emp
&after_from_clause;
```

## Supported Oracle Versions

The PL/SQL and SQL grammars from Oracle version 7.0 until version 23c are supported. 

The grammar implementation is based on the following documentation:

- [Oracle® SQLcl, User's Guide, Release 23.4, F89904-01, January 2024](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/23.4/sqcug/)
- [SQL\*Plus®, User's Guide and Reference, 23c, F47057-03, September 2023](https://docs.oracle.com/en/database/oracle/oracle-database/23/sqpug/)
- [Oracle® Database SQL Language Reference, 23c, F47038-11, March 2024](https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/)
- [Oracle® Database Database, PL/SQL Language Reference, 23c F46753-04, September 2023](https://docs.oracle.com/en/database/oracle/oracle-database/23/lnpls/)

Grammar changes and enhancements made in newer versions are not covered.
