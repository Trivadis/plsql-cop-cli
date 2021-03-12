# Validator Limitations

These document covers the known limitations of the `com.trivadis.tvdcc.validators.TrivadisGuidelines3` validator that checks SQL*Plus files for compliance violations of the [Trivadis PL/SQL & SQL Coding Guidelines Version 4.0](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/).

## General

### File Scope

PL/SQL Cop loads a single file into an Eclipse Ecore model and validates it. After the validation the model is cleared and the next file is processed. This allows to process a large amount of files. Hence the scope of a validator check cannot be more than a single file.

### Oracle Data Dictionary Access

Accessing data dictionary views would allow to extend the scope of validations. However, PL/SQL Cop does not provide features to work with a `DataSource` or a `Connection`. Hence the provided validators are based on files only.

## Guidelines

### [G-1030](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/4-language-usage/1-general/g-1030/): Avoid defining variables that are not used.

This guideline is checked for CreateFunction, CreatePackageBody, CreateProcedure, CreateTrigger, CreateTypeBody and PlsqlUnit with a simplified scope. Variable declarations and usages are identified by name. The real PL/SQL scope is not honored. This might lead to false negatives. Here's an example:

```sql
CREATE OR REPLACE PROCEDURE p IS
   l_var INTEGER;     -- used
BEGIN
   <<myblock>>
   DECLARE
      l_var INTEGER;  -- not used
   BEGIN
      p.l_var := 1;
   END myblock;
   sys.dbms_output.put_line('l_var: ' || l_var);
END p;
/
```

The `l_var` variable defined within the `myblock` is never used. It's a violation of G-1030. However, a G-2170 (never overload variables) violation is thrown in this case.

### [G-1050](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/4-language-usage/1-general/g-1050/): Avoid using literals in your code.

It is very difficult to write code without violating this guideline. To make it a bit more feasible the numeric literals `0` and `1` are ignored. 

You might find use cases where it feels ridiculous introducing a constant for a literal. Then you have basically the following options:

