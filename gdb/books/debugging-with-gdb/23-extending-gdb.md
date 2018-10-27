# 23 Extending GDB


GDB provides several mechanisms for extension. GDB also provides the ability to 
automatically load extensions when it reads a file for debugging. This allows 
the user to automatically customize GDB for the program being debugged.


* [23.1 Canned Sequences of Commands](#231-canned-sequences-of-commands)
    * [23.1.1 User-defined Commands](#2311-user-defined-commands)
    * [23.1.2 User-defined Command Hooks](#2312-user-defined-command-hooks)
    * [23.1.3 Command Files](#2313-command-files)
    * [23.1.4 Commands for Controlled Output](#2314-commands-for-controlled-output)
    * [23.1.5 Controlling auto-loading native GDB scripts](#2315-controlling-auto-loading-native-gdb-scripts)
* [23.2 Extending GDB using Python](#232-extending-gdb-using-python)
    * [23.2.1 Python Commands](#2321-python-commands)
    * [23.2.2 Python API](#2322-python-api)
    * [23.2.3 Python Auto-loading](#2323-python-auto-loading)
    * [23.2.4 Python modules](#2324-python-modules)
        * [23.2.4.1 gdb.printing](#23241-gdbprinting)
        * [23.2.4.2 gdb.types](#23242-gdbtypes)
        * [23.2.4.3 gdb.prompt](#23243-gdbprompt)
* [23.3 Extending GDB using Guile](#233-extending-gdb-using-guile)
    * [23.3.1 Guile Introduction](#2331-guile-introduction)
    * [23.3.2 Guile Commands](#2332-guile-commands)
    * [23.3.3 Guile API](#2333-guile-api)
    * [23.3.4 Guile Auto-loading](#2334-guile-auto-loading)
    * [23.3.5 Guile Modules](#2335-guile-modules)
        * [23.3.5.1 Guile Printing Module](#23351-guile-printing-module)
        * [23.3.5.2 Guile Types Module](#23352-guile-types-module)
* [23.4 Auto-loading extensions](#234-auto-loading-extensions)
    * [23.4.1 The objfile-gdb.ext file](#2341-the-objfile-gdbext-file)
    * [23.4.2 The `.debug_gdb_scripts` section](#2342-the-debug_gdb_scripts-section)
        * [23.4.2.1 Script File Entries](#23421-script-file-entries)
        * [23.4.2.2 Script Text Entries](#23422-script-text-entries)
    * [23.4.3 Which flavor to choose?](#2343-which-flavor-to-choose)
* [23.5 Multiple Extension Languages](#235-multiple-extension-languages)
    * [23.5.1 Python comes first](#2351-python-comes-first)
* [23.6 Creating new spellings of existing commands](#236-creating-new-spellings-of-existing-commands)


To facilitate the use of extension languages, GDB is capable of evaluating the 
contents of a file. When doing so, GDB can recognize which extension language is 
being used by looking at the filename extension. Files with an unrecognized 
filename extension are always treated as a GDB Command Files. See [Command files](Command-Files.html#Command-Files).

You can control how GDB evaluates these files with the following setting:

- `set script-extension off`

   All scripts are always evaluated as GDB Command Files.

- `set script-extension soft`

   The debugger determines the scripting language based on filename extension.

   If this scripting language is supported, GDB evaluates the script using that 
   language. Otherwise, it evaluates the file as a GDB Command File.

- `set script-extension strict`

   The debugger determines the scripting language based on filename extension, and 
   evaluates the script using that language. If the language is not supported, then 
   the evaluation fails.

- `show script-extension`

   Display the current value of the `script-extension` option.


## 23.1 Canned Sequences of Commands

Aside from breakpoint commands (see [Breakpoint Command Lists](Break-
Commands.html#Break-Commands)), GDB provides two ways to store sequences of 
commands for execution as a unit: user-defined commands and command files.


### 23.1.1 User-defined Commands

A *user-defined command* is a sequence of GDB commands to which you assign a new 
name as a command. This is done with the `define` command. User commands may 
accept an unlimited number of arguments separated by whitespace. Arguments are 
accessed within the user command via `$arg0…$argN`. A trivial example:

```gdb
define adder
  print $arg0 + $arg1 + $arg2
end
```

To execute the command use:

```gdb
adder 1 2 3
```

This defines the command `adder`, which prints the sum of its three arguments. 
Note the arguments are text substitutions, so they may reference variables, use 
complex expressions, or even perform inferior functions calls.

In addition, `$argc` may be used to find out how many arguments have been passed.

```gdb
define adder
  if $argc == 2
    print $arg0 + $arg1
  end
  if $argc == 3
    print $arg0 + $arg1 + $arg2
  end
end
```

Combining with the `eval` command (see [eval](Output.html#eval)) makes it easier 
to process a variable number of arguments:

```gdb
define adder
  set $i = 0
  set $sum = 0
  while $i < $argc
    eval "set $sum = $sum + $arg%d", $i
    set $i = $i + 1
  end
  print $sum
end
```

- `define commandname`  
   Define a command named commandname.

   If there is already a command by that name, you are asked to confirm that you 
   want to redefine it. The argument commandname may be a bare command name 
   consisting of letters, numbers, dashes, and underscores. It may also start with 
   any predefined prefix command. For example, ‘define target my-target’ creates a 
   user-defined ‘target my-target’ command.

   The definition of the command is made up of other GDB command lines, which are 
   given following the `define` command. The end of these commands is marked by a 
   line containing `end`.

- `document commandname`  
   Document the user-defined command commandname, so that it can be accessed by 
   `help`.

   The command commandname must already be defined. This command reads 
   lines of documentation just as `define` reads the lines of the command 
   definition, ending with `end`. After the `document` command is finished, `help` 
   on command commandname displays the documentation you have written.

   You may use the `document` command again to change the documentation of a 
   command. Redefining the command with `define` does not change the documentation.

- `dont-repeat`  
   Used inside a user-defined command, this tells GDB that this command should not 
   be repeated when the user hits `RET` (see [repeat last command](Command-Syntax.html#Command-Syntax)).

- `help user-defined`  
   List all user-defined commands and all python commands defined in class 
   `COMAND_USER`. 

   The first line of the documentation or docstring is included (if any).

- `show user`  
  `show user commandname`  
   Display the GDB commands used to define commandname (but not its documentation). 

   If no commandname is given, display the definitions for all user-defined 
   commands. This does not work for user-defined python commands.

- `show max-user-call-depth`  
  `set max-user-call-depth`  
   The value of `max-user-call-depth` controls how many recursion levels are 
   allowed in user-defined commands before GDB suspects an infinite recursion and 
   aborts the command. 

   This does not apply to user-defined python commands.

In addition to the above commands, user-defined commands frequently use control 
flow commands, described in [Command Files](Command-Files.html#Command-Files).

When user-defined commands are executed, the commands of the definition are not 
printed. An error in any command stops execution of the user-defined command.

If used interactively, commands that would ask for confirmation proceed without 
asking when used inside a user-defined command. Many GDB commands that normally 
print messages to say what they are doing omit the messages when used in a user-
defined command.


### 23.1.2 User-defined Command Hooks

You may define *hooks*, which are a special kind of user-defined command. 
Whenever you run the command ‘foo’, if the user-defined command ‘hook-foo’ 
exists, it is executed (with no arguments) before that command.

A hook may also be defined which is run after the command you executed. Whenever 
you run the command ‘foo’, if the user-defined command ‘hookpost-foo’ exists, it 
is executed (with no arguments) after that command. Post-execution hooks may 
exist simultaneously with pre-execution hooks, for the same command.

It is valid for a hook to call the command which it hooks. If this occurs, the 
hook is not re-executed, thereby avoiding infinite recursion.

In addition, a pseudo-command, ‘stop’ exists. Defining (‘hook-stop’) makes the 
associated commands execute every time execution stops in your program: before 
breakpoint commands are run, displays are printed, or the stack frame is 
printed.

For example, to ignore `SIGALRM` signals while single-stepping, but treat them 
normally during normal execution, you could define:

```gdb
define hook-stop
handle SIGALRM nopass
end

define hook-run
handle SIGALRM pass
end

define hook-continue
handle SIGALRM pass
end
```

As a further example, to hook at the beginning and end of the `echo` command, 
and to add extra text to the beginning and end of the message, you could define:

```gdb
define hook-echo
echo <<<---
end

define hookpost-echo
echo --->>>\n
end

(gdb) echo Hello World
<<<---Hello World--->>>
(gdb)
```

You can define a hook for any single-word command in GDB, but not for command 
aliases; you should define a hook for the basic command name, e.g. `backtrace` 
rather than `bt`. You can hook a multi-word command by adding `hook-` or 
`hookpost-` to the last word of the command, e.g. ‘define target hook-remote’ to 
add a hook to ‘target remote’.

If an error occurs during the execution of your hook, execution of GDB commands 
stops and GDB issues a prompt (before the command that you actually typed had a 
chance to run).

If you try to define a hook which does not match any known command, you get a 
warning from the `define` command.


### 23.1.3 Command Files

A command file for GDB is a text file made of lines that are GDB commands. 
Comments (lines starting with \#) may also be included. An empty line in a 
command file does nothing; it does not mean to repeat the last command, as it 
would from the terminal.

You can request the execution of a command file with the `source` command. Note 
that the `source` command is also used to evaluate scripts that are not Command 
Files. The exact behavior can be configured using the `script-extension` 
setting. See [Extending GDB](Extending-GDB.html#Extending-GDB).

`source [-s] [-v] filename`  

Execute the command file filename.

The lines in a command file are generally executed sequentially, unless the 
order of execution is changed by one of the *flow-control commands* described 
below. The commands are not printed as they are executed. An error in any 
command terminates execution of the command file and control is returned to the 
console.

GDB first searches for filename in the current directory. If the file is not 
found there, and filename does not specify a directory, then GDB also looks for 
the file on the source search path (specified with the ‘directory’ command); 
except that $cdir is not searched because the compilation directory is not 
relevant to scripts.

If `-s` is specified, then GDB searches for filename on the search path even if 
filename specifies a directory. The search is done by appending filename to each 
element of the search path. So, for example, if filename is mylib/myscript and 
the search path contains /home/user then GDB will look for the script /home/
user/mylib/myscript. The search is also done if filename is an absolute path. 
For example, if filename is /tmp/myscript and the search path contains /home/
user then GDB will look for the script /home/user/tmp/myscript. For DOS-like 
systems, if filename contains a drive specification, it is stripped before 
concatenation. For example, if filename is d:myscript and the search path 
contains c:/tmp then GDB will look for the script c:/tmp/myscript.

If `-v`, for verbose mode, is given then GDB displays each command as it is 
executed. The option must be given before filename, and is interpreted as part 
of the filename anywhere else.

Commands that would ask for confirmation if used interactively proceed without 
asking when used in a command file. Many GDB commands that normally print 
messages to say what they are doing omit the messages when called from command 
files.

GDB also accepts command input from standard input. In this mode, normal output 
goes to standard output and error output goes to standard error. Errors in a 
command file supplied on standard input do not terminate execution of the 
command file—execution continues with the next command.

```gdb
gdb < cmds > log 2>&1
```

(The syntax above will vary depending on the shell used.) This example will 
execute commands from the file cmds. All output and errors would be directed to 
log.

Since commands stored on command files tend to be more general than commands 
typed interactively, they frequently need to deal with complicated situations, 
such as different or unexpected values of variables and symbols, changes in how 
the program being debugged is built, etc. GDB provides a set of flow-control 
commands to deal with these complexities. Using these commands, you can write 
complex scripts that loop over data structures, execute commands conditionally, 
etc.

- `if`  
  `else`  
   This command allows to include in your script conditionally executed commands. 

   The `if` command takes a single argument, which is an expression to evaluate. It 
   is followed by a series of commands that are executed only if the expression is 
   true (its value is nonzero). There can then optionally be an `else` line, 
   followed by a series of commands that are only executed if the expression was 
   false. The end of the list is marked by a line containing `end`.

- `while`  
   This command allows to write loops. 

   Its syntax is similar to `if`: the command 
   takes a single argument, which is an expression to evaluate, and must be 
   followed by the commands to execute, one per line, terminated by an `end`. These 
   commands are called the *body* of the loop. The commands in the body of `while` 
   are executed repeatedly as long as the expression evaluates to true.

- `loop_break`  
   This command exits the `while` loop in whose body it is included.

   Execution of the script continues after that `while`s `end` line.

- `loop_continue`  
   This command skips the execution of the rest of the body of commands in the 
   `while` loop in whose body it is included. Execution branches to the beginning 
   of the `while` loop, where it evaluates the controlling expression.

- `end`  
   Terminate the block of commands that are the body of `if`, `else`, or `while` 
   flow-control commands.


### 23.1.4 Commands for Controlled Output

During the execution of a command file or a user-defined command, normal GDB 
output is suppressed; the only output that appears is what is explicitly printed 
by the commands in the definition. This section describes three commands useful 
for generating exactly the output you want.

- `echo text`  
   Print text. 

   Nonprinting characters can be included in text using C escape sequences, such as 
   ‘\\n’ to print a newline. **No newline is printed unless you specify one.** In 
   addition to the standard C escape sequences, a backslash followed by a space 
   stands for a space. This is useful for displaying a string with spaces at the 
   beginning or the end, since leading and trailing spaces are otherwise trimmed 
   from all arguments. To print ‘ and foo = ’, use the command ‘echo \\ and foo = \\ ’.

   A backslash at the end of text can be used, as in C, to continue the command 
   onto subsequent lines. For example,

   ```gdb
   echo This is some text\n\
   which is continued\n\
   onto several lines.\n
   ```

   produces the same output as

   ```gdb
   echo This is some text\n
   echo which is continued\n
   echo onto several lines.\n
   ```

- `output expression`  
   Print the value of expression and nothing but that value: no newlines, no ‘$nn = ’. 

   The value is not entered in the value history either. See [Expressions](Expressions.html#Expressions),
   for more information on expressions.

- `output/fmt expression`  
   Print the value of expression in format fmt.

   You can use the same formats as for `print`. See [Output Formats](Output-Formats.html#Output-Formats),
   for more information.

- `printf template, expressions…`  
   Print the values of one or more expressions under the control of the string template. 

   To print several values, make expressions be a comma-separated list of 
   individual expressions, which may be either numbers or pointers. Their values 
   are printed as specified by template, exactly as a C program would do by 
   executing the code below:

   ```gdb
   printf (template, expressions…);
   ```

   As in `C` `printf`, ordinary characters in template are printed verbatim, while 
   *conversion specification* introduced by the ‘%’ character cause subsequent 
   expressions to be evaluated, their values converted and formatted according to 
   type and style information encoded in the conversion specifications, and then 
   printed.

   For example, you can print two values in hex like this:

   ```gdb
   printf "foo, bar-foo = 0x%x, 0x%x\n", foo, bar-foo
   ```

   `printf` supports all the standard `C` conversion specifications, including the 
   flags and modifiers between the ‘%’ character and the conversion letter, with 
   the following exceptions:

   -  The argument-ordering modifiers, such as ‘2$’, are not supported.
   -  The modifier ‘\*’ is not supported for specifying precision or width.
   -  The ‘'’ flag (for separation of digits into groups according to `LC_NUMERIC'`) is not supported.
   -  The type modifiers ‘hh’, ‘j’, ‘t’, and ‘z’ are not supported.
   -  The conversion letter ‘n’ (as in ‘%n’) is not supported.
   -  The conversion letters ‘a’ and ‘A’ are not supported.

   Note that the ‘ll’ type modifier is supported only if the underlying `C` 
   implementation used to build GDB supports the `long long int` type, and the ‘L’ 
   type modifier is supported only if `long double` type is available.

   As in `C`, `printf` supports simple backslash-escape sequences, such as `\n`, ‘\
   \t’, ‘\\\\’, ‘\\"’, ‘\\a’, and ‘\\f’, that consist of backslash followed by a 
   single character. Octal and hexadecimal escape sequences are not supported.

   Additionally, `printf` supports conversion specifications for DFP (*Decimal 
   Floating Point*) types using the following length modifiers together with a 
   floating point specifier. letters:

   -  ‘H’ for printing `Decimal32` types.
   -  ‘D’ for printing `Decimal64` types.
   -  ‘DD’ for printing `Decimal128` types.

   If the underlying `C` implementation used to build GDB has support for the three 
   length modifiers for DFP types, other modifiers such as width and precision will 
   also be available for GDB to use.

   In case there is no such `C` support, no additional modifiers will be available 
   and the value will be printed in the standard way.

   Here’s an example of printing DFP types using the above conversion letters:

   ```gdb
   printf "D32: %Hf - D64: %Df - D128: %DDf\n",1.2345df,1.2E10dd,1.2E1dl
   ```

- `eval template, expressions…`  
   Convert the values of one or more expressions under the control of the string 
   template to a command line, and call it.


### 23.1.5 Controlling auto-loading native GDB scripts

When a new object file is read (for example, due to the `file` command, or 
because the inferior has loaded a shared library), GDB will look for the command 
file objfile-gdb.gdb. See [Auto-loading extensions](Auto_002dloading-
extensions.html#Auto_002dloading-extensions).

Auto-loading can be enabled or disabled, and the list of auto-loaded scripts can 
be printed.

- `set auto-load gdb-scripts [on|off]`  
   Enable or disable the auto-loading of canned sequences of commands scripts.


- `show auto-load gdb-scripts`  
   Show whether auto-loading of canned sequences of commands scripts is enabled 
   or disabled.

- `info auto-load gdb-scripts [regexp]`  
   Print the list of all canned sequences of commands scripts that GDB auto-loaded.

   If regexp is supplied only canned sequences of commands scripts with matching 
   names are printed.


## 23.2 Extending GDB using Python

You can extend GDB using the [Python programming language](http://www.python.org/). 
This feature is available only if GDB was configured using --
with-python.

Python scripts used by GDB should be installed in data-directory/python, where 
data-directory is the data directory as determined at GDB startup (see [Data Files](Data-Files.html#Data-Files)). 
This directory, known as the *python 
directory*, is automatically added to the Python Search Path in order to allow 
the Python interpreter to locate all scripts installed at this location.

Additionally, GDB commands and convenience functions which are written in Python 
and are located in the `data-directory/python/gdb/command or data-directory/python/gdb/function` 
directories are automatically imported when GDB starts.


### 23.2.1 Python Commands

GDB provides two commands for accessing the Python interpreter, and one related setting:

- `python-interactive [command]`  
  `pi [command]`
   Without an argument, the `python-interactive` command can be used to start an 
   interactive Python prompt. 

   To return to GDB, type the `EOF` character (e.g., Ctrl-D on an empty prompt).

   Alternatively, a single-line Python command can be given as an argument and 
   evaluated. If the command is an expression, the result will be printed; 
   otherwise, nothing will be printed. For example:

   ```gdb
   (gdb) python-interactive 2 + 3
   5
   ```

- `python [command]`  
  `py [command]`
   The `python` command can be used to evaluate Python code.

   If given an argument, the `python` command will evaluate the argument as a 
   Python command. For example:

   ```gdb
   (gdb) python print 23
   23
   ```

   If you do not provide an argument to `python`, it will act as a multi-line 
   command, like `define`. In this case, the Python script is made up of subsequent 
   command lines, given after the `python` command. This command list is terminated 
   using a line containing `end`. For example:

   ```gdb
   (gdb) python
   Type python script
   End with a line saying just "end".
   >print 23
   >end
   23
   ```

- `set python print-stack`  
   By default, GDB will print only the message component of a Python exception when 
   an error occurs in a Python script. 

   This can be controlled using `set python print-stack`: if `full`, then full 
   Python stack printing is enabled; if `none`, then Python stack and message 
   printing is disabled; if `message`, the default, only the message component of 
   the error is printed.

   It is also possible to execute a Python script from the GDB interpreter:

- `source script-name`  
   The script name must end with ‘.py’ and GDB must be configured to recognize the 
   script language based on filename extension using the `script-extension` 
   setting. See [Extending GDB](Extending-GDB.html#Extending-GDB).

- `python execfile ("script-name")`  
   This method is based on the `execfile` Python built-in function, and thus is 
   always available.


### 23.2.2 Python API

You can get quick online help for GDB’s Python API by issuing the command 
python help (gdb).

Functions and methods which have two or more optional arguments allow them to be 
specified using keyword syntax. This allows passing some optional arguments 
while skipping others. Example: `gdb.some_function ('foo', bar = 1, baz = 2)`.

|                                                                                                   |     |                                                      |
|---------------------------------------------------------------------------------------------------|-----|------------------------------------------------------|
| • [Basic Python](Basic-Python.html#Basic-Python):                                                 |     | Basic Python Functions.                              |
| • [Exception Handling](Exception-Handling.html#Exception-Handling):                               |     | How Python exceptions are translated.                |
| • [Values From Inferior](Values-From-Inferior.html#Values-From-Inferior):                         |     | Python representation of values.                     |
| • [Types In Python](Types-In-Python.html#Types-In-Python):                                        |     | Python representation of types.                      |
| • [Pretty Printing API](Pretty-Printing-API.html#Pretty-Printing-API):                            |     | Pretty-printing values.                              |
| • [Selecting Pretty-Printers](Selecting-Pretty_002dPrinters.html#Selecting-Pretty_002dPrinters):  |     | How GDB chooses a pretty-printer.                    |
| • [Writing a Pretty-Printer](Writing-a-Pretty_002dPrinter.html#Writing-a-Pretty_002dPrinter):     |     | Writing a Pretty-Printer.                            |
| • [Type Printing API](Type-Printing-API.html#Type-Printing-API):                                  |     | Pretty-printing types.                               |
| • [Frame Filter API](Frame-Filter-API.html#Frame-Filter-API):                                     |     | Filtering Frames.                                    |
| • [Frame Decorator API](Frame-Decorator-API.html#Frame-Decorator-API):                            |     | Decorating Frames.                                   |
| • [Writing a Frame Filter](Writing-a-Frame-Filter.html#Writing-a-Frame-Filter):                   |     | Writing a Frame Filter.                              |
| • [Unwinding Frames in Python](Unwinding-Frames-in-Python.html#Unwinding-Frames-in-Python):       |     | Writing frame unwinder.                              |
| • [Xmethods In Python](Xmethods-In-Python.html#Xmethods-In-Python):                               |     | Adding and replacing methods of C++ classes.         |
| • [Xmethod API](Xmethod-API.html#Xmethod-API):                                                    |     | Xmethod types.                                       |
| • [Writing an Xmethod](Writing-an-Xmethod.html#Writing-an-Xmethod):                               |     | Writing an xmethod.                                  |
| • [Inferiors In Python](Inferiors-In-Python.html#Inferiors-In-Python):                            |     | Python representation of inferiors (processes)       |
| • [Events In Python](Events-In-Python.html#Events-In-Python):                                     |     | Listening for events from GDB.                       |
| • [Threads In Python](Threads-In-Python.html#Threads-In-Python):                                  |     | Accessing inferior threads from Python.              |
| • [Recordings In Python](Recordings-In-Python.html#Recordings-In-Python):                         |     | Accessing recordings from Python.                    |
| • [Commands In Python](Commands-In-Python.html#Commands-In-Python):                               |     | Implementing new commands in Python.                 |
| • [Parameters In Python](Parameters-In-Python.html#Parameters-In-Python):                         |     | Adding new GDB parameters.                           |
| • [Functions In Python](Functions-In-Python.html#Functions-In-Python):                            |     | Writing new convenience functions.                   |
| • [Progspaces In Python](Progspaces-In-Python.html#Progspaces-In-Python):                         |     | Program spaces.                                      |
| • [Objfiles In Python](Objfiles-In-Python.html#Objfiles-In-Python):                               |     | Object files.                                        |
| • [Frames In Python](Frames-In-Python.html#Frames-In-Python):                                     |     | Accessing inferior stack frames from Python.         |
| • [Blocks In Python](Blocks-In-Python.html#Blocks-In-Python):                                     |     | Accessing blocks from Python.                        |
| • [Symbols In Python](Symbols-In-Python.html#Symbols-In-Python):                                  |     | Python representation of symbols.                    |
| • [Symbol Tables In Python](Symbol-Tables-In-Python.html#Symbol-Tables-In-Python):                |     | Python representation of symbol tables.              |
| • [Line Tables In Python](Line-Tables-In-Python.html#Line-Tables-In-Python):                      |     | Python representation of line tables.                |
| • [Breakpoints In Python](Breakpoints-In-Python.html#Breakpoints-In-Python):                      |     | Manipulating breakpoints using Python.               |
| • [Finish Breakpoints in Python](Finish-Breakpoints-in-Python.html#Finish-Breakpoints-in-Python): |     | Setting Breakpoints on function return using Python. |
| • [Lazy Strings In Python](Lazy-Strings-In-Python.html#Lazy-Strings-In-Python):                   |     | Python representation of lazy strings.               |
| • [Architectures In Python](Architectures-In-Python.html#Architectures-In-Python):                |     | Python representation of architectures.              |


### 23.2.3 Python Auto-loading

When a new object file is read (for example, due to the `file` command, or 
because the inferior has loaded a shared library), GDB will look for Python 
support scripts in several ways: objfile-gdb.py and `.debug_gdb_scripts` 
section. See [Auto-loading extensions](Auto_002dloading-
extensions.html#Auto_002dloading-extensions).

The auto-loading feature is useful for supplying application-specific debugging 
commands and scripts.

Auto-loading can be enabled or disabled, and the list of auto-loaded scripts can 
be printed.

- `set auto-load python-scripts [on|off]`  
   Enable or disable the auto-loading of Python scripts.

- `show auto-load python-scripts`  
   Show whether auto-loading of Python scripts is enabled or disabled.

- `info auto-load python-scripts [regexp]`  
   Print the list of all Python scripts that GDB auto-loaded.

Also printed is the list of Python scripts that were mentioned in the 
`.debug_gdb_scripts` section and were either not found (see 
[dotdebug\_gdb\_scripts section](dotdebug_005fgdb_005fscripts-section.html#dotdebug_005fgdb_005fscripts-section))
or were not auto-loaded due to `auto-load safe-path` rejection (see [Auto-loading]
(Auto_002dloading.html#Auto_002dloading)). This is useful because their names 
are not printed when GDB tries to load them and fails. There may be many of 
them, and printing an error message for each one is problematic.

If regexp is supplied only Python scripts with matching names are printed.

**Example:**

```gdb
(gdb) info auto-load python-scripts
Loaded Script
Yes    py-section-script.py
       full name: /tmp/py-section-script.py
No     my-foo-pretty-printers.py
```

When reading an auto-loaded file or script, GDB sets the *current objfile*. This 
is available via the `gdb.current_objfile` function (see [Objfiles In Python](Objfiles-In-Python.html#Objfiles-In-Python)).
This can be useful for registering objfile-specific pretty-printers and frame-filters.


### 23.2.4 Python modules

GDB comes with several modules to assist writing Python code.


#### 23.2.4.1 gdb.printing

This module provides a collection of utilities for working with pretty-printers.

- `PrettyPrinter (name, subprinters=None)`  
   This class specifies the API that makes ‘info pretty-printer’, ‘enable pretty-
   printer’ and ‘disable pretty-printer’ work.

   Pretty-printers should generally inherit from this class.

- `SubPrettyPrinter (name)`  
   For printers that handle multiple types, this class specifies the corresponding 
   API for the subprinters.

- `RegexpCollectionPrettyPrinter (name)`  
   Utility class for handling multiple printers, all recognized via regular 
   expressions. See [Writing a Pretty-Printer](Writing-a-Pretty_002dPrinter.html#Writing-a-Pretty_002dPrinter), 
   for an example.

- `FlagEnumerationPrinter (name)`  
   A pretty-printer which handles printing of `enum` values.

   Unlike GDB’s built-in `enum` printing, this printer attempts to work properly 
   when there is some overlap between the enumeration constants. The argument name 
   is the name of the printer and also the name of the `enum` type to look up.

- `register_pretty_printer (obj, printer, replace=False)`  
   Register printer with the pretty-printer list of obj.

   If replace is `True` then any existing copy of the printer is replaced. 
   Otherwise a `RuntimeError` exception is raised if a printer with the same name 
   already exists.


#### 23.2.4.2 gdb.types

This module provides a collection of utilities for working with `gdb.Type` objects.

- `get_basic_type (type)`  
   Return type with const and volatile qualifiers stripped, and with typedefs and 
   C`++` references converted to the underlying type.

   **C++ example:**

   ```gdb
   typedef const int const_int;
   const_int foo (3);
   const_int& foo_ref (foo);
   int main () { return 0; }
   ```

   Then in gdb:

   ```gdb
   (gdb) start
   (gdb) python import gdb.types
   (gdb) python foo_ref = gdb.parse_and_eval("foo_ref")
   (gdb) python print gdb.types.get_basic_type(foo_ref.type)
   int
   ```

- `has_field (type, field)`  
   Return `True` if type, assumed to be a type with fields (e.g., a structure or union), has field field.

- `make_enum_dict (enum_type)`  
   Return a Python `dictionary` type produced from enum\_type.

- `deep_items (type)`  
   Returns a Python iterator similar to the standard `gdb.Type.iteritems` 
   method, except that the iterator returned by `deep_items` will recursively 
   traverse anonymous struct or union fields. For example:

   ```c
   struct A
   {
     int a;
     union {
       int b0;
       int b1;
     };
   };
   ```

   Then in GDB:

   ```gdb
   (gdb) python import gdb.types
   (gdb) python struct_a = gdb.lookup_type("struct A")
   (gdb) python print struct_a.keys ()
   {['a', '']}
   (gdb) python print [k for k,v in gdb.types.deep_items(struct_a)]
   {['a', 'b0', 'b1']}
   ```

- `get_type_recognizers ()`  
   Return a list of the enabled type recognizers for the current context.

   This is called by GDB during the type-printing process (see [Type Printing API](Type-Printing-API.html#Type-Printing-API)).

- `apply_type_recognizers (recognizers, type_obj)`  
   Apply the type recognizers, recognizers, to the type object type\_obj.

   If any recognizer returns a string, return that string. Otherwise, return 
   `None`. This is called by GDB during the type-printing process (see [Type Printing API](Type-Printing-API.html#Type-Printing-API)).

- `register_type_printer (locus, printer)`  
   This is a convenience function to register a type printer printer.

   The printer must implement the type printer protocol. The locus argument is 
   either a `gdb.Objfile`, in which case the printer is registered with that 
   objfile; a `gdb.Progspace`, in which case the printer is registered with that 
   progspace; or `None`, in which case the printer is registered globally.

- `TypePrinter`  
   This is a base class that implements the type printer protocol.

   Type printers are encouraged, but not required, to derive from this class. It 
   defines a constructor:

   []()Method on TypePrinter: **\_\_init\_\_** *(self, name)*  
   Initialize the type printer with the given name. The new printer starts in 
   the enabled state.


#### 23.2.4.3 gdb.prompt

This module provides a method for prompt value-substitution.

- `substitute_prompt (string)`  
   Return string with escape sequences substituted by values.

   Some escape sequences take arguments. You can specify arguments inside “{}” 
   immediately following the escape sequence.

   The escape sequences you can pass to this function are:

   - `\\`  
      Substitute a backslash.
   
   - `\e`  
      Substitute an ESC character.
   
   - `\f`  
      Substitute the selected frame; an argument names a frame parameter.
   
   - `\n`  
      Substitute a newline.
   
   - `\p`  
      Substitute a parameter’s value; the argument names the parameter.
   
   - `\r`  
      Substitute a carriage return.
   
   - `\t`  
      Substitute the selected thread; an argument names a thread parameter.
   
   - `\v`  
      Substitute the version of GDB.
   
   - `\w`  
      Substitute the current working directory.
   
   - `\[`  
      Begin a sequence of non-printing characters.
   
      These sequences are typically used with the ESC character, and are not counted 
      in the string length. Example: “\\\[\\e\[0;34m\\\](gdb)\\\[\\e\[0m\\\]” will 
      return a blue-colored “(gdb)” prompt where the length is five.
   
   `\]`  
      End a sequence of non-printing characters.
   
      For example:
   
   ```gdb
   substitute_prompt (``frame: \f,
                   print arguments: \p{print frame-arguments}'')
   ```

will return the string:

```gdb
"frame: main, print arguments: scalars"
```


## 23.3 Extending GDB using Guile

You can extend GDB using the [Guile implementation of the Scheme programming language](http://www.gnu.org/software/guile/).
This feature is available only if GDB was configured using --with-guile.


### 23.3.1 Guile Introduction

Guile is an implementation of the Scheme programming language and is the GNU 
project’s official extension language.

Guile support in GDB follows the Python support in GDB reasonably closely, so 
concepts there should carry over. However, some things are done differently 
where it makes sense.

GDB requires Guile version 2.0 or greater. Older versions are not supported.

Guile scripts used by GDB should be installed in data-directory/guile, where 
data-directory is the data directory as determined at GDB startup (see [Data Files](Data-Files.html#Data-Files)). 
This directory, known as the *guile 
directory*, is automatically added to the Guile Search Path in order to allow 
the Guile interpreter to locate all scripts installed at this location.


### 23.3.2 Guile Commands

GDB provides two commands for accessing the Guile interpreter:

- `guile-repl`  
  `gr`  
   The `guile-repl` command can be used to start an interactive Guile prompt or 
   *repl*. 

   To return to GDB, type ,q or the `EOF` character (e.g., Ctrl-D on an empty 
   prompt). These commands do not take any arguments.

- `guile [scheme-expression]`  
  `gu [scheme-expression]`  
   The `guile` command can be used to evaluate a Scheme expression.

   If given an argument, GDB will pass the argument to the Guile interpreter for 
   evaluation.

   ```gdb
   (gdb) guile (display (+ 20 3)) (newline)
   23
   ```

   The result of the Scheme expression is displayed using normal Guile rules.

   ```gdb
   (gdb) guile (+ 20 3)
   23
   ```

   If you do not provide an argument to `guile`, it will act as a multi-line 
   command, like `define`. In this case, the Guile script is made up of subsequent 
   command lines, given after the `guile` command. This command list is terminated 
   using a line containing `end`. For example:

   ```gdb
   (gdb) guile
   >(display 23)
   >(newline)
   >end
   23
   ```

   It is also possible to execute a Guile script from the GDB interpreter:

- `source script-name`  
   The script name must end with ‘.scm’ and GDB must be configured to recognize the 
   script language based on filename extension using the `script-extension` 
   setting. See [Extending GDB](Extending-GDB.html#Extending-GDB).

- `guile (load "script-name")`  
   This method uses the `load` Guile function. It takes a string argument that is 
   the name of the script to load. See the Guile documentation for a description of 
   this function. (see [Loading](http://www.gnu.org/software/guile/manual/html_node/Loading.html#Loading)
   in GNU Guile Reference Manual).


### 23.3.3 Guile API

You can get quick online help for GDB’s Guile API by issuing the command 
help guile, or by issuing the command ,help from an interactive Guile session. 
Furthermore, most Guile procedures provided by GDB have doc strings which can be 
obtained with ,describe procedure-name or ,d procedure-name from the Guile 
interactive prompt.

|                                                                                                                    |     |                                                |
|--------------------------------------------------------------------------------------------------------------------|-----|------------------------------------------------|
| • [Basic Guile](Basic-Guile.html#Basic-Guile):                                                                     |     | Basic Guile Functions                          |
| • [Guile Configuration](Guile-Configuration.html#Guile-Configuration):                                             |     | Guile configuration variables                  |
| • [GDB Scheme Data Types](GDB-Scheme-Data-Types.html#GDB-Scheme-Data-Types):                                       |     | Scheme representations of GDB objects          |
| • [Guile Exception Handling](Guile-Exception-Handling.html#Guile-Exception-Handling):                              |     | How Guile exceptions are translated            |
| • [Values From Inferior In Guile](Values-From-Inferior-In-Guile.html#Values-From-Inferior-In-Guile):               |     | Guile representation of values                 |
| • [Arithmetic In Guile](Arithmetic-In-Guile.html#Arithmetic-In-Guile):                                             |     | Arithmetic in Guile                            |
| • [Types In Guile](Types-In-Guile.html#Types-In-Guile):                                                            |     | Guile representation of types                  |
| • [Guile Pretty Printing API](Guile-Pretty-Printing-API.html#Guile-Pretty-Printing-API):                           |     | Pretty-printing values with Guile              |
| • [Selecting Guile Pretty-Printers](Selecting-Guile-Pretty_002dPrinters.html#Selecting-Guile-Pretty_002dPrinters): |     | How GDB chooses a pretty-printer               |
| • [Writing a Guile Pretty-Printer](Writing-a-Guile-Pretty_002dPrinter.html#Writing-a-Guile-Pretty_002dPrinter):    |     | Writing a pretty-printer                       |
| • [Commands In Guile](Commands-In-Guile.html#Commands-In-Guile):                                                   |     | Implementing new commands in Guile             |
| • [Parameters In Guile](Parameters-In-Guile.html#Parameters-In-Guile):                                             |     | Adding new GDB parameters                      |
| • [Progspaces In Guile](Progspaces-In-Guile.html#Progspaces-In-Guile):                                             |     | Program spaces                                 |
| • [Objfiles In Guile](Objfiles-In-Guile.html#Objfiles-In-Guile):                                                   |     | Object files in Guile                          |
| • [Frames In Guile](Frames-In-Guile.html#Frames-In-Guile):                                                         |     | Accessing inferior stack frames from Guile     |
| • [Blocks In Guile](Blocks-In-Guile.html#Blocks-In-Guile):                                                         |     | Accessing blocks from Guile                    |
| • [Symbols In Guile](Symbols-In-Guile.html#Symbols-In-Guile):                                                      |     | Guile representation of symbols                |
| • [Symbol Tables In Guile](Symbol-Tables-In-Guile.html#Symbol-Tables-In-Guile):                                    |     | Guile representation of symbol tables          |
| • [Breakpoints In Guile](Breakpoints-In-Guile.html#Breakpoints-In-Guile):                                          |     | Manipulating breakpoints using Guile           |
| • [Lazy Strings In Guile](Lazy-Strings-In-Guile.html#Lazy-Strings-In-Guile):                                       |     | Guile representation of lazy strings           |
| • [Architectures In Guile](Architectures-In-Guile.html#Architectures-In-Guile):                                    |     | Guile representation of architectures          |
| • [Disassembly In Guile](Disassembly-In-Guile.html#Disassembly-In-Guile):                                          |     | Disassembling instructions from Guile          |
| • [I/O Ports in Guile](I_002fO-Ports-in-Guile.html#I_002fO-Ports-in-Guile):                                        |     | GDB I/O ports                                  |
| • [Memory Ports in Guile](Memory-Ports-in-Guile.html#Memory-Ports-in-Guile):                                       |     | Accessing memory through ports and bytevectors |
| • [Iterators In Guile](Iterators-In-Guile.html#Iterators-In-Guile):                                                |     | Basic iterator support                         |


### 23.3.4 Guile Auto-loading

When a new object file is read (for example, due to the `file` command, or 
because the inferior has loaded a shared library), GDB will look for Guile 
support scripts in two ways: objfile-gdb.scm and the `.debug_gdb_scripts` 
section. See [Auto-loading extensions](Auto_002dloading-
extensions.html#Auto_002dloading-extensions).

The auto-loading feature is useful for supplying application-specific debugging 
commands and scripts.

Auto-loading can be enabled or disabled, and the list of auto-loaded scripts can 
be printed.

- `set auto-load guile-scripts [on|off]`  
   Enable or disable the auto-loading of Guile scripts.

- `show auto-load guile-scripts`  
   Show whether auto-loading of Guile scripts is enabled or disabled.

- `info auto-load guile-scripts [regexp]`  
   Print the list of all Guile scripts that GDB auto-loaded.

   Also printed is the list of Guile scripts that were mentioned in the 
   `.debug_gdb_scripts` section and were not found. This is useful because their 
   names are not printed when GDB tries to load them and fails. There may be many 
   of them, and printing an error message for each one is problematic.

   If regexp is supplied only Guile scripts with matching names are printed.

   **Example:**

   ```gdb
   (gdb) info auto-load guile-scripts
   Loaded Script
   Yes    scm-section-script.scm
       full name: /tmp/scm-section-script.scm
   No     my-foo-pretty-printers.scm
   ```

When reading an auto-loaded file, GDB sets the *current objfile*. This is 
available via the `current-objfile` procedure (see [Objfiles In Guile](Objfiles-In-Guile.html#Objfiles-In-Guile)).
This can be useful for registering objfile-specific pretty-printers.


### 23.3.5 Guile Modules

GDB comes with several modules to assist writing Guile code.


#### 23.3.5.1 Guile Printing Module

This module provides a collection of utilities for working with pretty-printers.

Usage:

```gdb
(use-modules (gdb printing))
```

-  Scheme Procedure: **prepend-pretty-printer!** *object printer*  
   Add printer to the front of the list of pretty-printers for object.

   The object must either be a `<gdb:objfile>` object, or `#f` in which case 
   printer is added to the global list of printers.

-  Scheme Procecure: **append-pretty-printer!** *object printer*  
   Add printer to the end of the list of pretty-printers for object.

   The object must either be a `<gdb:objfile>` object, or `#f` in which case 
   printer is added to the global list of printers.


#### 23.3.5.2 Guile Types Module

This module provides a collection of utilities for working with `<gdb:type>` 
objects.

Usage:

```gdb
(use-modules (gdb types))
```

-  Scheme Procedure: **get-basic-type** *type*  
   Return type with const and volatile qualifiers stripped, and with typedefs and 
   C`++` references converted to the underlying type.

   C++ example:

   ```gdb
   typedef const int const_int;
   const_int foo (3);
   const_int& foo_ref (foo);
   int main () { return 0; }
   ```

   Then in gdb:

   ```gdb
   (gdb) start
   (gdb) guile (use-modules (gdb) (gdb types))
   (gdb) guile (define foo-ref (parse-and-eval "foo_ref"))
   (gdb) guile (get-basic-type (value-type foo-ref))
   int
   ```

-  Scheme Procedure: **type-has-field-deep?** *type field*  
   Return `#t` if type, assumed to be a type with fields (e.g., a structure or 
   union), has field field. Otherwise return `#f`. This searches baseclasses, 
   whereas `type-has-field?` does not.

-  Scheme Procedure: **make-enum-hashtable** *enum-type*  
   Return a Guile hash table produced from enum-type. Elements in the hash table 
   are referenced with `hashq-ref`.


## 23.4 Auto-loading extensions

GDB provides two mechanisms for automatically loading extensions when a new 
object file is read (for example, due to the `file` command, or because the 
inferior has loaded a shared library): objfile-gdb.ext and the 
`.debug_gdb_scripts` section of modern file formats like ELF.

The auto-loading feature is useful for supplying application-specific debugging 
commands and features.

Auto-loading can be enabled or disabled, and the list of auto-loaded scripts can 
be printed. See the ‘auto-loading’ section of each extension language for more 
information. For GDB command files see [Auto-loading sequences]
(Auto_002dloading-sequences.html#Auto_002dloading-sequences). For Python files 
see [Python Auto-loading](Python-Auto_002dloading.html#Python-Auto_002dloading).

Note that loading of this script file also requires accordingly configured 
`auto-load safe-path` (see [Auto-loading safe path](Auto_002dloading-safe-path.html#Auto_002dloading-safe-path)).


### 23.4.1 The objfile-gdb.ext file

When a new object file is read, GDB looks for a file named objfile-gdb.ext (we 
call it script-name below), where objfile is the object file’s name and where 
ext is the file extension for the extension language:

- `objfile-gdb.gdb`  
   GDB’s own command language

- `objfile-gdb.py`  
   Python

- `objfile-gdb.scm`  
   Guile

script-name is formed by ensuring that the file name of objfile is absolute, 
following all symlinks, and resolving `.` and `..` components, and appending the 
-gdb.ext suffix. If this file exists and is readable, GDB will evaluate it as a 
script in the specified extension language.

If this file does not exist, then GDB will look for script-name file in all of 
the directories as specified below.

Note that loading of these files requires an accordingly configured `auto-load 
safe-path` (see [Auto-loading safe path](Auto_002dloading-safe-path.html#Auto_002dloading-safe-path)).

For object files using .exe suffix GDB tries to load first the scripts normally 
according to its .exe filename. But if no scripts are found GDB also tries 
script filenames matching the object file without its .exe suffix. This .exe 
stripping is case insensitive and it is attempted on any platform. This makes 
the script filenames compatible between Unix and MS-Windows hosts.

- `set auto-load scripts-directory [directories]`  
   Control GDB auto-loaded scripts location.

   Multiple directory entries may be delimited by the host platform path separator 
   in use (‘:’ on Unix, ‘;’ on MS-Windows and MS-DOS).

   Each entry here needs to be covered also by the security setting `set auto-load 
   safe-path` (see [set auto-load safe-path](Auto_002dloading-safe-path.html#set-auto_002dload-safe_002dpath)).

   This variable defaults to $debugdir:$datadir/auto-load. The default `set auto-
   load safe-path` value can be also overriden by GDB configuration option --with-
   auto-load-dir.

   Any reference to $debugdir will get replaced by debug-file-directory value (see 
   [Separate Debug Files](Separate-Debug-Files.html#Separate-Debug-Files)) and any 
   reference to $datadir will get replaced by data-directory which is determined at 
   GDB startup (see [Data Files](Data-Files.html#Data-Files)). $debugdir and 
   $datadir must be placed as a directory component — either alone or delimited 
   by / or \\ directory separators, depending on the host platform.

   The list of directories uses path separator (‘:’ on GNU and Unix systems, ‘;’ on 
   MS-Windows and MS-DOS) to separate directories, similarly to the `PATH` 
   environment variable.

- `show auto-load scripts-directory`  
   Show GDB auto-loaded scripts location.

- `add-auto-load-scripts-directory [directories…]`  
   Add an entry (or list of entries) to the list of auto-loaded scripts locations.

   Multiple entries may be delimited by the host platform path separator in use.

GDB does not track which files it has already auto-loaded this way. GDB will 
load the associated script every time the corresponding objfile is opened. So 
your -gdb.ext file should be careful to avoid errors if it is evaluated more 
than once.


### 23.4.2 The `.debug_gdb_scripts` section

For systems using file formats like ELF and COFF, when GDB loads a new object 
file it will look for a special section named `.debug_gdb_scripts`. If this 
section exists, its contents is a list of null-terminated entries specifying 
scripts to load. Each entry begins with a non-null prefix byte that specifies 
the kind of entry, typically the extension language and whether the script is in 
a file or inlined in `.debug_gdb_scripts`.

The following entries are supported:

`SECTION_SCRIPT_ID_PYTHON_FILE = 1`
`SECTION_SCRIPT_ID_SCHEME_FILE = 3`
`SECTION_SCRIPT_ID_PYTHON_TEXT = 4`
`SECTION_SCRIPT_ID_SCHEME_TEXT = 6`

#### 23.4.2.1 Script File Entries

If the entry specifies a file, GDB will look for the file first in the current 
directory and then along the source search path (see [Specifying Source 
Directories](Source-Path.html#Source-Path)), except that $cdir is not searched, 
since the compilation directory is not relevant to scripts.

File entries can be placed in section `.debug_gdb_scripts` with, for example, 
this GCC macro for Python scripts.

```gdb
/* Note: The "MS" section flags are to remove duplicates.  */
#define DEFINE_GDB_PY_SCRIPT(script_name) \
  asm("\
.pushsection \".debug_gdb_scripts\", \"MS\",@progbits,1\n\
.byte 1 /* Python */\n\
.asciz \"" script_name "\"\n\
.popsection \n\
");
```

For Guile scripts, replace `.byte 1` with `.byte 3`. Then one can reference the 
macro in a header or source file like this:

```gdb
DEFINE_GDB_PY_SCRIPT ("my-app-scripts.py")
```

The script name may include directories if desired.

Note that loading of this script file also requires accordingly configured 
`auto-load safe-path` (see [Auto-loading safe path](Auto_002dloading-safe-path.html#Auto_002dloading-safe-path)).

If the macro invocation is put in a header, any application or library using 
this header will get a reference to the specified script, and with the use of 
`"MS"` attributes on the section, the linker will remove duplicates.

#### 23.4.2.2 Script Text Entries

Script text entries allow to put the executable script in the entry itself 
instead of loading it from a file. The first line of the entry, everything after 
the prefix byte and up to the first newline (`0xa`) character, is the script 
name, and must not contain any kind of space character, e.g., spaces or tabs. 
The rest of the entry, up to the trailing null byte, is the script to execute in 
the specified language. The name needs to be unique among all script names, as 
GDB executes each script only once based on its name.

Here is an example from file py-section-script.c in the GDB testsuite.

```gdb
#include "symcat.h"
#include "gdb/section-scripts.h"
asm(
".pushsection \".debug_gdb_scripts\", \"MS\",@progbits,1\n"
".byte " XSTRING (SECTION_SCRIPT_ID_PYTHON_TEXT) "\n"
".ascii \"gdb.inlined-script\\n\"\n"
".ascii \"class test_cmd (gdb.Command):\\n\"\n"
".ascii \"  def __init__ (self):\\n\"\n"
".ascii \"    super (test_cmd, self).__init__ ("
    "\\\"test-cmd\\\", gdb.COMMAND_OBSCURE)\\n\"\n"
".ascii \"  def invoke (self, arg, from_tty):\\n\"\n"
".ascii \"    print (\\\"test-cmd output, arg = %s\\\" % arg)\\n\"\n"
".ascii \"test_cmd ()\\n\"\n"
".byte 0\n"
".popsection\n"
);
```

Loading of inlined scripts requires a properly configured `auto-load safe-path` 
(see [Auto-loading safe path](Auto_002dloading-safe-path.html#Auto_002dloading-safe-path)).
The path to specify in `auto-load safe-path` is the path of the 
file containing the `.debug_gdb_scripts` section.


### 23.4.3 Which flavor to choose?

Given the multiple ways of auto-loading extensions, it might not always be clear 
which one to choose. This section provides some guidance.

Benefits of the -gdb.ext way:

-  Can be used with file formats that don’t support multiple sections.
-  Ease of finding scripts for public libraries.

   Scripts specified in the `.debug_gdb_scripts` section are searched for in 
   the source search path. For publicly installed libraries, e.g., libstdc++, there 
   typically isn’t a source directory in which to find the script.

-  Doesn’t require source code additions.

Benefits of the `.debug_gdb_scripts` way:

-  Works with static linking.

   Scripts for libraries done the -gdb.ext way require an objfile to trigger 
   their loading. When an application is statically linked the only objfile 
   available is the executable, and it is cumbersome to attach all the scripts from 
   all the input libraries to the executable’s -gdb.ext script.

-  Works with classes that are entirely inlined.

   Some classes can be entirely inlined, and thus there may not be an associated 
   shared library to attach a -gdb.ext script to.

-  Scripts needn’t be copied out of the source tree.

   In some circumstances, apps can be built out of large collections of internal 
   libraries, and the build infrastructure necessary to install the -gdb.ext 
   scripts in a place where GDB can find them is cumbersome. It may be easier to 
   specify the scripts in the `.debug_gdb_scripts` section as relative paths, and 
   add a path to the top of the source tree to the source search path.


## 23.5 Multiple Extension Languages

The Guile and Python extension languages do not share any state, and generally 
do not interfere with each other. There are some things to be aware of, however.

### 23.5.1 Python comes first

Python was GDB’s first extension language, and to avoid breaking existing 
behaviour Python comes first. This is generally solved by the “first one wins” 
principle. GDB maintains a list of enabled extension languages, and when it 
makes a call to an extension language, (say to pretty-print a value), it tries 
each in turn until an extension language indicates it has performed the request 
(e.g., has returned the pretty-printed form of a value). This extends to errors 
while performing such requests: If an error happens while, for example, trying 
to pretty-print an object then the error is reported and any following extension 
languages are not tried.


## 23.6 Creating new spellings of existing commands

It is often useful to define alternate spellings of existing commands. For 
example, if a new GDB command defined in Python has a long name to type, it is 
handy to have an abbreviated version of it that involves less typing.

GDB itself uses aliases. For example ‘s’ is an alias of the ‘step’ command even 
though it is otherwise an ambiguous abbreviation of other commands like ‘set’ 
and ‘show’.

Aliases are also used to provide shortened or more common versions of multi-word 
commands. For example, GDB provides the ‘tty’ alias of the ‘set inferior-tty’ 
command.

You can define a new alias with the ‘alias’ command.

`alias [-a] [--] ALIAS = COMMAND`

ALIAS specifies the name of the new alias.

Each word of ALIAS must consist of letters, numbers, dashes and underscores.

COMMAND specifies the name of an existing command that is being aliased.

The ‘-a’ option specifies that the new alias is an abbreviation of the command. 
Abbreviations are not shown in command lists displayed by the ‘help’ command.

The ‘--’ option specifies the end of options, and is useful when ALIAS begins 
with a dash.

Here is a simple example showing how to make an abbreviation of a command so 
that there is less to type. Suppose you were tired of typing ‘disas’, the 
current shortest unambiguous abbreviation of the ‘disassemble’ command and you 
wanted an even shorter version named ‘di’. The following will accomplish this.

```gdb
(gdb) alias -a di = disas
```

Note that aliases are different from user-defined commands. With a user-defined 
command, you also need to write documentation for it with the ‘document’ 
command. An alias automatically picks up the documentation of the existing 
command.

Here is an example where we make ‘elms’ an abbreviation of ‘elements’ in the 
‘set print elements’ command. This is to show that you can make an abbreviation 
of any part of a command.

```gdb
(gdb) alias -a set print elms = set print elements
(gdb) alias -a show print elms = show print elements
(gdb) set p elms 20
(gdb) show p elms
Limit on string chars or array elements to print is 200.
```

Note that if you are defining an alias of a ‘set’ command, and you want to have 
an alias for the corresponding ‘show’ command, then you need to define the 
latter separately.

Unambiguously abbreviated commands are allowed in COMMAND and ALIAS, just as 
they are normally.

```gdb
(gdb) alias -a set pr elms = set p ele
```

Finally, here is an example showing the creation of a one word alias for a more 
complex command. This creates alias ‘spe’ of the command ‘set print elements’.

```gdb
(gdb) alias spe = set print elements
(gdb) spe 20
```
