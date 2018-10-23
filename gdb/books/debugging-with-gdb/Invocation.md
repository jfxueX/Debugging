Next: [Commands](Commands.html#Commands), Previous: [Sample Session](Sample-Session.html#Sample-Session), Up: [Top](index.html#Top)   [[Contents](index.html#SEC_Contents)][[Index](Concept-Index.html#Concept-Index)]

---

## 2 Getting In and Out of GDB

This chapter discusses how to start GDB, and how to get out of it.
The essentials are:

-  type &lsquo;gdb&rsquo; to start GDB.

-  type quit or Ctrl-d to exit.

### 2.1 Invoking GDB

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
