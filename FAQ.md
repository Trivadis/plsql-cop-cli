# Frequently Asked Questions

## I get the error "Could not reserve enough space for …KB object heap". What’s wrong?

You are probably using a 32-bit Java version and trying to configure more than 1GB of heap space, e.g. `-Xmx2048m`.

Define either `-Xmx1024m` or use a 64-bit Java version.

## I get the error "The system cannot find the path specified". What’s wrong?

Check if you have defined the correct `JAVA_HOME` environment variable in the .cmd/.sh file.

On Windows the `java.exe` is located in the `%JAVA_HOME%\bin` directory.

## What are the limitations?

See [parser limitations](parser-limitations.md) and [validator limitations](validator-limitations.md).

## What has changed in the latest version?

See the [release information](https://github.com/Trivadis/plsql-cop-cli/releases) for each version.

## What is the difference between "db\* CODECOP" and "db\* CODECOP for SQL Developer"?

*db\* CODECOP* is a standalone command line utility. It processes SQL*Plus scripts within a directory tree and produces an HTML report and an Excel workbook. It reports metrics on file and PL/SQL unit level as well as violations of the [Trivadis PL/SQL & SQL Coding Guidelines Version 4.0](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/).

*db\* CODECOP for SQL Developer* is a cost-free extension to Oracle’s SQL Developer. It processes the content of an editor window and produces a HTML report which is similar to the one of db\* CODECOP but reduced and adapted to the scope of a single editor window. Additionally the violations of the [Trivadis PL/SQL & SQL Coding Guidelines Version 4.0](https://trivadis.github.io/plsql-and-sql-coding-guidelines/v4.0/) are presented in a dedicated tab pane linked to the editor window to support quick navigation to the corresponding code.

## What are the licensing terms?

The preview/trial version of db\* CODECOP is licensed under the Creative Commons Attribution-NonCommercial-NoDerivs 3.0 Unported License. You may obtain a copy of the License at https://creativecommons.org/licenses/by-nc-nd/3.0/.

![CC-BY_NC-ND](images/CC-BY-NC-ND.png)

The trial/preview version provides full functionality but is limited in time and volume.

For production use a separate software license agreement is required.
