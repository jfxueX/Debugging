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

&bull; [File Options](File-Options.html#File-Options):  Choosing files
&bull; [Mode Options](Mode-Options.html#Mode-Options):  Choosing modes
&bull; [Startup](Startup.html#Startup):  What GDB does during startup