1. Disable the check for this guideline completely
2. Disable checks for a particular line by adding a `NOSONAR` comment
3. Write a own validator check, overriding the default behavior (see [this example](https://github.com/Trivadis/plsql-cop-validators/blob/main/src/main/java/com/trivadis/tvdcc/validators/OverrideTrivadisGuidelines.xtend#L67) ignoring calls of the logger framework)

It's currently not possible to configure the behavior of this guideline check.

### [G-2410](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/4-language-usage/2-variables-and-types/4-boolean-data-types/g-2410/): Try to use boolean data type for values with dual meaning.

How are values with dual meaning identified?

When you are using only `Y` and `N` values for a PL/SQL variable, PL/SQL Cop reports a violation of G-2410. 

But if you are using other values such as `Da` and `Net`, you do not get such a message. The validator tries to minimize the number of false positives, which results sometimes in false negatives. Consider a variable containing a value for sex. Valid values are `male` and `female`. Even if this is a variable with dual meaning you do not want to represent the sex variable as a boolean type. So PL/SQL Cop checks if exactly two of the following values are stored in a variable: 

`TRUE`, `FALSE`, `T`, `F`, `0`, `1`, `2`, `YES`, `NO`, `Y`, `N`, `JA`, `NEIN`, `J`, `SI`, `S`, `OUI`, `NON`, `O`, `L_TRUE`, `L_FALSE`, `CO_TRUE`, `CO_FALSE`.

If other values are detected, no violation of G-2410 is reported.

This guideline is checked for CreateFunction, CreatePackageBody, CreateProcedure, CreateTrigger, CreateTypeBody and PlsqlUnit with a simplified scope. Variable declarations and usages are identified by name. The real PL/SQL scope is not honored. This might lead to false negatives. Here's an example:

```sql
CREATE PACKAGE BODY pkg IS
   PROCEDURE p1 IS
      l_var INTEGER;
   BEGIN
      l_var  := 1;
      l_var  := 2;
   END p1;
   PROCEDURE p2 IS
      l_var INTEGER;
   BEGIN
      l_var  := 0;
      l_var  := 1;
   END p2;
END pkg;
/
```

In this case for both `l_var` declarations a G-2410 violation should be thrown. Due to the simplified analysis scope three values are found for `l_var` and therefore no violation is thrown.


### [G-3160](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/4-language-usage/3-dml-and-sql/1-general/g-3160/): Avoid virtual columns to be visible.

This check is not yet implemented. 

Requires `CREATE TABLE` and `ALTER TABLE` parser support or access to the Oracle Data Dicionary.

### [G–3170](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/4-language-usage/3-dml-and-sql/1-general/g-3170/): Always use DEFAULT ON NULL declarations to assign default values to table columns if you refuse to store NULL values.

This check is not yet implemented. 

Requires `CREATE TABLE` and `ALTER TABLE` parser support or access to the Oracle Data Dicionary.

### [G-5010](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/4-language-usage/5-exception-handling/g-5010/): Try to use a error/logging framework for your application.

This check is not implemented.

Requires further definition regarding naming of the error/logging framework and its minimal use in PL/SQL code. However, could be implemented as a custom validator.

### [G-7140](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/4-language-usage/7-stored-objects/1-general/g-7140/): Always ensure that locally defined procedures or functions are referenced.

This guideline is checked for CreateFunction, CreatePackageBody, CreateProcedure, CreateTrigger, CreateTypeBody and PlsqlUnit with a simplified scope. Variable declarations and usages are identified by name. The real PL/SQL scope is not honored. This might lead to false negatives. Here's an example:

```sql
CREATE PACKAGE BODY pkg IS
   PROCEDURE p1 IS
      FUNCTION f RETURN NUMBER     -- unused
         DETERMINISTIC
      IS
      BEGIN
         RETURN 1;
      END f;
   BEGIN
      NULL;
   END p1;
   PROCEDURE p2 IS
      FUNCTION f RETURN NUMBER     -- used
         DETERMINISTIC
      IS
      BEGIN
         RETURN 1;
      END f;
   BEGIN
      sys.dbms_output.put_line(to_char(f()));
   END p2;
END pkg;
/
```

The first function `f` is not used. However, it is not reported due to the simplified analysis scope. Another function `f` is found in the scope and this function is used. 

### [G-7150](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/4-language-usage/7-stored-objects/1-general/g-7150/): Try to remove unused parameters.

This guideline is checked for CreateFunction, CreatePackageBody, CreateProcedure, CreateTrigger, CreateTypeBody and PlsqlUnit with a simplified scope. Variable declarations and usages are identified by name. The real PL/SQL scope is not honored. This might lead to false negatives. Here's an example:

```sql
CREATE PACKAGE BODY pkg IS
   PROCEDURE p (
      in_param IN DATE         -- unused
   ) IS
   BEGIN
      NULL;
   END p;
   PROCEDURE p (
      in_param IN INTEGER      -- used
   ) IS
   BEGIN
      sys.dbms_output.put_line(to_char(in_param));
   END p2;
END pkg;
/
```

In the first procedure `p` the parameter `in_param` is not used. However, it is not reported due to the simplified analysis scope. Another procedure `p` has also a parameter `in_param` that has been used. 

### [G-8410](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/4-language-usage/8-patterns/4-ensure-single-execution-at-a-time-of-a-program-unit/g-8410/): Always use application locks to ensure a program unit only running once at a given time.

This check is not implemented.

Algorithms to detect wrong, missing and right usages of this pattern are virtually impossible to implement without understanding the context of a certain PL/SQL code.

### [G-8510](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/4-language-usage/8-patterns/5-use-dbms-application-info-package-to-follow-progress-of-a-process/g-8510/): Always use dbms_application_info to track program process transiently.

This check is not implemented.

Algorithms to detect wrong, missing and right usages of this pattern are virtually impossible to implement without understanding the context of a certain PL/SQL code.