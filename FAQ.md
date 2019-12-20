# Frequently Asked Questions

## Table of Contents
- [1. How are values with dual meaning identified?](#1-how-are-values-with-dual-meaning-identified)
- [2. I get the error "Could not reserve enough space for …KB object heap". What’s wrong?](#2-i-get-the-error-could-not-reserve-enough-space-for-kb-object-heap-whats-wrong)
- [3. I get the error "The system cannot find the path specified". What’s wrong?](#3-i-get-the-error-the-system-cannot-find-the-path-specified-whats-wrong)
- [4. What are the limitations?](#4-what-are-the-limitations)
- [5. What has changed in the latest version?](#5-what-has-changed-in-the-latest-version)
- [6. What is the difference between "PL/SQL Cop" and "PL/SQL Cop for SQL Developer"?](#6-what-is-the-difference-between-plsql-cop-and-plsql-cop-for-sql-developer)
- [7. What are the licensing terms?](#7-what-are-the-licensing-terms)

## 1. How are values with dual meaning identified?

When you are using only `Y` and `N` values for a PL/SQL variable, PL/SQL Cop produces the following message: 

[G-2410](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v3.6/4-language-usage/2-variables-and-types/4-boolean-data-types/g-2410/): Try to use boolean data type for values with dual meaning.

But if you are using other values such as `Da` and `Net`, you do not get such a message. PL/SQL Cop tries to minimize the number of false positives, which results sometimes in false negatives. Consider a variable containing a value for sex. Valid values are `male` and `female`. Even if this is a variable with dual meaning you do not want to represent the sex variable as a boolean type. So PL/SQL Cop checks if exactly two of the following values are stored in a variable: 

`TRUE`, `FALSE`, `T`, `F`, `0`, `1`, `2`, `YES`, `NO`, `Y`, `N`, `JA`, `NEIN`, `J`, `SI`, `S`, `OUI`, `NON`, `O`, `L_TRUE`, `L_FALSE`, `CO_TRUE`, `CO_FALSE`.

If other values are detected, the message [G-2410](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v3.6/4-language-usage/2-variables-and-types/4-boolean-data-types/g-2410/) will not be shown.

## 2. I get the error "Could not reserve enough space for …KB object heap". What’s wrong?

You are probably using a 32-bit Java version and trying to configure more than 1GB of heap space, e.g. `-Xmx2048m`.

Define either `-Xmx1024m` or use a 64-bit Java version.

## 3. I get the error "The system cannot find the path specified". What’s wrong?

Check if you have defined the correct `JAVA_HOME` environment variable in the .cmd/.sh file.

On Windows the `java.exe` is located in the `%JAVA_HOME%\bin` directory.

## 4. What are the limitations?

### 4.1 Maxim

If your SQL*Plus script runs successfully against an Oracle database but PL/SQL Cop or PL/SQL Analyzer reports an error, this is usually considered a bug. However, there are some known exceptions to this basic principle, which are documented below.

### 4.2 Block Terminator

Other block terminators than a dot (`.`) are not supported. This means the `SET BLOCKTERMINATOR` command is ignored.

### 4.3 Command Separator

Other command separators than semicolon (`;`) are not supported. This means that the `SET CMDSEP` command is ignored.

### 4.4 SQL Terminator

Other SQL terminators than semicolon (`;`) are not supported. This means that the `SET SQLTERMINATOR` command is ignored. Tailing whitespaces after a SQL terminator are not supported.

### 4.5 Line Continuation Character

Tailing whitespaces after a line continuation character (`-`) are not supported.

### 4.6 Slash Command

Tailing whitespaces after the slash command (`/`) are not supported.

### 4.7 Execute Command

The execute command must end on semicolon (`;`) if the last token is an expression.

### 4.8 Remark Command

The remark must not contain unterminated single (`'`) or double quotes (`"`).

### 4.9 Prompt Command

The prompt must not contain unterminated single (`'`) or double quotes (`"`).

### 4.10 Use of Keywords

The use of PL/SQL and SQL keywords as unquoted identifiers are generally not supported, due to the fact, that every single keyword needs to be treated as an exception. 

Oracle is quite gracious in that area and therefore we are eager to support more and more keywords as unquoted identifiers with every release, but the following keywords are causing conflicts in certain parts of the grammar and the use as literals should be therefore avoided: 

`CROSS`, `CURSOR`, `END`, `FULL`, `FUNCTION`, `INNER`, `JOIN`, `LEFT`, `MODEL`, `OUTER`, `RIGHT`, `ROWTYPE`, `TYPE`.

### 4.11 Quote Delimiter Characters

Oracle supports various quote literal characters except space, tab and return. The following example uses the `[]` quote character pair:

```
nq'[that’s very cool!]'
```

The following quote characters-pairs are supported: 

`$$`, `##`, `@@`, `££`, `""`, `||`, `()`, `{}`, `[]`, `<>`, `!!`, `++`, `~~`, `//`. 

All other quote characters lead to parse errors.

### 4.12 Conditional Compilation

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

### 4.13 Error Logging Clause
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

### 4.14 PL/SQL Source Text Wrapping

Since PL/SQL Cop and PL/SQL Analyzer do not include a PL/SQL unwrap utility, the use of wrapped PL/SQL code is not supported.

### 4.15 Supported Oracle Versions

The PL/SQL and SQL grammars from Oracle version 7.0 until version 12.2 are supported. The language is based on the following documentation:

- Oracle SQL*Plus User’s Guide and Reference, 12c Release 2 (12.2), E50028-08, January 2017
- Oracle SQL Language Reference, 12c Release 2 (12.2), E49448-12, January 2017
- Oracle PL/SQL Language Reference, 12c Release 2 (12.2), E49633-15, January 2017

Grammar changes and enhancements made in newer versions are not yet covered.

## 5. What has changed in the latest version?

See the [release information](https://github.com/Trivadis/plsql-cop-cli/releases) for each version.

## 6. What is the difference between "PL/SQL Cop" and "PL/SQL Cop for SQL Developer"?

*PL/SQL Cop* is a standalone command line utility. It processes SQL*Plus scripts within a directory tree and produces an HTML report and an Excel workbook. It reports metrics on file and PL/SQL unit level as well as violations of the [Trivadis PL/SQL & SQL Coding Guidelines Version 3.6](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v3.6/).

*PL/SQL Cop for SQL Developer* is a cost-free extension to Oracle’s SQL Developer. It processes the content of an editor window and produces a HTML report which is similar to the one of PL/SQL Cop but reduced and adapted to the scope of a single editor window. Additionally the violations of the [Trivadis PL/SQL & SQL Coding Guidelines Version 3.6](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v3.6/) are presented in a dedicated tab pane linked to the editor window to support quick navigation to the corresponding code.

## 7. What are the licensing terms?

The preview/trial version of PL/SQL Cop is licensed under the Creative Commons Attribution-NonCommercial-NoDerivs 3.0 Unported License. You may obtain a copy of the License at https://creativecommons.org/licenses/by-nc-nd/3.0/.

![CC-BY_NC-ND](images/CC-BY-NC-ND.png)

The trial/preview version provides full functionality but is limited in time and volume.

The commercial version of PL/SQL Cop requires a separate software license agreement.
