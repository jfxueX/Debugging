Next: [Commands](Commands.html#Commands), Previous: [Sample Session](Sample-Session.html#Sample-Session), Up: [Top](index.html#Top)   [[Contents](index.html#SEC_Contents)][[Index](Concept-Index.html#Concept-Index)]

---

# 2 Getting In and Out of GDB

This chapter discusses how to start GDB, and how to get out of it.
The essentials are:

-  type &lsquo;gdb&rsquo; to start GDB.

-  type quit or Ctrl-d to exit.


* [2.1 Invoking GDB](#21-invoking-gdb)
    * [2.1.1 Choosing Files](#211-choosing-files)
    * [2.1.2 Choosing Modes](#212-choosing-modes)
    * [2.1.3 What GDB Does During Startup](#213-what-gdb-does-during-startup)
    * [Footnotes](#footnotes)
* [2.2 Quitting GDB](#22-quitting-gdb)
* [2.3 Shell Commands](#23-shell-commands)
* [2.4 Logging Output](#24-logging-output)

## 2.1 Invoking GDB

Invoke GDB by running the program `gdb`.  Once started,
GDB reads commands from the terminal until you tell it to exit.

You can also run `gdb` with a variety of arguments and options,
to specify more of your debugging environment at the outset.

The command-line options described here are designed
to cover a variety of situations; in some environments, some of these
options may effectively be unavailable.

The most usual way to start GDB is with one argument,
specifying an executable program:

    gdb program

You can also start with both an executable program and a core file
specified:

    gdb programcore

You can, instead, specify a process ID as a second argument, if you want
to debug a running process:

    gdb program 1234
    

would attach GDB to process `1234` (unless you also have a file
named 1234; GDB does check for a core file first).

Taking advantage of the second command-line argument requires a fairly
complete operating system; when you use GDB as a remote
debugger attached to a bare board, there may not be any notion of
&ldquo;process&rdquo;, and there is often no way to get a core dump.  GDB
will warn you if it is unable to attach or to read core dumps.

You can optionally have `gdb` pass any arguments after the
executable file to the inferior using `--args`.  This option stops
option processing.

    gdb --args gcc -O2 -c foo.c
    

This will cause `gdb` to debug `gcc`, and to set
`gcc`&rsquo;s command-line arguments (see [Arguments](Arguments.html#Arguments)) to &lsquo;-O2 -c foo.c&rsquo;.

You can run `gdb` without printing the front material, which describes
GDB&rsquo;s non-warranty, by specifying `--silent`
(or `-q`/`--quiet`):

    gdb --silent
    

You can further control how GDB starts up by using command-line
options.  GDB itself can remind you of the options available.

Type

    gdb -help
    

to display all available options and briefly describe their use
(&lsquo;gdb -h&rsquo; is a shorter equivalent).

All options and command line arguments you give are processed
in sequential order.  The order makes a difference when the
&lsquo;-x&rsquo; option is used.

#### 2.1.1 Choosing Files

When GDB starts, it reads any arguments other than options as
specifying an executable file and core file (or process ID).  This is
the same as if the arguments were specified by the &lsquo;-se&rsquo; and
&lsquo;-c&rsquo; (or &lsquo;-p&rsquo;) options respectively.  (GDB reads the
first argument that does not have an associated option flag as
equivalent to the &lsquo;-se&rsquo; option followed by that argument; and the
second argument that does not have an associated option flag, if any, as
equivalent to the &lsquo;-c&rsquo;/&lsquo;-p&rsquo; option followed by that argument.)
If the second argument begins with a decimal digit, GDB will
first attempt to attach to it as a process, and if that fails, attempt
to open it as a corefile.  If you have a corefile whose name begins with
a digit, you can prevent GDB from treating it as a pid by
prefixing it with ./, e.g. ./12345.

If GDB has not been configured to included core file support,
such as for most embedded targets, then it will complain about a second
argument and ignore it.

Many options have both long and short forms; both are shown in the
following list.  GDB also recognizes the long forms if you truncate
them, so long as enough of the option is present to be unambiguous.
(If you prefer, you can flag option arguments with &lsquo;--&rsquo; rather
than &lsquo;-&rsquo;, though we illustrate the more usual convention.)

`-symbols file``-s file`
Read symbol table from file file.

`-exec file``-e file`
Use file file as the executable file to execute when appropriate,
and for examining pure data in conjunction with a core dump.

`-se file`
Read symbol table from file file and use it as the executable
file.

`-core file``-c file`
Use file file as a core dump to examine.

`-pid number``-p number`
Connect to process ID number, as with the `attach` command.

`-command file``-x file`
Execute commands from file file.  The contents of this file is
evaluated exactly as the `source` command would.
See [Command files](Command-Files.html#Command-Files).

`-eval-command command``-ex command`
Execute a single GDB command.

This option may be used multiple times to call multiple commands.  It may
also be interleaved with &lsquo;-command&rsquo; as required.

    gdb -ex 'target sim' -ex 'load' \
       -x setbreakpoints -ex 'run' a.out
    

`-init-command file``-ix file`
Execute commands from file file before loading the inferior (but
after loading gdbinit files).
See [Startup](Startup.html#Startup).

`-init-eval-command command``-iex command`
Execute a single GDB command before loading the inferior (but
after loading gdbinit files).
See [Startup](Startup.html#Startup).

`-directory directory``-d directory`
Add directory to the path to search for source and script files.

`-r``-readnow`
Read each symbol file&rsquo;s entire symbol table immediately, rather than
the default, which is to read it incrementally as it is needed.
This makes startup slower, but makes future operations faster.

`--readnever`
Do not read each symbol file&rsquo;s symbolic debug information.  This makes
startup faster but at the expense of not being able to perform
symbolic debugging.  DWARF unwind information is also not read,
meaning backtraces may become incomplete or inaccurate.  One use of
this is when a user simply wants to do the following sequence: attach,
dump core, detach.  Loading the debugging information in this case is
an unnecessary cause of delay.


### 2.1.2 Choosing Modes

You can run GDB in various alternative modes&mdash;for example, in
batch mode or quiet mode.

`-nx``-n`
Do not execute commands found in any initialization file.
There are three init files, loaded in the following order:

`system.gdbinit`
This is the system-wide init file.
Its location is specified with the `--with-system-gdbinit`
configure option (see [System-wide configuration](System_002dwide-configuration.html#System_002dwide-configuration)).
It is loaded first when GDB starts, before command line options
have been processed.

`~/.gdbinit`
This is the init file in your home directory.
It is loaded next, after system.gdbinit, and before
command options have been processed.

`./.gdbinit`
This is the init file in the current directory.
It is loaded last, after command line options other than `-x` and
`-ex` have been processed.  Command line options `-x` and
`-ex` are processed last, after ./.gdbinit has been loaded.

For further documentation on startup processing, See [Startup](Startup.html#Startup).
For documentation on how to write command files,
See [Command Files](Command-Files.html#Command-Files).

`-nh`
Do not execute commands found in ~/.gdbinit, the init file
in your home directory.
See [Startup](Startup.html#Startup).

`-quiet``-silent``-q`
&ldquo;Quiet&rdquo;.  Do not print the introductory and copyright messages.  These
messages are also suppressed in batch mode.

`-batch`
Run in batch mode.  Exit with status `0` after processing all the
command files specified with &lsquo;-x&rsquo; (and all commands from
initialization files, if not inhibited with &lsquo;-n&rsquo;).  Exit with
nonzero status if an error occurs in executing the GDB commands
in the command files.  Batch mode also disables pagination, sets unlimited
terminal width and height see [Screen Size](Screen-Size.html#Screen-Size), and acts as if set confirm
off were in effect (see [Messages/Warnings](Messages_002fWarnings.html#Messages_002fWarnings)).

Batch mode may be useful for running GDB as a filter, for
example to download and run a program on another computer; in order to
make this more useful, the message

    Program exited normally.
    

(which is ordinarily issued whenever a program running under
GDB control terminates) is not issued when running in batch
mode.

`-batch-silent`
Run in batch mode exactly like &lsquo;-batch&rsquo;, but totally silently.  All
GDB output to `stdout` is prevented (`stderr` is
unaffected).  This is much quieter than &lsquo;-silent&rsquo; and would be useless
for an interactive session.

This is particularly useful when using targets that give &lsquo;Loading section&rsquo;
messages, for example.

Note that targets that give their output via GDB, as opposed to
writing directly to `stdout`, will also be made silent.

`-return-child-result`
The return code from GDB will be the return code from the child
process (the process being debugged), with the following exceptions:

- GDB exits abnormally.  E.g., due to an incorrect argument or an
internal error.  In this case the exit code is the same as it would have been
without &lsquo;-return-child-result&rsquo;.

-  The user quits with an explicit value.  E.g., &lsquo;quit 1&rsquo;.

-  The child process never runs, or is not allowed to terminate, in which case
the exit code will be -1.

This option is useful in conjunction with &lsquo;-batch&rsquo; or &lsquo;-batch-silent&rsquo;,
when GDB is being used as a remote program loader or simulator
interface.

`-nowindows``-nw`
&ldquo;No windows&rdquo;.  If GDB comes with a graphical user interface
(GUI) built in, then this option tells GDB to only use the command-line
interface.  If no GUI is available, this option has no effect.

`-windows``-w`
If GDB includes a GUI, then this option requires it to be
used if possible.

`-cd directory`
Run GDB using directory as its working directory,
instead of the current directory.

`-data-directory directory``-D directory`
Run GDB using directory as its data directory.
The data directory is where GDB searches for its
auxiliary files.  See [Data Files](Data-Files.html#Data-Files).

`-fullname``-f`
GNU Emacs sets this option when it runs GDB as a
subprocess.  It tells GDB to output the full file name and line
number in a standard, recognizable fashion each time a stack frame is
displayed (which includes each time your program stops).  This
recognizable format looks like two &lsquo;\032&rsquo; characters, followed by
the file name, line number and character position separated by colons,
and a newline.  The Emacs-to-GDB interface program uses the two
&lsquo;\032&rsquo; characters as a signal to display the source code for the
frame.

`-annotate level`
This option sets the *annotation level* inside GDB.  Its
effect is identical to using &lsquo;set annotate level&rsquo;
(see [Annotations](Annotations.html#Annotations)).  The annotation level controls how much
information GDB prints together with its prompt, values of
expressions, source lines, and other types of output.  Level 0 is the
normal, level 1 is for use when GDB is run as a subprocess of
GNU Emacs, level 3 is the maximum annotation suitable for programs
that control GDB, and level 2 has been deprecated.

The annotation mechanism has largely been superseded by GDB/MI
(see [GDB/MI](GDB_002fMI.html#GDB_002fMI)).

`--args`
Change interpretation of command line so that arguments following the
executable file are passed as command line arguments to the inferior.
This option stops option processing.

`-baud bps``-b bps`
Set the line speed (baud rate or bits per second) of any serial
interface used by GDB for remote debugging.

`-l timeout`
Set the timeout (in seconds) of any communication used by GDB
for remote debugging.

`-tty device``-t device`
Run using device for your program&rsquo;s standard input and output.

`-tui`
Activate the *Text User Interface* when starting.  The Text User
Interface manages several text windows on the terminal, showing
source, assembly, registers and GDB command outputs
(see [GDB Text User Interface](TUI.html#TUI)).  Do not use this
option if you run GDB from Emacs (see [Using GDB under GNU Emacs](Emacs.html#Emacs)).

`-interpreter interp`
Use the interpreter interp for interface with the controlling
program or device.  This option is meant to be set by programs which
communicate with GDB using it as a back end.
See [Command Interpreters](Interpreters.html#Interpreters).

&lsquo;--interpreter=mi&rsquo; (or &lsquo;--interpreter=mi2&rsquo;) causes
GDB to use the *GDB/MI interface* (see [The GDB/MI Interface](GDB_002fMI.html#GDB_002fMI)) included since GDB version 6.0.  The
previous GDB/MI interface, included in GDB version 5.3 and
selected with &lsquo;--interpreter=mi1&rsquo;, is deprecated.  Earlier
GDB/MI interfaces are no longer supported.

`-write`
Open the executable and core files for both reading and writing.  This
is equivalent to the &lsquo;set write on&rsquo; command inside GDB
(see [Patching](Patching.html#Patching)).

`-statistics`
This option causes GDB to print statistics about time and
memory usage after it completes each command and returns to the prompt.

`-version`
This option causes GDB to print its version number and
no-warranty blurb, and exit.

`-configuration`
This option causes GDB to print details about its build-time
configuration parameters, and then exit.  These details can be
important when reporting GDB bugs (see [GDB Bugs](GDB-Bugs.html#GDB-Bugs)).


### 2.1.3 What GDB Does During Startup

Here&rsquo;s the description of what GDB does during session startup:

1.  Sets up the command interpreter as specified by the command line
(see [interpreter](Mode-Options.html#Mode-Options)).

2.  Reads the system-wide *init file* (if --with-system-gdbinit was
used when building GDB; see [System-wide configuration and settings](System_002dwide-configuration.html#System_002dwide-configuration)) and executes all the commands in
that file.

3.  Reads the init file (if any) in your home directory[1](#FOOT1) and executes all the commands in
that file.

4.  Executes commands and command files specified by the &lsquo;-iex&rsquo; and
&lsquo;-ix&rsquo; options in their specified order.  Usually you should use the
&lsquo;-ex&rsquo; and &lsquo;-x&rsquo; options instead, but this way you can apply
settings before GDB init files get executed and before inferior
gets loaded.

5.  Processes command line options and operands.

6.  Reads and executes the commands from init file (if any) in the current
working directory as long as &lsquo;set auto-load local-gdbinit&rsquo; is set to
&lsquo;on&rsquo; (see [Init File in the Current Directory](Init-File-in-the-Current-Directory.html#Init-File-in-the-Current-Directory)).
This is only done if the current directory is
different from your home directory.  Thus, you can have more than one
init file, one generic in your home directory, and another, specific
to the program you are debugging, in the directory where you invoke
GDB.

7.  If the command line specified a program to debug, or a process to
attach to, or a core file, GDB loads any auto-loaded
scripts provided for the program or for its loaded shared libraries.
See [Auto-loading](Auto_002dloading.html#Auto_002dloading).

If you wish to disable the auto-loading during startup,
you must do something like the following:

    $ gdb -iex "set auto-load python-scripts off" myprogram
    

Option &lsquo;-ex&rsquo; does not work because the auto-loading is then turned
off too late.

8.  Executes commands and command files specified by the &lsquo;-ex&rsquo; and
&lsquo;-x&rsquo; options in their specified order.  See [Command Files](Command-Files.html#Command-Files), for
more details about GDB command files.

9.  Reads the command history recorded in the *history file*.
See [Command History](Command-History.html#Command-History), for more details about the command history and the
files where GDB records it.

Init files use the same syntax as *command files* (see [Command Files](Command-Files.html#Command-Files)) and are processed by GDB in the same way.  The init
file in your home directory can set options (such as &lsquo;set
complaints&rsquo;) that affect subsequent processing of command line options
and operands.  Init files are not executed if you use the &lsquo;-nx&rsquo;
option (see [Choosing Modes](Mode-Options.html#Mode-Options)).

To display the list of init files loaded by gdb at startup, you
can use gdb --help.

The GDB init files are normally called .gdbinit.
The DJGPP port of GDB uses the name gdb.ini, due to
the limitations of file names imposed by DOS filesystems.  The Windows
port of GDB uses the standard name, but if it finds a
gdb.ini file in your home directory, it warns you about that
and suggests to rename the file to the standard name.

---

#### Footnotes

### [(1)](#DOCF1)

On
DOS/Windows systems, the home directory is the one pointed to by the
`HOME` environment variable.


## 2.2 Quitting GDB

`quit [expression]``q`
To exit GDB, use the `quit` command (abbreviated
`q`), or type an end-of-file character (usually Ctrl-d).  If you
do not supply expression, GDB will terminate normally;
otherwise it will terminate using the result of expression as the
error code.

An interrupt (often Ctrl-c) does not exit from GDB, but rather
terminates the action of any GDB command that is in progress and
returns to GDB command level.  It is safe to type the interrupt
character at any time because GDB does not allow it to take effect
until a time when it is safe.

If you have been using GDB to control an attached process or
device, you can release it with the `detach` command
(see [Debugging an Already-running Process](Attach.html#Attach)).


## 2.3 Shell Commands

If you need to execute occasional shell commands during your
debugging session, there is no need to leave or suspend GDB; you can
just use the `shell` command.

`shell command-string``!command-string`
Invoke a standard shell to execute command-string.
Note that no space is needed between `!` and command-string.
If it exists, the environment variable `SHELL` determines which
shell to run.  Otherwise GDB uses the default shell
(/bin/sh on Unix systems, COMMAND.COM on MS-DOS, etc.).

The utility `make` is often needed in development environments.
You do not have to use the `shell` command for this purpose in
GDB:

`make make-args`
Execute the `make` program with the specified
arguments.  This is equivalent to &lsquo;shell make make-args&rsquo;.


## 2.4 Logging Output

You may want to save the output of GDB commands to a file.
There are several commands to control GDB&rsquo;s logging.

`set logging on`
Enable logging.

`set logging off`
Disable logging.

`set logging file file`
Change the name of the current logfile.  The default logfile is gdb.txt.

`set logging overwrite [on|off]`
By default, GDB will append to the logfile.  Set `overwrite` if
you want `set logging on` to overwrite the logfile instead.

`set logging redirect [on|off]`
By default, GDB output will go to both the terminal and the logfile.
Set `redirect` if you want output to go only to the log file.

`show logging`
Show the current values of the logging settings.



