## Introduction

This script lets you monitor one or more log files in an endless loop, _a la_ `tail -f`.  As lines are added to the files, they are compared to one or more patterns specified as Perl regular expressions.  And as matches are found, the script reacts by running a block of Perl code.  Thus, for example, you could use **log-guardian** to monitor web logs for problematic behaviour and add troublesome hosts to a blocklist and/or report the abuse dynamically.  You could even use it as a [port knocking service](https://slashdot.org/story/04/02/05/1834228/port-knocking-for-added-security)!

**log-guardian** is not a general-purpose log analyzer, though, since it looks at lines only as they are added to logs, not at whole log files themselves.  It's intended simply to allow you to react to issues as they arise.

By virtue of its use of the Perl module `File::Tail`, **log-guardian** offers a number of advantages.  For one, it's unaffected by log file rotation -- if a file does not appear to have input for a period of time, `File::Tail` will quietly re-open the file and continue reading.  For another, it does not spend excessive time checking log files with little or no traffic since `File::Tail` adjusts how frequently it polls for input based on past history.

You control what is monitored and how by modifying the scalar `$monitors`, either in the script itself or in a separate config file.  The scalar should be defined as a hash.  Each key is a log file to monitor while the value is an array of hashes specifying what to look for in the log file and how to react.  Keys in these secondary hashes should be either `label`, `pattern`, or `action`, corresponding to values representing respectively a descriptive label of the monitor, a Perl regular expression to use as a pattern, and an anonymous subroutine to be run if a match occurs. `pattern` is required while the other two key / value pairs are optional.  If `action` is not present, the default action in `$default_action` will be used instead.

When taking an action, **log-guardian** passes the subroutine several arguments: name of log, contents of line, value of `label`, value of `pattern`, and the results of matching the line against the pattern in an array contents (eg, `$1`, `$2`, etc).  These may be used however you want.

**log-guardian** is written in Perl.  It should work on any unix-like system with Perl 5.003 or later.  It also requires the following Perl modules:

* `Carp`
* `Getopt::Long`
* `File::Tail`
* `Safe`

If your system does not have these modules installed already, visit [CPAN](http://search.cpan.org/) for help.  Note that `File::Tail` must be at least version 0.90 and `Safe` at least version 2.0 (thus, it will not work with versions of Perl older than 5.003).  Note also that `File::Tail` is not included in the default Perl distribution so you may need to install it yourself.


## Installation

* Retrieve [the script](log-guardian) and save it locally.
* Verify ownership and permissions on the script - it will need to be invoked by root.
* Edit the script and set `$ENV{PATH}` and `$monitors` according to your environment. You may also wish to adjust the location of the perl interpreter in the first line as well as `$default_action`, `$max_interval`, and `$select_timeout` to suit your tastes.

## Use

If you have just a few logs to monitor, you may wish to run only one instance of **log-guardian**, starting it, say, when the system boots and putting it in the background.  Alternatively, if you have high volume services, you may find associating one instance with each of those services improves responsiveness and / or maintainability.  In this case, you might find it best to invoke **log-guardian** as part of each service's startup script.  Regardless, understand that the script will run in an endless loop and that, if you put it in the background, you will need to redirect output to a file somewhere in order to see it.

Examples:

| Invocation | Meaning |
| ---------- | ------- |
| `log-guardian` | monitors logfiles. |
| `log-guardian -d` | same as above but with copious debugging messages. |
| `log-guardian /etc/log-guardian/weblogs` | monitors logfiles using settings in `/etc/log-guardian/weblogs`. |


## Known Bugs and Caveats

Currently, I am not aware of any bugs in this script.

Understand that actions undertaken by **log-guardian** are arbitrary Perl code.  Be careful to control on one hand access to that code and on the other the content of that code.  And read [Daniel Cid's discussion of Attacking Log analysis tools](http://dcid.me/texts/attacking-log-analysis-tools.html) to understand some of the pitfalls that can arise.

If you encounter an error saying something like `Can't parse '_file_' - '_function_' trapped by operation mask`, you will need to adjust the list of operators permitted by `Safe`.  Look for the line with `$sandbox->permit_only` and refer to the manpage for the `Opcode` Perl module for possible operators.

You must include a pathname when specifying a separate configuration file; otherwise, it will be silently ignored.

When making changes to `$monitors`, you would be wise to redirect the script's output to a file for a period of time.  This will help diagnose problems in the patterns and / or actions.


## Copyright and License

Copyright (c) 2004-2020, George A. Theall.
All rights reserved.

This script is free software; you can redistribute it and/or modify it under the same terms as Perl itself.
