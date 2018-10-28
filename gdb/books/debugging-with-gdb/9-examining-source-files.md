# 9 Examining Source Files


* [9.1 Printing Source Lines](#91-printing-source-lines)
* [9.2 Specifying a Location](#92-specifying-a-location)
    * [9.2.1 Linespec Locations](#921-linespec-locations)
    * [9.2.2 Explicit Locations](#922-explicit-locations)
    * [9.2.3 Address Locations](#923-address-locations)
* [9.3 Editing Source Files](#93-editing-source-files)
    * [9.3.1 Choosing your Editor](#931-choosing-your-editor)
    * [Footnotes](#footnotes)
    * [(9)](#9)
* [9.4 Searching Source Files](#94-searching-source-files)
* [9.5 Specifying Source Directories](#95-specifying-source-directories)
* [9.6 Source and Machine Code](#96-source-and-machine-code)


GDB can print parts of your program’s source, since the debugging information 
recorded in the program tells GDB what source files were used to build it. When 
your program stops, GDB spontaneously prints the line where it stopped. 
Likewise, when you select a stack frame (see [Selecting a Frame](Selection.html#Selection)), 
GDB prints the line where execution in that frame 
has stopped. You can print other portions of source files by explicit command.

If you use GDB through its GNU Emacs interface, you may prefer to use Emacs 
facilities to view source; see [Using GDB under GNU Emacs](Emacs.html#Emacs).


## 9.1 Printing Source Lines

To print lines from a source file, use the `list` command (abbreviated `l`). By 
default, ten lines are printed. There are several ways to specify what part of 
the file you want to print; see [Specify Location](Specify-Location.html#Specify-Location), 
for the full list.

Here are the forms of the `list` command most commonly used:

- `list linenum`  
   Print lines centered around line number `linenum` in the current source file.

- `list function`  
   Print lines centered around the beginning of function `function`.

- `list`  
   Print more lines. 

   If the last lines printed were printed with a `list` command, this prints 
   lines following the last lines printed; however, if the last line printed was a 
   solitary line printed as part of displaying a stack frame (see [Examining the Stack](Stack.html#Stack)), 
   this prints lines centered around that line.

- `list -`  
   Print lines just before the lines last printed.

By default, GDB prints ten source lines with any of these forms of the `list` 
command. You can change this using `set listsize`:

- `set listsize count`  
  `set listsize unlimited`  
   Make the `list` command display count source lines (unless the `list` 
   argument explicitly specifies some other number).

   Setting count to `unlimited` or 0 means there’s no limit.

- `show listsize`  
   Display the number of lines that `list` prints.

Repeating a `list` command with `RET` discards the argument, so it is 
equivalent to typing just `list`. This is more useful than listing the same 
lines again. An exception is made for an argument of ‘-’; that argument is 
preserved in repetition so that each repetition moves up in the source file.

In general, the `list` command expects you to supply zero, one or two 
*locations*. Locations specify source lines; there are several ways of writing 
them (see [Specify Location](Specify-Location.html#Specify-Location)), but the 
effect is always to specify some source line.

Here is a complete description of the possible arguments for `list`:

- `list location`  
   Print lines centered around the line specified by `location`.

- `list first,last`  
   Print lines from first to last. 

   Both arguments are locations. When a `list` command has two locations, and 
   the source file of the second location is omitted, this refers to the same 
   source file as the first location.

- `list ,last`  
   Print lines ending with last.

- `list first,`  
   Print lines starting with first.

- `list +`  
   Print lines just after the lines last printed.

- `list -`  
   Print lines just before the lines last printed.

- `list`  
   As described in the preceding table.


## 9.2 Specifying a Location

Several GDB commands accept arguments that specify a location of your program’s 
code. Since GDB is a source-level debugger, a location usually specifies some 
line in the source code. Locations may be specified using three different 
formats: linespec locations, explicit locations, or address locations.


### 9.2.1 Linespec Locations

A *linespec* is a colon-separated list of source location parameters such as 
file name, function name, etc. Here are all the different ways of specifying a 
linespec:

- *`linenum`*  
   Specifies the line number *linenum* of the current source file.

- *`-offset`*  
  *`+offset`*  
   Specifies the line *offset* lines before or after the *current line*. 

   For the `list` command, the current line is the last one printed; for the 
   breakpoint commands, this is the line at which execution stopped in the 
   currently selected *stack frame* (see [Frames](Frames.html#Frames), for a 
   description of stack frames.) When used as the second of the two linespecs in a 
   `list` command, this specifies the line *offset* lines up or down from the first 
   linespec.

- *`filename:linenum`*  
   Specifies the line *linenum* in the source file *filename*. 

   If *filename* is a relative file name, then it will match any source file name 
   with the same trailing components. For example, if *filename* is ‘gcc/expr.c’, 
   then it will match source file name of `/build/trunk/gcc/expr.c`, but not 
   `/build/trunk/libcpp/expr.c` or `/build/trunk/gcc/x-expr.c`.

- *`function`*  
   Specifies the line that begins the body of the function *function*. 

   For example, in C, this is the line with the open brace.

   By default, in C++ and Ada, *function* is interpreted as specifying all 
   functions named *function* in all scopes. For C++, this means in all namespaces 
   and classes. For Ada, this means in all packages.

   For example, assuming a program with C++ symbols named `A::B::foobar` and 
   `B::foobar`, both commands `break foobar` and `break B::foobar` set a breakpoint 
   on both symbols.

   Commands that accept a linespec let you override this with the `-qualified` 
   option. For example, `break -qualified foobar` sets a breakpoint on a 
   free-function named `foobar` ignoring any C++ class methods and namespace 
   functions called `foobar`.

   See [Explicit Locations](#922-explicit-locations).

- `function:label`  
   Specifies the line where *label* appears in *function*.

- `filename:function`  
   Specifies the line that begins the body of the function *function* in the file 
   *filename*. 

   You only need the file name with a function name to avoid ambiguity when 
   there are identically named functions in different source files.

- `label`  
   Specifies the line at which the label named *label* appears in the function 
   corresponding to the currently selected stack frame. 

   If there is no current selected stack frame (for instance, if the inferior is 
   not running), then GDB will not search for a label.

- `-pstap|-probe-stap [objfile:[provider:]]name`  
   The GNU/Linux tool `SystemTap` provides a way for applications to embed static probes. 

   See [Static Probe Points](Static-Probe-Points.html#Static-Probe-Points), for 
   more information on finding and using static probes. This form of linespec 
   specifies the location of such a static probe.

   If *objfile* is given, only probes coming from that shared library or 
   executable matching *objfile* as a regular expression are considered. If *provider*
   is given, then only probes from that provider are considered. If several probes 
   match the spec, GDB will insert a breakpoint at each one of those probes.


### 9.2.2 Explicit Locations

*Explicit locations* allow the user to directly specify the source location’s 
parameters using option-value pairs.

Explicit locations are useful when several functions, labels, or file names have 
the same name (base name for files) in the program’s sources. In these cases, 
explicit locations point to the source line you meant more accurately and 
unambiguously. Also, using explicit locations might be faster in large programs.

For example, the linespec ‘foo:bar’ may refer to a function `bar` defined in the 
file named foo or the label `bar` in a function named `foo`. GDB must search 
either the file system or the symbol table to know.

The list of valid explicit location options is summarized in the following table:

- <code>-source *filename*</code>  
   The value specifies the source file name. 

   To differentiate between files with the same base name, prepend as many 
   directories as is necessary to uniquely identify the desired file, e.g., 
   foo/bar/baz.c. Otherwise GDB will use the first file it finds with the given 
   base name. This option requires the use of either `-function` or `-line`.

- <code>-function *function*</code>  
   The value specifies the name of a function.

   Operations on function locations unmodified by other options (such as
   `-label` or `-line`) refer to the line that begins the body of the function. 
   In C, for example, this is the line with the open brace.

   By default, in C++ and Ada, function is interpreted as specifying all 
   functions named function in all scopes. For C++, this means in all namespaces 
   and classes. For Ada, this means in all packages.

   For example, assuming a program with C++ symbols named `A::B::func` and 
   `B::func`, both commands ‘break -function func’ and ‘break -function B::func’ 
   set a breakpoint on both symbols.

   You can use the `-qualified` flag to override this (see below).

- `-qualified`  
   This flag makes GDB interpret a function name specified with -function as a 
   complete fully-qualified name.

   For example, assuming a C`++` program with symbols named `A::B::func` and 
   `B::func`, the ‘break -qualified -function B::func’ command sets a breakpoint on 
   `B::func`, only.

   (Note: the -qualified option can precede a linespec as well (see [Linespec Locations](Linespec-Locations.html#Linespec-Locations)), 
   so the particular example above could be simplified as 
   ‘break -qualified B::func’.)

- <code>-label <i>label</i></code>  
   The value specifies the name of a label.

   When the function name is not specified, the label is searched in the 
   function of the currently selected stack frame.

- <code>-line <i>number</i></code>  
   The value specifies a line offset for the location. 

   The offset may either be absolute (`-line 3`) or relative (`-line +3`), 
   depending on the command. When specified without any other options, the line 
   offset is relative to the current line.

Explicit location options may be abbreviated by omitting any non-unique trailing 
characters from the option name, e.g., ‘break -s main.c -li 3’.


### 9.2.3 Address Locations

*Address locations* indicate a specific program address. They have the 
generalized form \*address.

For line-oriented commands, such as `list` and `edit`, this specifies a source 
line that contains address. For `break` and other breakpoint-oriented commands, 
this can be used to set breakpoints in parts of your program which do not have 
debugging information or source files.

Here address may be any expression valid in the current working language (see 
[working language](Languages.html#Languages)) that specifies a code address. In 
addition, as a convenience, GDB extends the semantics of expressions used in 
locations to cover several situations that frequently occur during debugging. 
Here are the various forms of address:

- `expression`  
   Any expression valid in the current working language.

- `funcaddr`  
   An address of a function or procedure derived from its name. In C, C++, 
   Objective-C, Fortran, minimal, and assembly, this is simply the function’s name 
   function (and actually a special case of a valid expression). In Pascal and 
   Modula-2, this is `&function`. In Ada, this is `function'Address` (although the 
   Pascal form also works).

   This form specifies the address of the function’s first instruction, before the 
   stack frame and arguments have been set up.

- `'filename':funcaddr`  
   Like `funcaddr` above, but also specifies the name of the source file 
   explicitly. This is useful if the name of the function does not specify the 
   function unambiguously, e.g., if there are several functions with identical 
   names in different source files.


## 9.3 Editing Source Files

To edit the lines in a source file, use the `edit` command. The editing program 
of your choice is invoked with the current line set to the active line in the 
program. Alternatively, there are several ways to specify what part of the file 
you want to print if you want to see other parts of the program:

- `edit location`  
   Edit the source file specified by `location`. 

   Editing starts at that location, e.g., at the specified source line of the 
   specified file. See [Specify Location](#92-specifying-a-location), 
   for all the possible forms of the location argument; here are the forms of the 
   `edit` command most commonly used:

- `edit number`  
   Edit the current source file with `number` as the active line number.

- `edit function`  
   Edit the file containing `function` at the beginning of its definition.


### 9.3.1 Choosing your Editor

You can customize GDB to use any editor you want [<sup>9</sup>](#FOOT9). By 
default, it is /bin/ex, but you can change this by setting the environment 
variable `EDITOR` before using GDB. For example, to configure GDB to use the 
`vi` editor, you could use these commands with the `sh` shell:

```sh
EDITOR=/usr/bin/vi
export EDITOR
gdb …
```

or in the `csh` shell,

```sh
setenv EDITOR /usr/bin/vi
gdb …
```

------------------------------------------------------------------------

### Footnotes

### [(9)](#DOCF9)

The only restriction is that your editor (say `ex`), recognizes the following 
command-line syntax:

```sh
ex +number file
```

The optional numeric value `+number` specifies the number of the line in the file 
where to start editing.


## 9.4 Searching Source Files

There are two commands for searching through the current source file for a 
regular expression.

- `forward-search regexp`  
  `search regexp`  
   The command `forward-search regexp` checks each line, starting with the one 
   following the last line listed, for a match for regexp. 

   It lists the line that is found. You can use the synonym `search regexp` or 
   abbreviate the command name as `fo`.

- `reverse-search regexp`  
   The command `reverse-search regexp` checks each line, starting with the one 
   before the last line listed and going backward, for a match for regexp. 

   It lists the line that is found. You can abbreviate this command as `rev`.


## 9.5 Specifying Source Directories

Executable programs sometimes do not record the directories of the source files 
from which they were compiled, just the names. Even when they do, the 
directories could be moved between the compilation and your debugging session. 
GDB has a list of directories to search for source files; this is called the 
*source path*. Each time GDB wants a source file, it tries all the directories 
in the list, in the order they are present in the list, until it finds a file 
with the desired name.

For example, suppose an executable references the file `/usr/src/foo-1.0/lib/foo.c`, 
and our source path is `/mnt/cross`. The file is first looked up literally; 
if this fails, `/mnt/cross/usr/src/foo-1.0/lib/foo.c` is tried; if this fails, 
`/mnt/cross/foo.c` is opened; if this fails, an error message is printed. GDB does 
not look up the parts of the source file name, such as 
`/mnt/cross/src/foo-1.0/lib/foo.c`. Likewise, the subdirectories of the source 
path are not searched: if the source path is `/mnt/cross`, and the binary refers
to foo.c, GDB would not find it under `/mnt/cross/usr/src/foo-1.0/lib`.

Plain file names, relative file names with leading directories, file names 
containing dots, etc. are all treated as described above; for instance, if the 
source path is `/mnt/cross`, and the source file is recorded as `../lib/foo.c`, GDB 
would first try `../lib/foo.c`, then `/mnt/cross/../lib/foo.c`, and after that—
`/mnt/cross/foo.c`.

Note that the executable search path is *not* used to locate the source files.

*Whenever you reset or rearrange the source path, GDB clears out any information 
it has cached about where source files are found and where each line is in the 
file.*

When you start GDB, its source path includes only `cdir` and `cwd`, in that 
order. To add other directories, use the `directory` command.

The search path is used to find both program source files and GDB script files 
(read using the `-command` option and `source` command).

In addition to the source path, GDB provides a set of commands that manage a 
list of source path substitution rules. A *substitution rule* specifies how to 
rewrite source directories stored in the program’s debug information in case the 
sources were moved to a different directory between compilation and debugging. A 
rule is made of two strings, the first specifying what needs to be rewritten in 
the path, and the second specifying how it should be rewritten. In 
[set substitute-path](#set-substitute_002dpath), we name these two parts *from* and *to* 
respectively. GDB does a simple string replacement of *from* with *to* at the start 
of the directory part of the source file name, and uses that result instead of 
the original file name to look up the sources.

Using the previous example, suppose the `foo-1.0` tree has been moved from 
`/usr/src` to `/mnt/cross`, then you can tell GDB to replace `/usr/src` in all source path 
names with `/mnt/cross`. The first lookup will then be `/mnt/cross/foo-1.0/lib/foo.c`
in place of the original location of `/usr/src/foo-1.0/lib/foo.c`. To define 
a source path substitution rule, use the `set substitute-path` command (see [set 
substitute-path](#set-substitute_002dpath)).

To avoid unexpected substitution results, a rule is applied only if the from 
part of the directory name ends at a directory separator. For instance, a rule 
substituting `/usr/source` into `/mnt/cross` will be applied to `/usr/source/foo-1.0` 
but not to `/usr/sourceware/foo-2.0`. And because the substitution is applied only 
at the beginning of the directory name, this rule will not be applied to 
`/root/usr/source/baz.c` either.

In many cases, you can achieve the same result using the `directory` command. 
However, `set substitute-path` can be more efficient in the case where the 
sources are organized in a complex tree with multiple subdirectories. With the 
`directory` command, you need to add each subdirectory of your project. If you 
moved the entire tree while preserving its internal organization, then `set 
substitute-path` allows you to direct the debugger to all the sources with one 
single command.

`set substitute-path` is also more than just a shortcut command. The source path 
is only used if the file at the original location no longer exists. On the other 
hand, `set substitute-path` modifies the debugger behavior to look at the 
rewritten location instead. So, if for any reason a source file that is not 
relevant to your executable is located at the original location, a substitution 
rule is the only method available to point GDB at the new location.

You can configure a default source path substitution rule by configuring GDB 
with the ‘--with-relocated-sources=dir’ option. The dir should be the name of a 
directory under GDB’s configured prefix (set with ‘--prefix’ or 
‘--exec-prefix’), and directory names in debug information under dir will be adjusted 
automatically if the installed GDB is moved to a new location. This is useful if 
GDB, libraries or executables with debug information and corresponding source 
code are being moved together.

- `directory dirname …`  
  `dir dirname …`  
   Add directory `dirname` to the front of the source path. 

   Several directory names may be given to this command, separated by ‘:’ (‘;’ 
   on MS-DOS and MS-Windows, where ‘:’ usually appears as part of absolute file 
   names) or whitespace. You may specify a directory that is already in the source 
   path; this moves it forward, so GDB searches it sooner.

   You can use the string ‘$cdir’ to refer to the compilation directory (if one 
   is recorded), and ‘$cwd’ to refer to the current working directory. ‘$cwd’ is 
   not the same as ‘.’—the former tracks the current working directory as it 
   changes during your GDB session, while the latter is immediately expanded to the 
   current directory at the time you add an entry to the source path.

- `directory`  
   Reset the source path to its default value (‘$cdir:$cwd’ on Unix systems). 
   This requires confirmation.

- `set directories path-list`  
   Set the source path to path-list. ‘$cdir:$cwd’ are added if missing.

- `show directories`  
   Print the source path: show which directories it contains.

- `set substitute-path from to`  
   Define a source path substitution rule, and add it at the end of the current 
   list of existing substitution rules. 

   If a rule with the same from was already defined, then the old rule is also deleted.

   For example, if the file `/foo/bar/baz.c` was moved to `/mnt/cross/baz.c`, then 
   the command

   ```gdb
   (gdb) set substitute-path /foo/bar /mnt/cross
   ```

   will tell GDB to replace `/foo/bar` with `/mnt/cross`, which will allow GDB 
   to find the file baz.c even though it was moved.

   In the case when more than one substitution rule have been defined, the rules 
   are evaluated one by one in the order where they have been defined. The first 
   one matching, if any, is selected to perform the substitution.

   For instance, if we had entered the following commands:

   ```gdb
   (gdb) set substitute-path /usr/src/include /mnt/include
   (gdb) set substitute-path /usr/src /mnt/src
   ```

   GDB would then rewrite `/usr/src/include/defs.h` into `/mnt/include/defs.h` by 
   using the first rule. However, it would use the second rule to rewrite 
   `/usr/src/lib/foo.c` into `/mnt/src/lib/foo.c`.

- `unset substitute-path [path]`  
   If a path is specified, search the current list of substitution rules for a 
   rule that would rewrite that path. 

   Delete that rule if found. A warning is emitted by the debugger if no rule 
   could be found.

   If no path is specified, then all substitution rules are deleted.

- `show substitute-path [path]`  
   If a path is specified, then print the source path substitution rule which 
   would rewrite that path, if any.

   If no path is specified, then print all existing source path substitution rules.

If your source path is cluttered with directories that are no longer of 
interest, GDB may sometimes cause confusion by finding the wrong versions of 
source. You can correct the situation as follows:

1.  Use `directory` with no argument to reset the source path to its default value.

2.  Use `directory` with suitable arguments to reinstall the directories you 
want in the source path. You can add all the directories in one command.


## 9.6 Source and Machine Code

You can use the command `info line` to map source lines to program addresses 
(and vice versa), and the command `disassemble` to display a range of addresses 
as machine instructions. You can use the command `set disassemble-next-line` to 
set whether to disassemble next source line when execution stops. When run under 
GNU Emacs mode, the `info line` command causes the arrow to point to the line 
specified. Also, `info line` prints addresses in symbolic form as well as hex.

- `info line`  
  `info line location`  
   Print the starting and ending addresses of the compiled code for source line location. 

   You can specify source lines in any of the ways documented in [Specify Location](Specify-Location.html#Specify-Location). 
   With no location information about the current source line is printed.

   For example, we can use `info line` to discover the location of the object 
   code for the first line of function `m4_changequote`:

   ```gdb
   (gdb) info line m4_changequote
   Line 895 of "builtin.c" starts at pc 0x634c <m4_changequote> and \
        ends at 0x6350 <m4_changequote+4>.
   ```

   We can also inquire (using `*addr` as the form for location) what source line 
   covers a particular address:

   ```gdb
   (gdb) info line *0x63ff
   Line 926 of "builtin.c" starts at pc 0x63e4 <m4_changequote+152> and \
        ends at 0x6404 <m4_changequote+184>.
   ```

   After `info line`, the default address for the `x` command is changed to the 
   starting address of the line, so that ‘x/i’ is sufficient to begin examining the 
   machine code (see [Examining Memory](Memory.html#Memory)). Also, this address is 
   saved as the value of the convenience variable `$_` (see [Convenience Variables]
   (Convenience-Vars.html#Convenience-Vars)).

   After `info line`, using `info line` again without specifying a location will 
   display information about the next source line.

- `disassemble`  
  `disassemble /m`  
  `disassemble /s`  
  `disassemble /r`  
   This specialized command dumps a range of memory as machine instructions. 

   It can also print mixed source+disassembly by specifying the `/m` or `/s` modifier 
   and print the raw instructions in hex as well as in symbolic form by specifying 
   the `/r` modifier. The default memory range is the function surrounding the 
   program counter of the selected frame. A single argument to this command is a 
   program counter value; GDB dumps the function surrounding this value. When two 
   arguments are given, they should be separated by a comma, possibly surrounded by 
   whitespace. The arguments specify a range of addresses to dump, in one of two 
   forms:

   - `start,end`  
      the addresses from start (inclusive) to end (exclusive)

   - `start,+length`  
      the addresses from start (inclusive) to `start+length` (exclusive).

   When 2 arguments are specified, the name of the function is also printed 
   (since there could be several functions in the given range).

   The argument(s) can be any expression yielding a numeric value, such as 
   ‘0x32c4’, ‘&main+10’ or ‘$pc - 8’.

   If the range of memory being disassembled contains current program counter, 
   the instruction at that location is shown with a `=>` marker.

The following example shows the disassembly of a range of addresses of HP PA-RISC 2.0 code:

```gdb
(gdb) disas 0x32c4, 0x32e4
Dump of assembler code from 0x32c4 to 0x32e4:
   0x32c4 <main+204>:      addil 0,dp
   0x32c8 <main+208>:      ldw 0x22c(sr0,r1),r26
   0x32cc <main+212>:      ldil 0x3000,r31
   0x32d0 <main+216>:      ble 0x3f8(sr4,r31)
   0x32d4 <main+220>:      ldo 0(r31),rp
   0x32d8 <main+224>:      addil -0x800,dp
   0x32dc <main+228>:      ldo 0x588(r1),r26
   0x32e0 <main+232>:      ldil 0x3000,r31
End of assembler dump.
```

Here is an example showing mixed source+assembly for Intel x86 with `/m` or 
`/s`, when the program is stopped just after function prologue in a non-
optimized function with no inline code.

```gdb
(gdb) disas /m main
Dump of assembler code for function main:
5       {
   0x08048330 <+0>:    push   %ebp
   0x08048331 <+1>:    mov    %esp,%ebp
   0x08048333 <+3>:    sub    $0x8,%esp
   0x08048336 <+6>:    and    $0xfffffff0,%esp
   0x08048339 <+9>:    sub    $0x10,%esp

6         printf ("Hello.\n");
=> 0x0804833c <+12>:   movl   $0x8048440,(%esp)
   0x08048343 <+19>:   call   0x8048284 <puts@plt>

7         return 0;
8       }
   0x08048348 <+24>:   mov    $0x0,%eax
   0x0804834d <+29>:   leave
   0x0804834e <+30>:   ret

End of assembler dump.
```

The `/m` option is deprecated as its output is not useful when there is either 
inlined code or re-ordered code. The `/s` option is the preferred choice. Here 
is an example for AMD x86-64 showing the difference between `/m` output and `/s` 
output. This example has one inline function defined in a header file, and the 
code is compiled with ‘-O2’ optimization. Note how the `/m` output is missing 
the disassembly of several instructions that are present in the `/s` output.

**foo.h:**

```c
int
foo (int a)
{
  if (a < 0)
    return a * 2;
  if (a == 0)
    return 1;
  return a + 10;
}
```

**foo.c:**

```c
#include "foo.h"
volatile int x, y;
int
main ()
{
  x = foo (y);
  return 0;
}
```

```gdb
(gdb) disas /m main
Dump of assembler code for function main:
5   {

6     x = foo (y);
   0x0000000000400400 <+0>:   mov    0x200c2e(%rip),%eax # 0x601034 <y>
   0x0000000000400417 <+23>:  mov    %eax,0x200c13(%rip) # 0x601030 <x>

7     return 0;
8   }
   0x000000000040041d <+29>:  xor    %eax,%eax
   0x000000000040041f <+31>:  retq
   0x0000000000400420 <+32>:  add    %eax,%eax
   0x0000000000400422 <+34>:  jmp    0x400417 <main+23>

End of assembler dump.
(gdb) disas /s main
Dump of assembler code for function main:
foo.c:
5   {
6     x = foo (y);
   0x0000000000400400 <+0>:   mov    0x200c2e(%rip),%eax # 0x601034 <y>

foo.h:
4     if (a < 0)
   0x0000000000400406 <+6>:   test   %eax,%eax
   0x0000000000400408 <+8>:   js     0x400420 <main+32>

6     if (a == 0)
7       return 1;
8     return a + 10;
   0x000000000040040a <+10>:  lea    0xa(%rax),%edx
   0x000000000040040d <+13>:  test   %eax,%eax
   0x000000000040040f <+15>:  mov    $0x1,%eax
   0x0000000000400414 <+20>:  cmovne %edx,%eax

foo.c:
6     x = foo (y);
   0x0000000000400417 <+23>:  mov    %eax,0x200c13(%rip) # 0x601030 <x>

7     return 0;
8   }
   0x000000000040041d <+29>:  xor    %eax,%eax
   0x000000000040041f <+31>:  retq

foo.h:
5       return a * 2;
   0x0000000000400420 <+32>:  add    %eax,%eax
   0x0000000000400422 <+34>:  jmp    0x400417 <main+23>
End of assembler dump.
```

Here is another example showing raw instructions in hex for AMD x86-64,

```gdb
(gdb) disas /r 0x400281,+10
Dump of assembler code from 0x400281 to 0x40028b:
   0x0000000000400281:  38 36  cmp    %dh,(%rsi)
   0x0000000000400283:  2d 36 34 2e 73 sub    $0x732e3436,%eax
   0x0000000000400288:  6f     outsl  %ds:(%rsi),(%dx)
   0x0000000000400289:  2e 32 00       xor    %cs:(%rax),%al
End of assembler dump.
```

Addresses cannot be specified as a location (see [Specify Location](Specify-Location.html#Specify-Location)). 
So, for example, if you want to disassemble 
function `bar` in file `foo.c`, you must type ‘disassemble 'foo.c'::bar’ and not 
‘disassemble foo.c:bar’.

Some architectures have more than one commonly-used set of instruction mnemonics 
or other syntax.

For programs that were dynamically linked and use shared libraries, instructions 
that call functions or branch to locations in the shared libraries might show a 
seemingly bogus location—it’s actually a location of the relocation table. On 
some architectures, GDB might be able to resolve these to actual function names.

- `set disassembler-options option1[,option2…]`  
   This command controls the passing of target specific information to the 
   disassembler. 

   For a list of valid options, please refer to the `-M`/`--disassembler-
   options` section of the ‘objdump’ manual and/or the output of `objdump --help` 
   (see [objdump](http://sourceware.org/binutils/docs/binutils/objdump.html#objdump) 
   in The GNU Binary Utilities). The default value is the empty string.

   If it is necessary to specify more than one disassembler option, then 
   multiple options can be placed together into a comma separated list. Currently 
   this command is only supported on targets ARM, MIPS, PowerPC and S/390.

- `show disassembler-options`  
   Show the current setting of the disassembler options.

- `set disassembly-flavor instruction-set`  
   Select the instruction set to use when disassembling the program via the 
   `disassemble` or `x/i` commands.

   Currently this command is only defined for the Intel x86 family. You can set 
   instruction-set to either `intel` or `att`. The default is `att`, the AT&T 
   flavor used by default by Unix assemblers for x86-based targets.

- `show disassembly-flavor`  
   Show the current setting of the disassembly flavor.

- `set disassemble-next-line`  
  `show disassemble-next-line`  
   Control whether or not GDB will disassemble the next source line or 
   instruction when execution stops. 

   If ON, GDB will display disassembly of the next source line when execution of 
   the program being debugged stops. This is *in addition* to displaying the source 
   line itself, which GDB always does if possible. If the next source line cannot 
   be displayed for some reason (e.g., if GDB cannot find the source file, or 
   there’s no line info in the debug info), GDB will display disassembly of the 
   next *instruction* instead of showing the next source line. If AUTO, GDB will 
   display disassembly of next instruction only if the source line cannot be 
   displayed. This setting causes GDB to display some feedback when you step 
   through a function with no line info or whose source file is unavailable. The 
   default is OFF, which means never display the disassembly of the next line or 
   instruction.

