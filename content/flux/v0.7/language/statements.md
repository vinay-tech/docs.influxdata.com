---
title: Statements
description:
menu:
  flux_0_7:
    parent: Flux language
    name: Statements
    weight:
---
A statement controls execution.

    Statement = OptionStatement | VarAssignment |
                ReturnStatement | ExpressionStatement | BlockStatment .

#### Package statement

    PackageStatement = "package" identifier .

A package statement defines a package block.
Package names must be valid Flux identifiers.
The package statement must be the first statement of every Flux source file.
If a file does not declare a package statement, all identifiers in that file will belong to the special _main_ package.

[IMPL#247](https://github.com/influxdata/platform/issues/247) Add package/namespace support

##### package main

The _main_ package is special for a few reasons:

1. It defines the entrypoint of a Flux program
2. It cannot be imported
3. All query specifications produced after evaluating the _main_ package are coerced into producing side effects

#### Import statement

    ImportStatement = "import" [identifier] `"` unicode_char { unicode_char } `"`.

Associated with every package is a package name and an import path.
The import statement takes a package's import path and brings all of the identifiers defined in that package into the current scope.
The import statment defines a namespace through which to access the imported identifiers.
By default the identifer of this namespace is the package name unless otherwise specified.
For example, given a variable `x` declared in package `foo`, importing `foo` and referencing `x` would look like this:

```
import "import/path/to/package/foo"

foo.x
```

Or this:

```
import bar "import/path/to/package/foo"

bar.x
```

A package's import path is always absolute.
Flux does not support relative imports.
Assigment into the namespace of an imported package is not allowed.
A package cannot access nor modify the identifiers belonging to the imported packages of its imported packages.
Every statement contained in an imported package is evaluated.

#### Option statements

Options specify a context in which a Flux query is to be run. They define variables
that describe how to execute a Flux query. For example, the following Flux script sets
the `task` option to schedule a query to run periodically every hour:

    option task = {
        name: "mean",
        every: 1h,
    }

    from(bucket:"metrics/autogen")
        |> range(start:-task.every)
        |> group(by:["level"])
        |> mean()
        |> yield(name:"mean")

All options are designed to be completely optional and have default values to be used when not specified.
Grammatically, an option statement is just a variable assignment preceded by the "option" keyword.

    OptionStatement = "option" VarAssignment

Below is a list of all options that are currently implemented in the Flux language:

* now
* task
* location

##### now

The `now` option is a function that returns a time value to be used as a proxy for the current system time.

    // Query should execute as if the below time is the current system time
    option now = () => 2006-01-02T15:04:05-07:00

##### task

The `task` option is used by a scheduler to schedule the execution of a Flux query.

    option task = {
        name: "foo",        // name is required
        every: 1h,          // task should be run at this interval
        delay: 10m,         // delay scheduling this task by this duration
        cron: "0 2 * * *",  // cron is a more sophisticated way to schedule. every and cron are mutually exclusive
        retry: 5,           // number of times to retry a failed query
    }

##### location

The `location` option is used to set the default time zone of all times in the script.
The location maps the UTC offset in use at that location for a given time.
The default value is set using the time zone of the running process.

    option location = fixedZone(offset:-5h) // set timezone to be 5 hours west of UTC
    option location = loadLocation(name:"America/Denver") // set location to be America/Denver

[IMPL#660](https://github.com/influxdata/platform/issues/660) Implement Location option

#### Return statements

A terminating statement prevents execution of all statements that appear after it in the same block.
A return statement is a terminating statement.

    ReturnStatement = "return" Expression .

#### Expression statements

An expression statement is an expression where the computed value is discarded.

    ExpressionStatement = Expression .

Examples:

    1 + 1
    f()
    a