# 10 Examining Data


* [10.1 Expressions](#101-expressions)
* [10.2 Ambiguous Expressions](#102-ambiguous-expressions)
* [10.3 Program Variables](#103-program-variables)
* [10.4 Artificial Arrays](#104-artificial-arrays)
* [10.5 Output Formats](#105-output-formats)
        * [Footnotes](#footnotes)
    * [(10)](#10)
* [10.6 Examining Memory](#106-examining-memory)
* [10.7 Automatic Display](#107-automatic-display)
* [10.8 Print Settings](#108-print-settings)
* [10.9 Pretty Printing](#109-pretty-printing)
    * [10.9.1 Pretty-Printer Introduction](#1091-pretty-printer-introduction)
    * [10.9.2 Pretty-Printer Example](#1092-pretty-printer-example)
    * [10.9.3 Pretty-Printer Commands](#1093-pretty-printer-commands)
* [10.10 Value History](#1010-value-history)
* [10.11 Convenience Variables](#1011-convenience-variables)
* [10.12 Convenience Functions](#1012-convenience-functions)
* [10.13 Registers](#1013-registers)
        * [Footnotes](#footnotes-1)
    * [(11)](#11)
* [10.14 Floating Point Hardware](#1014-floating-point-hardware)
* [10.15 Vector Unit](#1015-vector-unit)
* [10.16 Operating System Auxiliary Information](#1016-operating-system-auxiliary-information)
* [10.17 Memory Region Attributes](#1017-memory-region-attributes)
        * [10.17.1 Attributes](#10171-attributes)
        * [10.17.1.1 Memory Access Mode](#101711-memory-access-mode)
        * [10.17.1.2 Memory Access Size](#101712-memory-access-size)
        * [10.17.1.3 Data Cache](#101713-data-cache)
        * [10.17.2 Memory Access Checking](#10172-memory-access-checking)
* [10.18 Copy Between Memory and a File](#1018-copy-between-memory-and-a-file)
* [10.19 How to Produce a Core File from Your Program](#1019-how-to-produce-a-core-file-from-your-program)
* [10.20 Character Sets](#1020-character-sets)
* [10.21 Caching Data of Targets](#1021-caching-data-of-targets)
        * [Footnotes](#footnotes-2)
    * [(12)](#12)
* [10.22 Search Memory](#1022-search-memory)
* [10.23 Value Sizes](#1023-value-sizes)


The usual way to examine data in your program is with the `print`
command (abbreviated `p`), or its synonym `inspect`.  It
evaluates and prints the value of an expression of the language your
program is written in (see [Using GDB with Different Languages](Languages.html#Languages)).  
It may also print the expression using a Python-based pretty-printer (see [Pretty Printing](Pretty-Printing.html#Pretty-Printing)).

- `print expr`
  `print /fexpr`

   expr is an expression (in the source language).  By default the
   value of expr is printed in a format appropriate to its data type;
   you can choose a different format by specifying &lsquo;/f&rsquo;, where
   f is a letter specifying the format; see [Output Formats](Output-Formats.html#Output-Formats).

- `print`
  `print /f`

   If you omit expr, GDB displays the last value again (from the
   *value history*; see [Value History](Value-History.html#Value-History)).  This allows you to
   conveniently inspect the same value in an alternative format.

   A more low-level way of examining data is with the `x` command.
   It examines data in memory at a specified address and prints it in a
   specified format.  See [Examining Memory](Memory.html#Memory).

   If you are interested in information about types, or about how the
   fields of a struct or a class are declared, use the `ptype exp`
   command rather than `print`.  See [Examining the Symbol Table](Symbols.html#Symbols).

   Another way of examining values of expressions and type information is
   through the Python extension command `explore` (available only if
   the GDB build is configured with `--with-python`).  It
   offers an interactive way to start at the highest level (or, the most
   abstract level) of the data type of an expression (or, the data type
   itself) and explore all the way down to leaf scalar values/fields
   embedded in the higher level data types.

- `explore arg`

   arg is either an expression (in the source language), or a type
   visible in the current context of the program being debugged.

   The working of the `explore` command can be illustrated with an
   example.  If a data type `struct ComplexStruct` is defined in your
   C program as

   ```c
   struct SimpleStruct
   {
     int i;
     double d;
   };
   
   struct ComplexStruct
   {
     struct SimpleStruct *ss_p;
     int arr[10];
   };
   ```

   followed by variable declarations as

   ```c
   struct SimpleStruct ss = { 10, 1.11 };
   struct ComplexStruct cs = { &ss, { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 } };
   ```
   then, the value of the variable `cs` can be explored using the
   `explore` command as follows.

   ```
   (gdb) explore cs
   The value of `cs' is a struct/class of type `struct ComplexStruct' with
   the following fields:
    
   ss_p = <Enter 0 to explore this field of type `struct SimpleStruct *'>
   arr = <Enter 1 to explore this field of type `int [10]'>
   ```

   Enter the field number of choice:

Since the fields of `cs` are not scalar values, you are being
prompted to chose the field you want to explore.  Let's say you choose
the field `ss_p` by entering `0`.  Then, since this field is a
pointer, you will be asked if it is pointing to a single value.  From
the declaration of `cs` above, it is indeed pointing to a single
value, hence you enter `y`.  If you enter `n`, then you will
be asked if it were pointing to an array of values, in which case this
field will be explored as if it were an array.

    `cs.ss_p' is a pointer to a value of type `struct SimpleStruct'
    Continue exploring it as a pointer to a single value [y/n]: y
    The value of `*(cs.ss_p)' is a struct/class of type `struct
    SimpleStruct' with the following fields:
    
      i = 10 .. (Value of type `int')
      d = 1.1100000000000001 .. (Value of type `double')
    
    Press enter to return to parent value:

If the field `arr` of `cs` was chosen for exploration by
entering `1` earlier, then since it is as array, you will be
prompted to enter the index of the element in the array that you want
to explore.

    `cs.arr` is an array of `int`.
    Enter the index of the element you want to explore in `cs.arr': 5
    
    `(cs.arr)[5]' is a scalar value of type `int'.
    
    (cs.arr)[5] = 4
    
    Press enter to return to parent value: 

In general, at any stage of exploration, you can go deeper towards the
leaf values by responding to the prompts appropriately, or hit the
return key to return to the enclosing data structure (the higher
level data structure).

Similar to exploring values, you can use the `explore` command to
explore types.  Instead of specifying a value (which is typically a
variable name or an expression valid in the current context of the
program being debugged), you specify a type name.  If you consider the
same example as above, your can explore the type
`struct ComplexStruct` by passing the argument
`struct ComplexStruct` to the `explore` command.

    (gdb) explore struct ComplexStruct

By responding to the prompts appropriately in the subsequent interactive
session, you can explore the type `struct ComplexStruct` in a
manner similar to how the value `cs` was explored in the above
example.

The `explore` command also has two sub-commands,
`explore value` and `explore type`. The former sub-command is
a way to explicitly specify that value exploration of the argument is
being invoked, while the latter is a way to explicitly specify that type
exploration of the argument is being invoked.

- `explore value expr`  
   This sub-command of `explore` explores the value of the
   expression expr (if expr is an expression valid in the
   current context of the program being debugged).  The behavior of this
   command is identical to that of the behavior of the `explore`
   command being passed the argument expr.

- `explore type arg`  
   This sub-command of `explore` explores the type of arg (if
   arg is a type visible in the current context of program being
   debugged), or the type of the value/expression arg (if arg
   is an expression valid in the current context of the program being
   debugged).  If arg is a type, then the behavior of this command is
   identical to that of the `explore` command being passed the
   argument arg.  If arg is an expression, then the behavior of
   this command will be identical to that of the `explore` command
   being passed the type of arg as the argument.


## 10.1 Expressions

`print` and many other GDB commands accept an expression and
compute its value.  Any kind of constant, variable or operator defined
by the programming language you are using is valid in an expression in
GDB.  This includes conditional expressions, function calls,
casts, and string constants.  It also includes preprocessor macros, if
you compiled your program to include this information; see
[Compilation](Compilation.html#Compilation).

GDB supports array constants in expressions input by
the user.  The syntax is {element, element&hellip;}.  For example,
you can use the command `print {1, 2, 3}` to create an array
of three integers.  If you pass an array to a function or assign it
to a program variable, GDB copies the array to memory that
is `malloc`ed in the target program.

Because C is so widespread, most of the expressions shown in examples in
this manual are in C.  See [Using GDB with Different
Languages](Languages.html#Languages), for information on how to use expressions in other
languages.

In this section, we discuss operators that you can use in GDB
expressions regardless of your programming language.

Casts are supported in all languages, not just in C, because it is so
useful to cast a number into a pointer in order to examine a structure
at that address in memory.

GDB supports these operators, in addition to those common
to programming languages:

- `@`
   `@` is a binary operator for treating parts of memory as arrays.
   See [Artificial Arrays](Arrays.html#Arrays), for more information.

- `::`
   `::` allows you to specify a variable in terms of the file or
   function where it is defined.  See [Program Variables](Variables.html#Variables).

- `{type} addr`
   Refers to an object of type type stored at address addr in
   memory.  The address addr may be any expression whose value is
   an integer or pointer (but parentheses are required around binary
   operators, just as in a cast).  This construct is allowed regardless
   of what kind of data is normally supposed to reside at addr.


## 10.2 Ambiguous Expressions

Expressions can sometimes contain some ambiguous elements.  For instance,
some programming languages (notably Ada, C++ and Objective-C) permit
a single function name to be defined several times, for application in
different contexts.  This is called *overloading*.  Another example
involving Ada is generics.  A *generic package* is similar to C++
templates and is typically instantiated several times, resulting in
the same function name being defined in different contexts.

In some cases and depending on the language, it is possible to adjust
the expression to remove the ambiguity.  For instance in C++, you
can specify the signature of the function you want to break on, as in
break function(types).  In Ada, using the fully
qualified name of your function often makes the expression unambiguous
as well.

When an ambiguity that needs to be resolved is detected, the debugger
has the capability to display a menu of numbered choices for each
possibility, and then waits for the selection with the prompt &lsquo;>&rsquo;.
The first option is always &lsquo;[0] cancel&rsquo;, and typing 0 RET
aborts the current command.  If the command in which the expression was
used allows more than one choice to be selected, the next option in the
menu is &lsquo;[1] all&rsquo;, and typing 1 RET selects all possible
choices.

For example, the following session excerpt shows an attempt to set a
breakpoint at the overloaded symbol `String::after`.
We choose three particular definitions of that function name:

    (gdb) b String::after
    [0] cancel
    [1] all
    [2] file:String.cc; line number:867
    [3] file:String.cc; line number:860
    [4] file:String.cc; line number:875
    [5] file:String.cc; line number:853
    [6] file:String.cc; line number:846
    [7] file:String.cc; line number:735
    > 2 4 6
    Breakpoint 1 at 0xb26c: file String.cc, line 867.
    Breakpoint 2 at 0xb344: file String.cc, line 875.
    Breakpoint 3 at 0xafcc: file String.cc, line 846.
    Multiple breakpoints were set.
    Use the "delete" command to delete unwanted
     breakpoints.
    (gdb)

- `set multiple-symbols mode`  
   This option allows you to adjust the debugger behavior when an expression
   is ambiguous.

   By default, mode is set to `all`.  If the command with which
   the expression is used allows more than one choice, then GDB
   automatically selects all possible choices.  For instance, inserting
   a breakpoint on a function using an ambiguous name results in a breakpoint
   inserted on each possible match.  However, if a unique choice must be made,
   then GDB uses the menu to help you disambiguate the expression.
   For instance, printing the address of an overloaded function will result
   in the use of the menu.

   When mode is set to `ask`, the debugger always uses the menu
   when an ambiguity is detected.

   Finally, when mode is set to `cancel`, the debugger reports
   an error due to the ambiguity and the command is aborted.

- `show multiple-symbols`  
   Show the current value of the `multiple-symbols` setting.


## 10.3 Program Variables

The most common kind of expression to use is the name of a variable
in your program.

Variables in expressions are understood in the selected stack frame
(see [Selecting a Frame](Selection.html#Selection)); they must be either:

-  global (or file-static)

or

-  visible according to the scope rules of the
   programming language from the point of execution in that frame

This means that in the function

```c
foo (a)
     int a;
{
  bar (a);
  {
    int b = test ();
    bar (b);
  }
}
```

you can examine and use the variable `a` whenever your program is
executing within the function `foo`, but you can only use or
examine the variable `b` while your program is executing inside
the block where `b` is declared.

There is an exception: you can refer to a variable or function whose
scope is a single source file even if the current execution point is not
in this file.  But it is possible to have more than one such variable or
function with the same name (in different source files).  If that
happens, referring to that name has unpredictable effects.  If you wish,
you can specify a static variable in a particular function or file by
using the colon-colon (`::`) notation:

    file::variablefunction::variable

Here file or function is the name of the context for the
static variable.  In the case of file names, you can use quotes to
make sure GDB parses the file name as a single word&mdash;for example,
to print a global value of `x` defined in f2.c:

    (gdb) p 'f2.c'::x

The `::` notation is normally used for referring to
static variables, since you typically disambiguate uses of local variables
in functions by selecting the appropriate frame and using the
simple name of the variable.  However, you may also use this notation
to refer to local variables in frames enclosing the selected frame:

```c
void
foo (int a)
{
  if (a < 10)
    bar (a);
  else
    process (a);    /* Stop here */
}

int
bar (int a)
{
  foo (a + 5);
}
```

For example, if there is a breakpoint at the commented line,
here is what you might see
when the program stops after executing the call `bar(0)`:

    (gdb) p a
    $1 = 10
    (gdb) p bar::a
    $2 = 5
    (gdb) up 2
    #2  0x080483d0 in foo (a=5) at foobar.c:12
    (gdb) p a
    $3 = 5
    (gdb) p bar::a
    $4 = 0

These uses of &lsquo;::&rsquo; are very rarely in conflict with the very
similar use of the same notation in C++.  When they are in
conflict, the C++ meaning takes precedence; however, this can be
overridden by quoting the file or function name with single quotes.

For example, suppose the program is stopped in a method of a class
that has a field named `includefile`, and there is also an
include file named includefile that defines a variable,
`some_global`.

    (gdb) p includefile
    $1 = 23
    (gdb) p includefile::some_global
    A syntax error in expression, near `'.
    (gdb) p 'includefile'::some_global
    $2 = 27

> *Warning:* Occasionally, a local variable may appear to have the
> wrong value at certain points in a function&mdash;just after entry to a new
> scope, and just before exit.

You may see this problem when you are stepping by machine instructions.
This is because, on most machines, it takes more than one instruction to
set up a stack frame (including local variable definitions); if you are
stepping by machine instructions, variables may appear to have the wrong
values until the stack frame is completely built.  On exit, it usually
also takes more than one machine instruction to destroy a stack frame;
after you begin stepping through that group of instructions, local
variable definitions may be gone.

This may also happen when the compiler does significant optimizations.
To be sure of always seeing accurate values, turn off all optimization
when compiling.

Another possible effect of compiler optimizations is to optimize
unused variables out of existence, or assign variables to registers (as
opposed to memory addresses).  Depending on the support for such cases
offered by the debug info format used by the compiler, GDB
might not be able to display values for such local variables.  If that
happens, GDB will print a message like this:

    No symbol "foo" in current context.

To solve such problems, either recompile without optimizations, or use a
different debug info format, if the compiler supports several such
formats.  See [Compilation](Compilation.html#Compilation), for more information on choosing compiler
options.  See [C and C++](C.html#C), for more information about debug
info formats that are best suited to C++ programs.

If you ask to print an object whose contents are unknown to
GDB, e.g., because its data type is not completely specified
by the debug information, GDB will say &lsquo;<incomplete
type>&rsquo;.  See [incomplete type](Symbols.html#Symbols), for more about this.

If you try to examine or use the value of a (global) variable for
which GDB has no type information, e.g., because the program
includes no debug information, GDB displays an error message.
See [unknown type](Symbols.html#Symbols), for more about unknown types.  If you
cast the variable to its declared type, GDB gets the
variable&rsquo;s value using the cast-to type as the variable&rsquo;s type.  For
example, in a C program:

      (gdb) p var
      'var' has unknown type; cast it to its declared type
      (gdb) p (float) var
      $1 = 3.14

If you append @entry string to a function parameter name you get its
value at the time the function got called.  If the value is not available an
error message is printed.  Entry values are available only with some compilers.
Entry values are normally also printed at the function parameter list according
to [set print entry-values](Print-Settings.html#set-print-entry_002dvalues).

    Breakpoint 1, d (i=30) at gdb.base/entry-value.c:29
    29	  i++;
    (gdb) next
    30	  e (i);
    (gdb) print i
    $1 = 31
    (gdb) print i@entry
    $2 = 30

Strings are identified as arrays of `char` values without specified
signedness.  Arrays of either `signed char` or `unsigned char` get
printed as arrays of 1 byte sized integers.  `-fsigned-char` or
`-funsigned-char`GCC options have no effect as GDB
defines literal string type `"char"` as `char` without a sign.
For program code

    char var0[] = "A";
    signed char var1[] = "A";

You get during debugging

    (gdb) print var0
    $1 = "A"
    (gdb) print var1
    $2 = {65 'A', 0 '\0'}


## 10.4 Artificial Arrays

It is often useful to print out several successive objects of the
same type in memory; a section of an array, or an array of
dynamically determined size for which only a pointer exists in the
program.

You can do this by referring to a contiguous span of memory as an
*artificial array*, using the binary operator `@`.  The left
operand of `@` should be the first element of the desired array
and be an individual object.  The right operand should be the desired length
of the array.  The result is an array value whose elements are all of
the type of the left argument.  The first element is actually the left
argument; the second element comes from bytes of memory immediately
following those that hold the first element, and so on.  Here is an
example.  If a program says

```c
int *array = (int *) malloc (len * sizeof (int));
```

you can print the contents of `array` with

    p *array@len

The left operand of `@` must reside in memory.  Array values made
with `@` in this way behave just like other arrays in terms of
subscripting, and are coerced to pointers when used in expressions.
Artificial arrays most often appear in expressions via the value history
(see [Value History](Value-History.html#Value-History)), after printing one out.

Another way to create an artificial array is to use a cast.
This re-interprets a value as if it were an array.
The value need not be in memory:

    (gdb) p/x (short[2])0x12345678
    $1 = {0x1234, 0x5678}

As a convenience, if you leave the array length out (as in
`(type[])value`) GDB calculates the size to fill
the value (as `sizeof(value)/sizeof(type)`:

    (gdb) p/x (short[])0x12345678
    $2 = {0x1234, 0x5678}

Sometimes the artificial array mechanism is not quite enough; in
moderately complex data structures, the elements of interest may not
actually be adjacent&mdash;for example, if you are interested in the values
of pointers in an array.  One useful work-around in this situation is
to use a convenience variable (see [Convenience Variables](Convenience-Vars.html#Convenience-Vars)) 
as a counter in an expression that prints the first
interesting value, and then repeat that expression via RET.  For
instance, suppose you have an array `dtab` of pointers to
structures, and you are interested in the values of a field `fv`
in each structure.  Here is an example of what you might type:

    set $i = 0
    p dtab[$i++]->fv
    RETRET
    &hellip;


## 10.5 Output Formats

By default, GDB prints a value according to its data type.  Sometimes
this is not what you want.  For example, you might want to print a number
in hex, or a pointer in decimal.  Or you might want to view data in memory
at a certain address as a character string or as an instruction.  To do
these things, specify an *output format* when you print a value.

The simplest use of output formats is to say how to print a value
already computed.  This is done by starting the arguments of the
`print` command with a slash and a format letter.  The format
letters supported are:

- `x`
   Regard the bits of the value as an integer, and print the integer in
   hexadecimal.

- `d`
   Print as integer in signed decimal.

- `u`
   Print as integer in unsigned decimal.

- `o`
   Print as integer in octal.

- `t`
   Print as integer in binary.  The letter `t` stands for 'two'.
   [10](#FOOT10)

- `a`
   Print as an address, both absolute in hexadecimal and as an offset from
   the nearest preceding symbol.  You can use this format used to discover
   where (in what function) an unknown address is located:

   ```
   (gdb) p/a 0x54320
   $3 = 0x54320 <_initialize_vx+396>
       
   
   The command `info symbol 0x54320` yields similar results.
   See [info symbol](Symbols.html#Symbols).
   ```

- `c`
   Regard as an integer and print it as a character constant.  This
   prints both the numerical value and its character representation.  The
   character representation is replaced with the octal escape `\nnn`
   for characters outside the 7-bit ASCII range.

   Without this format, GDB displays `char`,
   `unsigned char`, and `signed char` data as character
   constants.  Single-byte members of vectors are displayed as integer
   data.

- `f`
   Regard the bits of the value as a floating point number and print
   using typical floating point syntax.

- `s`
   Regard as a string, if possible.  With this format, pointers to single-byte
   data are displayed as null-terminated strings and arrays of single-byte data
   are displayed as fixed-length strings.  Other values are displayed in their
   natural types.

   Without this format, GDB displays pointers to and arrays of
   `char`, `unsigned char`, and `signed char` as
   strings.  Single-byte members of a vector are displayed as an integer
   array.

- `z`
   Like `x` formatting, the value is treated as an integer and
   printed as hexadecimal, but leading zeros are printed to pad the value
   to the size of the integer type.

- `r`
   Print using the 'raw' formatting.  By default, GDB will
   use a Python-based pretty-printer, if one is available (see [Pretty Printing](Pretty-Printing.html#Pretty-Printing)).
   This typically results in a higher-level display of the
   value's contents.  The `r` format bypasses any Python
   pretty-printer which might exist.

For example, to print the program counter in hex (see [Registers](Registers.html#Registers)), type

    p/x $pc

Note that no space is required before the slash; this is because command
names in GDB cannot contain a slash.

To reprint the last value in the value history with a different format,
you can use the `print` command with just a format and no
expression.  For example, `p/x` reprints the last value in hex.

---

### Footnotes


`b` cannot be used because these format letters are also
used with the `x` command, where `b` stands for 'byte';
see [Examining Memory](Memory.html#Memory).


## 10.6 Examining Memory

You can use the command `x` (for 'eXamine') to examine memory in
any of several formats, independently of your program's data types.

- `x/nfuaddr`
  `x addr`
  `x`
   Use the `x` command to examine memory.

`n`, `f`, and `u` *are all optional parameters* that specify how
much memory to display and how to format it; `addr` is an
expression giving the address where you want to start displaying memory.
If you use defaults for `nfu`, you need not type the slash `/`.
Several commands set convenient defaults for addr.

- `n`, *the repeat count*
   The repeat count is a decimal integer; the default is 1.  It specifies
   how much memory (counting by units u) to display.  If a negative
   number is specified, memory is examined backward from addr.

- `f`, *the display format*
   The display format is one of the formats used by `print`
   (`x`, `d`, `u`, `o`, `t`, `a`, `c`,
   `f`, `s`), and in addition `i` (for machine instructions).
   The default is `x` (hexadecimal) initially.  The default changes
   each time you use either `x` or `print`.

- `u`, *the unit size*
      The unit size is any of

      - `b`
         Bytes.

      - `h`
         Halfwords (two bytes).

      - `w`
         Words (four bytes).  This is the initial default.

      - `g`
         Giant words (eight bytes).

   Each time you specify a unit size with `x`, that size becomes the
   default unit the next time you use `x`.  For the `i` format,
   the unit size is ignored and is normally not written.  For the `s` format,
   the unit size defaults to `b`, unless it is explicitly given.
   Use x /hs to display 16-bit char strings and x /ws to display
   32-bit strings.  The next use of x /s will again display 8-bit strings.
   Note that the results depend on the programming language of the
   current compilation unit.  If the language is C, the `s`
   modifier will use the UTF-16 encoding while `w` will use
   UTF-32.  The encoding is set by the programming language and cannot
   be altered.

-  addr, *starting display address*
   addr is the address where you want GDB to begin displaying
   memory.  The expression need not have a pointer value (though it may);
   it is always interpreted as an integer address of a byte of memory.
   See [Expressions](Expressions.html#Expressions), for more information on expressions.  The default for
   addr is usually just after the last address examined&mdash;but several
   other commands also set the default address: `info breakpoints` (to
   the address of the last breakpoint listed), `info line` (to the
   starting address of a line), and `print` (if you use it to display
   a value from memory).

For example, `x/3uh 0x54320` is a request to display three <i>h</i>alfwords
(`h`) of memory, formatted as unsigned decimal integers (`u`),
starting at address `0x54320`.  `x/4xw $sp` prints the four
words (`w`) of memory above the stack pointer (here, `$sp`;
see [Registers](Registers.html#Registers)) in hexadecimal (`x`).

You can also specify a negative repeat count to examine memory backward
from the given address.  For example, `x/-3uh 0x54320` prints three
halfwords (`h`) at `0x54314`, `0x54328`, and `0x5431c`.

Since the letters indicating unit sizes are all distinct from the
letters specifying output formats, you do not have to remember whether
unit size or format comes first; either order works.  _The output
specifications `4xw` and `4wx` mean exactly the same thing_.
(However, the count n must come first; `wx4` does not work.)

Even though the unit size u is ignored for the formats `s`
and `i`, you might still want to use a count n; for example,
`3i` specifies that you want to see three machine instructions,
including any operands.  For convenience, especially when used with
the `display` command, the `i` format also prints branch delay
slot instructions, if any, beyond the count specified, which immediately
follow the last instruction that is within the count.  The command
`disassemble` gives an alternative way of inspecting machine
instructions; see [Source and Machine Code](Machine-Code.html#Machine-Code).

If a negative repeat count is specified for the formats `s` or `i`,
the command displays null-terminated strings or instructions before the given
address as many as the absolute value of the given number.  For the `i`
format, we use line number information in the debug info to accurately locate
instruction boundaries while disassembling backward.  If line info is not
available, the command stops examining memory with an error message.

All the defaults for the arguments to `x` are designed to make it
easy to continue scanning memory with minimal specifications each time
you use `x`.  For example, after you have inspected three machine
instructions with `x/3i addr`, you can inspect the next seven
with just `x/7`.  If you use RET to repeat the `x` command,
the repeat count n is used again; the other arguments default as
for successive uses of `x`.

When examining machine instructions, the instruction at current program
counter is shown with a `=>` marker. For example:

    (gdb) x/5i $pc-6
       0x804837f <main+11>: mov    %esp,%ebp
       0x8048381 <main+13>: push   %ecx
       0x8048382 <main+14>: sub    $0x4,%esp
    => 0x8048385 <main+17>: movl   $0x8048460,(%esp)
       0x804838c <main+24>: call   0x80482d4 <puts@plt>

The addresses and contents printed by the `x` command are not saved
in the value history because there is often too much of them and they
would get in the way.  Instead, GDB makes these values available for
subsequent use in expressions as values of the convenience variables
`$_` and `$__`.  After an `x` command, the last address
examined is available for use in expressions in the convenience variable
`$_`.  The contents of that address, as examined, are available in
the convenience variable `$__`.

If the `x` command has a repeat count, the address and contents saved
are from the last memory unit printed; this is not the same as the last
address printed if several units were printed on the last line of output.

Most targets have an addressable memory unit size of 8 bits.  This means
that to each memory address are associated 8 bits of data.  Some
targets, however, have other addressable memory unit sizes.
Within GDB and this document, the term
*addressable memory unit* (or *memory unit* for short) is used
when explicitly referring to a chunk of data of that size.  The word
*byte* is used to refer to a chunk of data of 8 bits, regardless of
the addressable memory unit size of the target.  For most systems,
addressable memory unit is a synonym of byte.

When you are debugging a program running on a remote target machine
(see [Remote Debugging](Remote-Debugging.html#Remote-Debugging)), you may wish to verify the program's image
in the remote machine's memory against the executable file you
downloaded to the target.  Or, on any target, you may want to check
whether the program has corrupted its own read-only sections.  The
`compare-sections` command is provided for such situations.

`compare-sections [section-name|`-r`]`
Compare the data of a loadable section section-name in the
executable file of the program being debugged with the same section in
the target machine's memory, and report any mismatches.  With no
arguments, compares all loadable sections.  With an argument of
`-r`, compares all loadable read-only sections.

Note: for remote targets, this command can be accelerated if the
target supports computing the CRC checksum of a block of memory
(see [qCRC packet](General-Query-Packets.html#qCRC-packet)).


## 10.7 Automatic Display

If you find that you want to print the value of an expression frequently
(to see how it changes), you might want to add it to the *automatic
display list* so that GDB prints its value each time your program stops.
Each expression added to the list is given a number to identify it;
to remove an expression from the list, you specify that number.
The automatic display looks like this:

    2: foo = 38
    3: bar[5] = (struct hack *) 0x3804

This display shows item numbers, expressions and their current values.  As with
displays you request manually using `x` or `print`, you can
specify the output format you prefer; in fact, `display` decides
whether to use `print` or `x` depending your format
specification&mdash;it uses `x` if you specify either the `i`
or `s` format, or a unit size; otherwise it uses `print`.

- `display expr`
   Add the expression expr to the list of expressions to display
   each time your program stops.  See [Expressions](Expressions.html#Expressions).

   `display` does not repeat if you press RET again after using it.

- `display/fmtexpr`
   For fmt specifying only a display format and not a size or
   count, add the expression expr to the auto-display list but
   arrange to display it each time in the specified format fmt.
   See [Output Formats](Output-Formats.html#Output-Formats).

- `display/fmtaddr`
   For fmt `i` or `s`, or including a unit-size or a
   number of units, add the expression addr as a memory address to
   be examined each time your program stops.  Examining means in effect
   doing `x/fmtaddr`.  See [Examining Memory](Memory.html#Memory).

   For example, `display/i $pc` can be helpful, to see the machine
   instruction about to be executed each time execution stops (`$pc`
   is a common name for the program counter; see [Registers](Registers.html#Registers)).

- `undisplay dnums&hellip;`
- `delete display dnums&hellip;`
   Remove items from the list of expressions to display.  Specify the
   numbers of the displays that you want affected with the command
   argument dnums.  It can be a single display number, one of the
   numbers shown in the first field of the `info display` display;
   or it could be a range of display numbers, as in `2-4`.

- `undisplay` does not repeat if you press RET after using it.
   (Otherwise you would just get the error `No display number &hellip;`.)

- `disable display dnums&hellip;`
   Disable the display of item numbers dnums.  A disabled display
   item is not printed automatically, but is not forgotten.  It may be
   enabled again later.  Specify the numbers of the displays that you
   want affected with the command argument dnums.  It can be a
   single display number, one of the numbers shown in the first field of
   the `info display` display; or it could be a range of display
   numbers, as in `2-4`.

- `enable display dnums&hellip;`
   Enable display of item numbers dnums.  It becomes effective once
   again in auto display of its expression, until you specify otherwise.
   Specify the numbers of the displays that you want affected with the
   command argument dnums.  It can be a single display number, one
   of the numbers shown in the first field of the `info display`
   display; or it could be a range of display numbers, as in `2-4`.

- `display`
   Display the current values of the expressions on the list, just as is
   done when your program stops.

- `info display`
   Print the list of expressions previously set up to display
   automatically, each one with its item number, but without showing the
   values.  This includes disabled expressions, which are marked as such.
   It also includes expressions which would not be displayed right now
   because they refer to automatic variables not currently available.

If a display expression refers to local variables, then it does not make
sense outside the lexical context for which it was set up.  Such an
expression is disabled when execution enters a context where one of its
variables is not defined.  For example, if you give the command
`display last_char` while inside a function with an argument
`last_char`, GDB displays this argument while your program
continues to stop inside that function.  When it stops elsewhere&mdash;where
there is no variable `last_char`&mdash;the display is disabled
automatically.  The next time your program stops where `last_char`
is meaningful, you can enable the display expression once again.


## 10.8 Print Settings

GDB provides the following ways to control how arrays, structures,
and symbols are printed.

These settings are useful for debugging programs in any language:

- `set print address` 
- `set print address on`
   GDB prints memory addresses showing the location of stack
   traces, structure values, pointer values, breakpoints, and so forth,
   even when it also displays the contents of those addresses.  The default
   is `on`.  For example, this is what a stack frame display looks like with
   `set print address on`:

   ```    
   (gdb) f
   #0  set_quotes (lq=0x34c78 "<<", rq=0x34c88 ">>")
   at input.c:530
   530         if (lquote != def_lquote)
   ```    

- `set print address off`
   Do not print addresses when displaying their contents.  For example,
   this is the same stack frame displayed with `set print address off`:

   ```    
   (gdb) set print addr off
   (gdb) f
   #0  set_quotes (lq="<<", rq=">>") at input.c:530
   530         if (lquote != def_lquote)
   ```    

   You can use `set print address off` to eliminate all machine
   dependent displays from the GDB interface.  For example, with
   `print address off`, you should get the same text for backtraces on
   all machines&mdash;whether or not they involve pointer arguments.

`show print address`
   Show whether or not addresses are to be printed.

   When GDB prints a symbolic address, it normally prints the
   closest earlier symbol plus an offset.  If that symbol does not uniquely
   identify the address (for example, it is a name whose scope is a single
   source file), you may need to clarify.  One way to do this is with
   `info line`, for example `info line *0x4537`.  Alternately,
   you can set GDB to print the source file and line number when
   it prints a symbolic address:

`set print symbol-filename on`
   Tell GDB to print the source file name and line number of a
   symbol in the symbolic form of an address.

`set print symbol-filename off`
   Do not print source file name and line number of a symbol.  This is the
   default.

`show print symbol-filename`
   Show whether or not GDB will print the source file name and
   line number of a symbol in the symbolic form of an address.

   Another situation where it is helpful to show symbol filenames and line
   numbers is when disassembling code; GDB shows you the line
   number and source file that corresponds to each instruction.

   Also, you may wish to see the symbolic form only if the address being
   printed is reasonably close to the closest earlier symbol:

- `set print max-symbolic-offset max-offset` 
- `set print max-symbolic-offset unlimited`
   Tell GDB to only display the symbolic form of an address if the
   offset between the closest earlier symbol and the address is less than
   max-offset.  The default is `unlimited`, which tells GDB
   to always print the symbolic form of an address if any symbol precedes
   it.  Zero is equivalent to `unlimited`.

- `show print max-symbolic-offset`
   Ask how large the maximum offset is that GDB prints in a
   symbolic address.

   If you have a pointer and you are not sure where it points, try
   `set print symbol-filename on`.  Then you can determine the name
   and source file location of the variable where it points, using
   `p/a pointer`.  This interprets the address in symbolic form.
   For example, here GDB shows that a variable `ptt` points
   at another variable `t`, defined in hi2.c:

   ```    
   (gdb) set print symbol-filename on
    (gdb) p/a ptt
   $4 = 0xe008 <t in hi2.c>
   ```    

   > *Warning:* For pointers that point to a local variable, `p/a`
   > does not show the symbol name and filename of the referent, even with
   > the appropriate `set print` options turned on.

   You can also enable `/a`-like formatting all the time using
   `set print symbol on`:

- `set print symbol on`
   Tell GDB to print the symbol corresponding to an address, if
   one exists.

- `set print symbol off`
   Tell GDB not to print the symbol corresponding to an
   address.  In this mode, GDB will still print the symbol
   corresponding to pointers to functions.  This is the default.

- `show print symbol`
   Show whether GDB will display the symbol corresponding to an
   address.

   Other settings control how different kinds of objects are printed:

- `set print array`
- `set print array on`
   Pretty print arrays.  This format is more convenient to read,
   but uses more space.  The default is off.

- `set print array off`
   Return to compressed format for arrays.

- `show print array`
   Show whether compressed or pretty format is selected for displaying
   arrays.

- `set print array-indexes`
- `set print array-indexes on`
   Print the index of each element when displaying arrays.  May be more
   convenient to locate a given element in the array or quickly find the
   index of a given element in that printed array.  The default is off.

- `set print array-indexes off`
   Stop printing element indexes when displaying arrays.

- `show print array-indexes`
   Show whether the index of each element is printed when displaying
   arrays.

- `set print elements number-of-elements`
- `set print elements unlimited`
   Set a limit on how many elements of an array GDB will print.
   If GDB is printing a large array, it stops printing after it has
   printed the number of elements set by the `set print elements` command.
   This limit also applies to the display of strings.
   When GDB starts, this limit is set to 200.
   Setting number-of-elements to `unlimited` or zero means
   that the number of elements to print is unlimited.

- `show print elements`
   Display the number of elements of a large array that GDB will print.
   If the number is 0, then the printing is unlimited.

- `set print frame-arguments value`
   This command allows to control how the values of arguments are printed
   when the debugger prints a frame (see [Frames](Frames.html#Frames)).  The possible
   values are:

- `all`
   The values of all arguments are printed.

- `scalars`
   Print the value of an argument only if it is a scalar.  The value of more
   complex arguments such as arrays, structures, unions, etc, is replaced
   by `&hellip;`.  This is the default.  Here is an example where
   only scalar arguments are shown:

   ```
   #1  0x08048361 in call_me (i=3, s=&hellip;, ss=0xbf8d508c, u=&hellip;, e=green)
   at frame-args.c:23
   ```

- `none`
   None of the argument values are printed.  Instead, the value of each argument
   is replaced by `&hellip;`.  In this case, the example above now becomes:

   ```
   #1  0x08048361 in call_me (i=&hellip;, s=&hellip;, ss=&hellip;, u=&hellip;, e=&hellip;)
   at frame-args.c:23
   ```

   By default, only scalar arguments are printed.  This command can be used
   to configure the debugger to print the value of all arguments, regardless
   of their type.  However, it is often advantageous to not print the value
   of more complex parameters.  For instance, it reduces the amount of
   information printed in each frame, making the backtrace more readable.
   Also, it improves performance when displaying Ada frames, because
   the computation of large arguments can sometimes be CPU-intensive,
   especially in large applications.  Setting `print frame-arguments`
   `scalars` (the default) or `none` avoids this computation,
   thus speeding up the display of each Ada frame.
   
- `show print frame-arguments`
   Show how the value of arguments should be displayed when printing a frame.

- `set print raw frame-arguments on`
   Print frame arguments in raw, non pretty-printed, form.

- `set print raw frame-arguments off`
   Print frame arguments in pretty-printed form, if there is a pretty-printer
   for the value (see [Pretty Printing](Pretty-Printing.html#Pretty-Printing)),
   otherwise print the value in raw form.
   This is the default.

- `show print raw frame-arguments`
   Show whether to print frame arguments in raw form.

`set print entry-values value`
   Set printing of frame argument values at function entry.  In some cases
   GDB can determine the value of function argument which was passed by
   the function caller, even if the value was modified inside the called function
   and therefore is different.  With optimized code, the current value could be
   unavailable, but the entry value may still be known.

   The default value is `default` (see below for its description).  Older
   GDB behaved as with the setting `no`.  Compilers not supporting
   this feature will behave in the `default` setting the same way as with the
   `no` setting.

   This functionality is currently supported only by DWARF 2 debugging format and
   the compiler has to produce `DW_TAG_call_site` tags.  With
   GCC, you need to specify -O -g during compilation, to get
   this information.

   The value parameter can be one of the following:

- `no`
   Print only actual parameter values, never print values from function entry
   point.

   ```
   #0  equal (val=5)
   #0  different (val=6)
   #0  lost (val=<optimized out>)
   #0  born (val=10)
   #0  invalid (val=<optimized out>)
   ```

- `only`
   Print only parameter values from function entry point.  The actual parameter
   values are never printed.

   ```
   #0  equal (val@entry=5)
   #0  different (val@entry=5)
   #0  lost (val@entry=5)
   #0  born (val@entry=<optimized out>)
   #0  invalid (val@entry=<optimized out>)
   ```

- `preferred`
   Print only parameter values from function entry point.  If value from function
   entry point is not known while the actual value is known, print the actual
   value for such parameter.

   ```
   #0  equal (val@entry=5)
   #0  different (val@entry=5)
   #0  lost (val@entry=5)
   #0  born (val=10)
   #0  invalid (val@entry=<optimized out>)
   ```

- `if-needed`
   Print actual parameter values.  If actual parameter value is not known while
   value from function entry point is known, print the entry point value for such
   parameter.

   ```
   #0  equal (val=5)
   #0  different (val=6)
   #0  lost (val@entry=5)
   #0  born (val=10)
   #0  invalid (val=<optimized out>)
   ```

- `both`
   Always print both the actual parameter value and its value from function entry
   point, even if values of one or both are not available due to compiler
   optimizations.

   ```
   #0  equal (val=5, val@entry=5)
   #0  different (val=6, val@entry=5)
   #0  lost (val=<optimized out>, val@entry=5)
   #0  born (val=10, val@entry=<optimized out>)
   #0  invalid (val=<optimized out>, val@entry=<optimized out>)
   ```

- `compact`
   Print the actual parameter value if it is known and also its value from
   function entry point if it is known.  If neither is known, print for the actual
   value `<optimized out>`.  If not in MI mode (see [GDB/MI](GDB_002fMI.html#GDB_002fMI)) and if both
   values are known and identical, print the shortened
   `param=param@entry=VALUE` notation.

   ```
   #0  equal (val=val@entry=5)
   #0  different (val=6, val@entry=5)
   #0  lost (val@entry=5)
   #0  born (val=10)
   #0  invalid (val=<optimized out>)
   ```

- `default`
   Always print the actual parameter value.  Print also its value from function
   entry point, but only if it is known.  If not in MI mode (see [GDB/MI](GDB_002fMI.html#GDB_002fMI)) and
   if both values are known and identical, print the shortened
   `param=param@entry=VALUE` notation.
   
   ```
   #0  equal (val=val@entry=5)
   #0  different (val=6, val@entry=5)
   #0  lost (val=<optimized out>, val@entry=5)
   #0  born (val=10)
   #0  invalid (val=<optimized out>)
   ```

   For analysis messages on possible failures of frame argument values at function
   entry resolution see [set debug entry-values](Tail-Call-Frames.html#set-debug-entry_002dvalues).

- `show print entry-values`
   Show the method being used for printing of frame argument values at function
   entry.

- `set print repeats number-of-repeats``set print repeats unlimited`
   Set the threshold for suppressing display of repeated array
   elements.  When the number of consecutive identical elements of an
   array exceeds the threshold, GDB prints the string
   `"<repeats n times>"`, where n is the number of
   identical repetitions, instead of displaying the identical elements
   themselves.  Setting the threshold to `unlimited` or zero will
   cause all elements to be individually printed.  The default threshold
   is 10.

- `show print repeats`
   Display the current threshold for printing repeated identical
   elements.

- `set print null-stop`
   Cause GDB to stop printing the characters of an array when the first
   NULL is encountered.  This is useful when large arrays actually
   contain only short strings.
   The default is off.

- `show print null-stop`
   Show whether GDB stops printing an array on the first
   NULL character.

- `set print pretty on`
   Cause GDB to print structures in an indented format with one member
   per line, like this:

   ```
   $1 = {
     next = 0x0,
     flags = {
       sweet = 1,
       sour = 1
     },
   meat = 0x54 "Pork"
   }
   ```

- `set print pretty off`
   Cause GDB to print structures in a compact format, like this:

   ```
   $1 = {next = 0x0, flags = {sweet = 1, sour = 1}, \
   meat = 0x54 "Pork"}
   ```

   This is the default format.

- `show print pretty`
   Show which format GDB is using to print structures.

- `set print sevenbit-strings on`
   Print using only seven-bit characters; if this option is set,
   GDB displays any eight-bit characters (in strings or
   character values) using the notation `\nnn`.  This setting is
   best if you are working in English (ASCII) and you use the
   high-order bit of characters as a marker or `meta` bit.

- `set print sevenbit-strings off`
   Print full eight-bit characters.  This allows the use of more
   international character sets, and is the default.

- `show print sevenbit-strings`
   Show whether or not GDB is printing only seven-bit characters.

- `set print union on`
   Tell GDB to print unions which are contained in structures
   and other unions.  This is the default setting.

- `set print union off`
   Tell GDB not to print unions which are contained in
   structures and other unions.  GDB will print `"{...}"`
   instead.

- `show print union`
   Ask GDB whether or not it will print unions which are contained in
   structures and other unions.

   For example, given the declarations

   ```c
   typedef enum {Tree, Bug} Species;
   typedef enum {Big_tree, Acorn, Seedling} Tree_forms;
   typedef enum {Caterpillar, Cocoon, Butterfly} Bug_forms;
    
   struct thing {
     Species it;
     union {
       Tree_forms tree;
       Bug_forms bug;
     } form;
   };
    
   struct thing foo = {Tree, {Acorn}};
   ```

   with `set print union on` in effect `p foo` would print

   ```
   $1 = {it = Tree, form = {tree = Acorn, bug = Cocoon}}
   ```

   and with `set print union off` in effect it would print

   ```
   $1 = {it = Tree, form = {...}}
   ```

   `set print union` affects programs written in C-like languages
   and in Pascal.

   These settings are of interest when debugging C++ programs:

- `set print demangle`
- `set print demangle on`
   Print C++ names in their source form rather than in the encoded
   (`mangled`) form passed to the assembler and linker for type-safe
   linkage.  The default is on.

- `show print demangle`
   Show whether C++ names are printed in mangled or demangled form.

- `set print asm-demangle`
  `set print asm-demangle on`
   Print C++ names in their source form rather than their mangled form, even
   in assembler code printouts such as instruction disassemblies.
   The default is off.

- `show print asm-demangle`
   Show whether C++ names in assembly listings are printed in mangled
   or demangled form.

- `set demangle-style style`
   Choose among several encoding schemes used by different compilers to
   represent C++ names.  The choices for style are currently:

- `auto`
   Allow GDB to choose a decoding style by inspecting your program.
   This is the default.

- `gnu`
   Decode based on the GNU C++ compiler (`g++`) encoding algorithm.

- `hp`
   Decode based on the HP ANSI C++ (`aCC`) encoding algorithm.

- `lucid`
   Decode based on the Lucid C++ compiler (`lcc`) encoding algorithm.

- `arm`
   Decode using the algorithm in the C++ Annotated Reference Manual.
   **Warning:** this setting alone is not sufficient to allow
   debugging `cfront`-generated executables.  GDB would
   require further enhancement to permit that.

   If you omit style, you will see a list of possible formats.

- `show demangle-style`
   Display the encoding style currently in use for decoding C++ symbols.

- `set print object``set print object on`
   When displaying a pointer to an object, identify the *actual*
   (derived) type of the object rather than the *declared* type, using
   the virtual function table.  Note that the virtual function table is
   required&mdash;this feature can only work for objects that have run-time
   type identification; a single virtual method in the object's declared
   type is sufficient.  Note that this setting is also taken into account when
   working with variable objects via MI (see [GDB/MI](GDB_002fMI.html#GDB_002fMI)).

- `set print object off`
   Display only the declared type of objects, without reference to the
   virtual function table.  This is the default setting.

- `show print object`
   Show whether actual, or declared, object types are displayed.

- `set print static-members``set print static-members on`
   Print static members when displaying a C++ object.  The default is on.

- `set print static-members off`
   Do not print static members when displaying a C++ object.

- `show print static-members`
   Show whether C++ static members are printed or not.

- `set print pascal_static-members``set print pascal_static-members on`
   Print static members when displaying a Pascal object.  The default is on.

- `set print pascal_static-members off`
   Do not print static members when displaying a Pascal object.

- `show print pascal_static-members`
   Show whether Pascal static members are printed or not.

- `set print vtbl``set print vtbl on`
   Pretty print C++ virtual function tables.  The default is off.
   (The `vtbl` commands do not work on programs compiled with the HP
   ANSI C++ compiler (`aCC`).)

- `set print vtbl off`
   Do not pretty print C++ virtual function tables.

- `show print vtbl`
   Show whether C++ virtual function tables are pretty printed, or not.


## 10.9 Pretty Printing

GDB provides a mechanism to allow pretty-printing of values using
Python code.  It greatly simplifies the display of complex objects.  This
mechanism works for both MI and the CLI.


### 10.9.1 Pretty-Printer Introduction

When GDB prints a value, it first sees if there is a pretty-printer
registered for the value.  If there is then GDB invokes the
pretty-printer to print the value.  Otherwise the value is printed normally.

Pretty-printers are normally named.  This makes them easy to manage.
The &lsquo;info pretty-printer&rsquo; command will list all the installed
pretty-printers with their names.
If a pretty-printer can handle multiple data types, then its
*subprinters* are the printers for the individual data types.
Each such subprinter has its own name.
The format of the name is printer-name;subprinter-name.

Pretty-printers are installed by *registering* them with GDB.
Typically they are automatically loaded and registered when the corresponding
debug information is loaded, thus making them available without having to
do anything special.

There are three places where a pretty-printer can be registered.

-  Pretty-printers registered globally are available when debugging
   all inferiors.

-  Pretty-printers registered with a program space are available only
   when debugging that program.
   See [Progspaces In Python](Progspaces-In-Python.html#Progspaces-In-Python), for more details on program spaces in Python.

-  Pretty-printers registered with an objfile are loaded and unloaded
   with the corresponding objfile (e.g., shared library).
   See [Objfiles In Python](Objfiles-In-Python.html#Objfiles-In-Python), for more details on objfiles in Python.

See [Selecting Pretty-Printers](Selecting-Pretty_002dPrinters.html#Selecting-Pretty_002dPrinters), for further information on how 
pretty-printers are selected,

See [Writing a Pretty-Printer](Writing-a-Pretty_002dPrinter.html#Writing-a-Pretty_002dPrinter), for implementing pretty printers
for new types.


### 10.9.2 Pretty-Printer Example

Here is how a C++`std::string` looks without a pretty-printer:

    (gdb) print s
    $1 = {
      static npos = 4294967295, 
      _M_dataplus = {
        <std::allocator<char>> = {
          <__gnu_cxx::new_allocator<char>> = {
            <No data fields>}, <No data fields>
          },
        members of std::basic_string<char, std::char_traits<char>,
          std::allocator<char> >::_Alloc_hider:
        _M_p = 0x804a014 "abcd"
      }
    }

With a pretty-printer for `std::string` only the contents are printed:

    (gdb) print s
    $2 = "abcd"


### 10.9.3 Pretty-Printer Commands

- `info pretty-printer [object-regexp [name-regexp]]`
   Print the list of installed pretty-printers.
   This includes disabled pretty-printers, which are marked as such.

   object-regexp is a regular expression matching the objects
   whose pretty-printers to list.
   Objects can be `global`, the program space&rsquo;s file
   (see [Progspaces In Python](Progspaces-In-Python.html#Progspaces-In-Python)),
   and the object files within that program space (see [Objfiles In Python](Objfiles-In-Python.html#Objfiles-In-Python)).
   See [Selecting Pretty-Printers](Selecting-Pretty_002dPrinters.html#Selecting-Pretty_002dPrinters), for details on how GDB
   looks up a printer from these three objects.

   name-regexp is a regular expression matching the name of the printers
   to list.

- `disable pretty-printer [object-regexp [name-regexp]]`
   Disable pretty-printers matching object-regexp and name-regexp.
   A disabled pretty-printer is not forgotten, it may be enabled again later.

- `enable pretty-printer [object-regexp [name-regexp]]`
   Enable pretty-printers matching object-regexp and name-regexp.

**Example:**

Suppose we have three pretty-printers installed: one from library1.so
named `foo` that prints objects of type `foo`, and
another from library2.so named `bar` that prints two types of objects,
`bar1` and `bar2`.

    (gdb) info pretty-printer
    library1.so:
      foo
    library2.so:
      bar
        bar1
        bar2
    (gdb) info pretty-printer library2
    library2.so:
      bar
        bar1
        bar2
    (gdb) disable pretty-printer library1
    1 printer disabled
    2 of 3 printers enabled
    (gdb) info pretty-printer
    library1.so:
      foo [disabled]
    library2.so:
      bar
        bar1
        bar2
    (gdb) disable pretty-printer library2 bar:bar1
    1 printer disabled
    1 of 3 printers enabled
    (gdb) info pretty-printer library2
    library1.so:
      foo [disabled]
    library2.so:
      bar
        bar1 [disabled]
        bar2
    (gdb) disable pretty-printer library2 bar
    1 printer disabled
    0 of 3 printers enabled
    (gdb) info pretty-printer library2
    library1.so:
      foo [disabled]
    library2.so:
      bar [disabled]
        bar1 [disabled]
        bar2

Note that for `bar` the entire printer can be disabled,
as can each individual subprinter.


## 10.10 Value History

Values printed by the `print` command are saved in the GDB *value history*.
This allows you to refer to them in other expressions.
Values are kept until the symbol table is re-read or discarded
(for example with the `file` or `symbol-file` commands).
When the symbol table changes, the value history is discarded,
since the values may contain pointers back to the types defined in the
symbol table.

The values printed are given *history numbers* by which you can
refer to them.  These are successive integers starting with one.
`print` shows you the history number assigned to a value by
printing `$num = ` before the value; here num is the
history number.

To refer to any previous value, use `$` followed by the value's
history number.  The way `print` labels its output is designed to
remind you of this.  Just `$` refers to the most recent value in
the history, and `$$` refers to the value before that.
`$$n` refers to the nth value from the end; `$$2`
is the value just prior to `$$`, `$$1` is equivalent to
`$$`, and `$$0` is equivalent to `$`.

For example, suppose you have just printed a pointer to a structure and
want to see the contents of the structure.  It suffices to type

    p *$

If you have a chain of structures where the component `next` points
to the next one, you can print the contents of the next one with this:

    p *$.next

You can print successive links in the chain by repeating this
command&mdash;which you can do by just typing RET.

Note that the history records values, not expressions.  If the value of
`x` is 4 and you type these commands:

    print x
    set x=5

then the value recorded in the value history by the `print` command
remains 4 even though the value of `x` has changed.

- `show values`
   Print the last ten values in the value history, with their item numbers.
   This is like `p $$9` repeated ten times, except that `show values` 
   does not change the history.

- `show values n`
   Print ten history values centered on history item number n.

- `show values +`
   Print ten history values just after the values last printed.  If no more
   values are available, `show values +` produces no display.

Pressing RET to repeat `show values n` has exactly the
same effect as `show values +`.


## 10.11 Convenience Variables

GDB provides *convenience variables* that you can use within
GDB to hold on to a value and refer to it later.  These variables
exist entirely within GDB; they are not part of your program, and
setting a convenience variable has no direct effect on further execution
of your program.  That is why you can use them freely.

Convenience variables are prefixed with `$`.  Any name preceded by
`$` can be used for a convenience variable, unless it is one of
the predefined machine-specific register names (see [Registers](Registers.html#Registers)).
(Value history references, in contrast, are *numbers* preceded
by `$`.  See [Value History](Value-History.html#Value-History).)

You can save a value in a convenience variable with an assignment
expression, just as you would set a variable in your program.
For example:

    set $foo = *object_ptr

would save in `$foo` the value contained in the object pointed to by
`object_ptr`.

Using a convenience variable for the first time creates it, but its
value is `void` until you assign a new value.  You can alter the
value with another assignment at any time.

Convenience variables have no fixed types.  You can assign a convenience
variable any type of value, including structures and arrays, even if
that variable already has a value of a different type.  The convenience
variable, when used as an expression, has the type of its current value.

- `show convenience`
   Print a list of convenience variables used so far, and their values,
   as well as a list of the convenience functions.
   Abbreviated `show conv`.

`init-if-undefined $variable = expression`
   Set a convenience variable if it has not already been set.  This is useful
   for user-defined commands that keep some state.  It is similar, in concept,
   to using local static variables with initializers in C (except that
   convenience variables are global).  It can also be used to allow users to
   override default values used in a command script.

   If the variable is already defined then the expression is not evaluated so
   any side-effects do not occur.

   One of the ways to use a convenience variable is as a counter to be
   incremented or a pointer to be advanced.  For example, to print
   a field from successive elements of an array of structures:

   ```
   set $i = 0
   print bar[$i++]->contents
   ```

   Repeat that command by typing RET.

   Some convenience variables are created automatically by GDB and given
   values likely to be useful.

- `$_`
   The variable `$_` is automatically set by the `x` command to
   the last address examined (see [Examining Memory](Memory.html#Memory)).  Other
   commands which provide a default address for `x` to examine also
   set `$_` to that address; these commands include `info line`
   and `info breakpoint`.  The type of `$_` is `void *`
   except when set by the `x` command, in which case it is a pointer
   to the type of `$__`.

- `$__`
   The variable `$__` is automatically set by the `x` command
   to the value found in the last address examined.  Its type is chosen
   to match the format in which the data was printed.

- `$_exitcode`
   When the program being debugged terminates normally, GDB
   automatically sets this variable to the exit code of the program, and
   resets `$_exitsignal` to `void`.

- `$_exitsignal`
   When the program being debugged dies due to an uncaught signal,
   GDB automatically sets this variable to that signal's number,
   and resets `$_exitcode` to `void`.

   To distinguish between whether the program being debugged has exited
   (i.e., `$_exitcode` is not `void`) or signalled (i.e.,
   `$_exitsignal` is not `void`), the convenience function
   `$_isvoid` can be used (see [Convenience Functions](Convenience-Funs.html#Convenience-Funs)).
   For example, considering the following source code:

   ```c
   #include <signal.h>
    
   int
   main (int argc, char *argv[])
   {
     raise (SIGALRM);
     return 0;
   }
   ```

   A valid way of telling whether the program being debugged has exited
   or signalled would be:

   ```
   (gdb) define has_exited_or_signalled
   Type commands for definition of ``has_exited_or_signalled''.
   End with a line saying just ``end''.
   >if $_isvoid ($_exitsignal)
   >echo The program has exited\n
   >else
   >echo The program has signalled\n
   >end
   >end
   (gdb) run
   Starting program:
    
   Program terminated with signal SIGALRM, Alarm clock.
   The program no longer exists.
   (gdb) has_exited_or_signalled
   The program has signalled
   ```

   As can be seen, GDB correctly informs that the program being
   debugged has signalled, since it calls `raise` and raises a
   `SIGALRM` signal.  If the program being debugged had not called
   `raise`, then GDB would report a normal exit:

   ```
   (gdb) has_exited_or_signalled
   The program has exited
   ```

- `$_exception`
   The variable `$_exception` is set to the exception object being
   thrown at an exception-related catchpoint.  See [Set Catchpoints](Set-Catchpoints.html#Set-Catchpoints).

- `$_probe_argc``$_probe_arg0&hellip;$_probe_arg11`
   Arguments to a static probe.  See [Static Probe Points](Static-Probe-Points.html#Static-Probe-Points).

- `$_sdata`
   The variable `$_sdata` contains extra collected static tracepoint
   data.  See [Tracepoint Action Lists](Tracepoint-Actions.html#Tracepoint-Actions).  Note that
   `$_sdata` could be empty, if not inspecting a trace buffer, or
   if extra static tracepoint data has not been collected.

- `$_siginfo`
   The variable `$_siginfo` contains extra signal information
   (see [extra signal information](Signals.html#extra-signal-information)).  Note that `$_siginfo`
   could be empty, if the application has not yet received any signals.
   For example, it will be empty before you execute the `run` command.

- `$_tlb`
   The variable `$_tlb` is automatically set when debugging
   applications running on MS-Windows in native mode or connected to
   gdbserver that supports the `qGetTIBAddr` request. 
   See [General Query Packets](General-Query-Packets.html#General-Query-Packets).
   This variable contains the address of the thread information block.

- `$_inferior`
   The number of the current inferior.  See [Debugging Multiple Inferiors and Programs](Inferiors-and-Programs.html#Inferiors-and-Programs).

- `$_thread`
   The thread number of the current thread.  See [thread numbers](Threads.html#thread-numbers).

- `$_gthread`
   The global number of the current thread.  See [global thread numbers](Threads.html#global-thread-numbers).


## 10.12 Convenience Functions

GDB also supplies some *convenience functions*.  These
have a syntax similar to convenience variables.  A convenience
function can be used in an expression just like an ordinary function;
however, a convenience function is implemented internally to
GDB.

These functions do not require GDB to be configured with
`Python` support, which means that they are always available.

- `$_isvoid (expr)`
   Return one if the expression expr is `void`.  Otherwise it
   returns zero.

   A `void` expression is an expression where the type of the result
   is `void`.  For example, you can examine a convenience variable
   (see [Convenience Variables](Convenience-Vars.html#Convenience-Vars)) to check whether
   it is `void`:

   ```
   (gdb) print $_exitcode
   $1 = void
   (gdb) print $_isvoid ($_exitcode)
   $2 = 1
   (gdb) run
   Starting program: ./a.out
    [Inferior 1 (process 29572) exited normally]
   (gdb) print $_exitcode
   $3 = 0
   (gdb) print $_isvoid ($_exitcode)
   $4 = 0
   ```

   In the example above, we used `$_isvoid` to check whether
   `$_exitcode` is `void` before and after the execution of the
   program being debugged.  Before the execution there is no exit code to
   be examined, therefore `$_exitcode` is `void`.  After the
   execution the program being debugged returned zero, therefore
   `$_exitcode` is zero, which means that it is not `void`
   anymore.

   The `void` expression can also be a call of a function from the
   program being debugged.  For example, given the following function:

   ```c
   void
   foo (void)
   {
   }
   ```

   The result of calling it inside GDB is `void`:

   ```
   (gdb) print foo ()
   $1 = void
   (gdb) print $_isvoid (foo ())
   $2 = 1
    (gdb) set $v = foo ()
   (gdb) print $v
   $3 = void
    (gdb) print $_isvoid ($v)
   $4 = 1
   ```

   These functions require GDB to be configured with `Python` support.

- `$_memeq(buf1, buf2, length)`
   Returns one if the length bytes at the addresses given by
   buf1 and buf2 are equal.
   Otherwise it returns zero.

- `$_regex(str, regex)`
   Returns one if the string str matches the regular expression
   regex.  Otherwise it returns zero.
   The syntax of the regular expression is that specified by `Python`&rsquo;s
   regular expression support.

- `$_streq(str1, str2)`
   Returns one if the strings str1 and str2 are equal.
   Otherwise it returns zero.

- `$_strlen(str)`
   Returns the length of string str.

- `$_caller_is(name[, number_of_frames])`
   Returns one if the calling function&rsquo;s name is equal to name.
   Otherwise it returns zero.

   If the optional argument `number_of_frames` is provided,
   it is the number of frames up in the stack to look.
   The default is 1.

   **Example:**

   ```
   (gdb) backtrace
   #0  bottom_func ()
       at testsuite/gdb.python/py-caller-is.c:21
   #1  0x00000000004005a0 in middle_func ()
       at testsuite/gdb.python/py-caller-is.c:27
   #2  0x00000000004005ab in top_func ()
       at testsuite/gdb.python/py-caller-is.c:33
   #3  0x00000000004005b6 in main ()
       at testsuite/gdb.python/py-caller-is.c:39
   (gdb) print $_caller_is ("middle_func")
   $1 = 1
   (gdb) print $_caller_is ("top_func", 2)
   $1 = 1
   ```

- `$_caller_matches(regexp[, number_of_frames])`
   Returns one if the calling function's name matches the regular expression
   regexp.  Otherwise it returns zero.

   If the optional argument `number_of_frames` is provided,
   it is the number of frames up in the stack to look.
   The default is 1.

- `$_any_caller_is(name[, number_of_frames])`
   Returns one if any calling function's name is equal to name.
   Otherwise it returns zero.

   If the optional argument `number_of_frames` is provided,
   it is the number of frames up in the stack to look.
   The default is 1.

   This function differs from `$_caller_is` in that this function
   checks all stack frames from the immediate caller to the frame specified
   by number_of_frames, whereas `$_caller_is` only checks the
   frame specified by `number_of_frames`.

- `$_any_caller_matches(regexp[, number_of_frames])`
   Returns one if any calling function's name matches the regular expression
   regexp.  Otherwise it returns zero.

   If the optional argument `number_of_frames` is provided,
   it is the number of frames up in the stack to look.
   The default is 1.

   This function differs from `$_caller_matches` in that this function
   checks all stack frames from the immediate caller to the frame specified
   by number_of_frames, whereas `$_caller_matches` only checks the
   frame specified by `number_of_frames`.

- `$_as_string(value)`
   Return the string representation of value.

   This function is useful to obtain the textual label (enumerator) of an
   enumeration value.  For example, assuming the variable node is of
   an enumerated type:

   ```
   (gdb) printf "Visiting node of type %s\n", $_as_string(node)
   Visiting node of type NODE_INTEGER
   ```

   GDB provides the ability to list and get help on convenience functions.

- `help function`
   Print a list of all convenience functions.


## 10.13 Registers

You can refer to machine register contents, in expressions, as variables
with names starting with `$`.  The names of registers are different
for each machine; use `info registers` to see the names used on
your machine.

- `info registers`
   Print the names and values of all registers except floating-point
   and vector registers (in the selected stack frame).

`info all-registers`
   Print the names and values of all registers, including floating-point
   and vector registers (in the selected stack frame).

`info registers reggroup &hellip;`
   Print the name and value of the registers in each of the specified
   reggroups.  The reggoup can be any of those returned by
   `maint print reggroups` (see [Maintenance Commands](Maintenance-Commands.html#Maintenance-Commands)).

`info registers regname &hellip;`
   Print the *relativized* value of each specified register regname.
   As discussed in detail below, register values are normally relative to
   the selected stack frame.  The regname may be any register name valid on
   the machine you are using, with or without the initial `$`.

GDB has four `standard` register names that are available (in
expressions) on most machines&mdash;whenever they do not conflict with an
architecture's canonical mnemonics for registers.  The register names
`$pc` and `$sp` are used for the program counter register and
the stack pointer.  `$fp` is used for a register that contains a
pointer to the current stack frame, and `$ps` is used for a
register that contains the processor status.  For example,
you could print the program counter in hex with

    p/x $pc

or print the instruction to be executed next with

    x/i $pc

or add four to the stack pointer[11](#FOOT11) with

    set $sp += 4

Whenever possible, these four standard register names are available on
your machine even though the machine has different canonical mnemonics,
so long as there is no conflict.  The `info registers` command
shows the canonical names.  For example, on the SPARC, `info
registers` displays the processor status register as `$psr` but you
can also refer to it as `$ps`; and on x86-based machines `$ps`
is an alias for the EFLAGS register.

GDB always considers the contents of an ordinary register as an
integer when the register is examined in this way.  Some machines have
special registers which can hold nothing but floating point; these
registers are considered to have floating point values.  There is no way
to refer to the contents of an ordinary register as floating point value
(although you can *print* it as a floating point value with
`print/f $regname`).

Some registers have distinct `raw` and `virtual` data formats.  This
means that the data format in which the register contents are saved by
the operating system is not the same one that your program normally
sees.  For example, the registers of the 68881 floating point
coprocessor are always saved in `extended` (raw) format, but all C
programs expect to work with `double` (virtual) format.  In such
cases, GDB normally works with the virtual format only (the format
that makes sense for your program), but the `info registers` command
prints the data in both formats.

Some machines have special registers whose contents can be interpreted
in several different ways.  For example, modern x86-based machines
have SSE and MMX registers that can hold several values packed
together in several different formats.  GDB refers to such
registers in `struct` notation:

    (gdb) print $xmm1
    $1 = {
      v4_float = {0, 3.43859137e-038, 1.54142831e-044, 1.821688e-044},
      v2_double = {9.92129282474342e-303, 2.7585945287983262e-313},
      v16_int8 = "\000\000\000\000\3706;\001\v\000\000\000\r\000\000",
      v8_int16 = {0, 0, 14072, 315, 11, 0, 13, 0},
      v4_int32 = {0, 20657912, 11, 13},
      v2_int64 = {88725056443645952, 55834574859},
      uint128 = 0x0000000d0000000b013b36f800000000
    }

To set values of such registers, you need to tell GDB which
view of the register you wish to change, as if you were assigning
value to a `struct` member:

    (gdb) set $xmm1.uint128 = 0x000000000000000000000000FFFFFFFF

Normally, register values are relative to the selected stack frame
(see [Selecting a Frame](Selection.html#Selection)).  This means that you get the
value that the register would contain if all stack frames farther in
were exited and their saved registers restored.  In order to see the
true contents of hardware registers, you must select the innermost
frame (with `frame 0`).

Usually ABIs reserve some registers as not needed to be saved by the
callee (a.k.a.: `caller-saved`, `call-clobbered` or `volatile`
registers).  It may therefore not be possible for GDB to know
the value a register had before the call (in other words, in the outer
frame), if the register value has since been changed by the callee.
GDB tries to deduce where the inner frame saved
(`callee-saved`) registers, from the debug info, unwind info, or the
machine code generated by your compiler.  If some register is not
saved, and GDB knows the register is `caller-saved` (via
its own knowledge of the ABI, or because the debug/unwind info
explicitly says the register's value is undefined), GDB
displays `<not saved>` as the register's value.  With targets
that GDB has no knowledge of the register saving convention,
if a register was not saved by the callee, then its value and location
in the outer frame are assumed to be the same of the inner frame.
This is usually harmless, because if the register is call-clobbered,
the caller either does not care what is in the register after the
call, or has code to restore the value that it does care about.  Note,
however, that if you change such a register in the outer frame, you
may also be affecting the inner frame.  Also, the more `outer` the
frame is you're looking at, the more likely a call-clobbered
register's value is to be wrong, in the sense that it doesn't actually
represent the value the register had just before the call.

---

#### Footnotes

### [(11)](#DOCF11)

This is a way of removing
one word from the stack, on machines where stacks grow downward in
memory (most machines, nowadays).  This assumes that the innermost
stack frame is selected; setting `$sp` is not allowed when other
stack frames are selected.  To pop entire frames off the stack,
regardless of machine architecture, use `return`;
see [Returning from a Function](Returning.html#Returning).


## 10.14 Floating Point Hardware

Depending on the configuration, GDB may be able to give
you more information about the status of the floating point hardware.

- `info float`
   Display hardware-dependent information about the floating
   point unit.  The exact contents and layout vary depending on the
   floating point chip.  Currently, `info float` is supported on
   the ARM and x86 machines.


## 10.15 Vector Unit

Depending on the configuration, GDB may be able to give you
more information about the status of the vector unit.

- `info vector`
   Display information about the vector unit.  The exact contents and
   layout vary depending on the hardware.


## 10.16 Operating System Auxiliary Information

GDB provides interfaces to useful OS facilities that can help
you debug your program.

Some operating systems supply an *auxiliary vector* to programs at
startup.  This is akin to the arguments and environment that you
specify for a program, but contains a system-dependent variety of
binary values that tell system libraries important details about the
hardware, operating system, and process.  Each value's purpose is
identified by an integer tag; the meanings are well-known but system-specific.
Depending on the configuration and operating system facilities,
GDB may be able to show you this information.  For remote
targets, this functionality may further depend on the remote stub's
support of the `qXfer:auxv:read` packet, see
[qXfer auxiliary vector read](General-Query-Packets.html#qXfer-auxiliary-vector-read).

- `info auxv`
   Display the auxiliary vector of the inferior, which can be either a
   live process or a core dump file.  GDB prints each tag value
   numerically, and also shows names and text descriptions for recognized
   tags.  Some values in the vector are numbers, some bit masks, and some
   pointers to strings or other data.  GDB displays each value in the
   most appropriate form for a recognized tag, and in hexadecimal for
   an unrecognized tag.

   On some targets, GDB can access operating system-specific
   information and show it to you.  The types of information available
   will differ depending on the type of operating system running on the
   target.  The mechanism used to fetch the data is described in
   [Operating System Information](Operating-System-Information.html#Operating-System-Information).
   For remote targets, this functionality depends on the remote stub's support of the
   `qXfer:osdata:read` packet, see [qXfer osdata read](General-Query-Packets.html#qXfer-osdata-read).

- `info os infotype`
   Display OS information of the requested type.

   On GNU/Linux, the following values of infotype are valid:

- `cpus`
   Display the list of all CPUs/cores. For each CPU/core, GDB prints
   the available fields from /proc/cpuinfo. For each supported architecture
   different fields are available. Two common entries are processor which gives
   CPU number and bogomips; a system constant that is calculated during
   kernel initialization.

- `files`
   Display the list of open file descriptors on the target.  For each
   file descriptor, GDB prints the identifier of the process
   owning the descriptor, the command of the owning process, the value
   of the descriptor, and the target of the descriptor.

- `modules`
   Display the list of all loaded kernel modules on the target.  For each
   module, GDB prints the module name, the size of the module in
   bytes, the number of times the module is used, the dependencies of the
   module, the status of the module, and the address of the loaded module
   in memory.

- `msg`
   Display the list of all System V message queues on the target.  For each
   message queue, GDB prints the message queue key, the message
   queue identifier, the access permissions, the current number of bytes
   on the queue, the current number of messages on the queue, the processes
   that last sent and received a message on the queue, the user and group
   of the owner and creator of the message queue, the times at which a
   message was last sent and received on the queue, and the time at which
   the message queue was last changed.

- `processes`
   Display the list of processes on the target.  For each process,
   GDB prints the process identifier, the name of the user, the
   command corresponding to the process, and the list of processor cores
   that the process is currently running on.  (To understand what these
   properties mean, for this and the following info types, please consult
   general GNU/Linux documentation.)

- `procgroups`
   Display the list of process groups on the target.  For each process,
   GDB prints the identifier of the process group that it belongs
   to, the command corresponding to the process group leader, the process
   identifier, and the command line of the process.  The list is sorted
   first by the process group identifier, then by the process identifier,
   so that processes belonging to the same process group are grouped together
   and the process group leader is listed first.

- `semaphores`
   Display the list of all System V semaphore sets on the target.  For each
   semaphore set, GDB prints the semaphore set key, the semaphore
   set identifier, the access permissions, the number of semaphores in the
   set, the user and group of the owner and creator of the semaphore set,
   and the times at which the semaphore set was operated upon and changed.

- `shm`
   Display the list of all System V shared-memory regions on the target.
   For each shared-memory region, GDB prints the region key,
   the shared-memory identifier, the access permissions, the size of the
   region, the process that created the region, the process that last
   to or detached from the region, the current number of live
   attaches to the region, and the times at which the region was last
   attached to, detach from, and changed.

- `sockets`
   Display the list of Internet-domain sockets on the target.  For each
   socket, GDB prints the address and port of the local and
   remote endpoints, the current state of the connection, the creator of
   the socket, the IP address family of the socket, and the type of the
   connection.

- `threads`
   Display the list of threads running on the target.  For each thread,
   GDB prints the identifier of the process that the thread
   belongs to, the command of the process, the thread identifier, and the
   processor core that it is currently running on.  The main thread of a
   process is not listed.

- `info os`
   If infotype is omitted, then list the possible values for
   infotype and the kind of OS information available for each
   infotype.  If the target does not return a list of possible
   types, this command will report an error.


## 10.17 Memory Region Attributes

*Memory region attributes* allow you to describe special handling
required by regions of your target's memory.  GDB uses
attributes to determine whether to allow certain types of memory
accesses; whether to use specific width accesses; and whether to cache
target memory.  By default the description of memory regions is
fetched from the target (if the current target supports this), but the
user can override the fetched regions.

Defined memory regions can be individually enabled and disabled.  When a
memory region is disabled, GDB uses the default attributes when
accessing memory in that region.  Similarly, if no memory regions have
been defined, GDB uses the default attributes when accessing
all memory.

When a memory region is defined, it is given a number to identify it;
to enable, disable, or remove a memory region, you specify that number.

- `mem lowerupperattributes&hellip;`
   Define a memory region bounded by lower and upper with
   attributes attributes&hellip;, and add it to the list of regions
   monitored by GDB.  Note that upper == 0 is a special
   case: it is treated as the target's maximum memory address.
   (0xffff on 16 bit targets, 0xffffffff on 32 bit targets, etc.)

- `mem auto`
   Discard any user changes to the memory regions and use target-supplied
   regions, if available, or no regions if the target does not support.

- `delete mem nums&hellip;`
   Remove memory regions nums&hellip; from the list of regions
   monitored by GDB.

- `disable mem nums&hellip;`
   Disable monitoring of memory regions nums&hellip;.
   A disabled memory region is not forgotten.
   It may be enabled again later.

- `enable mem nums&hellip;`
   Enable monitoring of memory regions nums&hellip;.

- `info mem`
   Print a table of all defined memory regions, with the following columns
   for each region:

- *Memory Region Number*
  *Enabled or Disabled.*
   Enabled memory regions are marked with `y`.
   Disabled memory regions are marked with `n`.

- *Lo Address*
   The address defining the inclusive lower bound of the memory region.

- *Hi Address*
   The address defining the exclusive upper bound of the memory region.

- *Attributes*
   The list of attributes set for this memory region.

#### 10.17.1 Attributes

#### 10.17.1.1 Memory Access Mode

The access mode attributes set whether GDB may make read or
write accesses to a memory region.

While these attributes prevent GDB from performing invalid
memory accesses, they do nothing to prevent the target system, I/O DMA,
etc. from accessing memory.

- `ro`
   Memory is read only.

- `wo`
   Memory is write only.

- `rw`
   Memory is read/write.  This is the default.


#### 10.17.1.2 Memory Access Size

The access size attribute tells GDB to use specific sized
accesses in the memory region.  Often memory mapped device registers
require specific sized accesses.  If no access size attribute is
specified, GDB may use accesses of any size.

- `8`
   Use 8 bit memory accesses.

- `16`
   Use 16 bit memory accesses.

- `32`
   Use 32 bit memory accesses.

- `64`
   Use 64 bit memory accesses.


#### 10.17.1.3 Data Cache

The data cache attributes set whether GDB will cache target
memory.  While this generally improves performance by reducing debug
protocol overhead, it can lead to incorrect results because GDB
does not know about volatile variables or memory mapped device
registers.

- `cache`
   Enable GDB to cache target memory.

- `nocache`
   Disable GDB from caching target memory.  This is the default.


#### 10.17.2 Memory Access Checking

GDB can be instructed to refuse accesses to memory that is
not explicitly described.  This can be useful if accessing such
regions has undesired effects for a specific target, or to provide
better error checking.  The following commands control this behaviour.

- `set mem inaccessible-by-default [on|off]`
   If `on` is specified, make  GDB treat memory not
   explicitly described by the memory ranges as non-existent and refuse accesses
   to such memory.  The checks are only performed if there's at least one
   memory range defined.  If `off` is specified, make GDB
   treat the memory not explicitly described by the memory ranges as RAM.
   The default value is `on`.

- `show mem inaccessible-by-default`
   Show the current handling of accesses to unknown memory.


## 10.18 Copy Between Memory and a File

You can use the commands `dump`, `append`, and
`restore` to copy data between target memory and a file.  The
`dump` and `append` commands write data to a file, and the
`restore` command reads data from a file back into the inferior's
memory.  Files may be in binary, Motorola S-record, Intel hex,
Tektronix Hex, or Verilog Hex format; however, GDB can only
append to binary files, and cannot read from Verilog Hex files.

- `dump [format] memory filenamestart_addrend_addr`
  `dump [format] value filenameexpr`
   Dump the contents of memory from start_addr to end_addr,
   or the value of expr, to filename in the given format.

   The format parameter may be any one of:

- `binary`
   Raw binary form.

- `ihex`
   Intel hex format.

- `srec`
   Motorola S-record format.

- `tekhex`
   Tektronix Hex format.

- `verilog`
   Verilog Hex format.

   GDB uses the same definitions of these formats as the
   GNU binary utilities, like `objdump` and `objcopy`.  If
   format is omitted, GDB dumps the data in raw binary
   form.

- `append [binary] memory filenamestart_addrend_addr` 
  `append [binary] value filenameexpr`
   Append the contents of memory from start_addr to end_addr,
   or the value of expr, to the file filename, in raw binary form.
   (GDB can only append data to files in raw binary form.)

- `restore filename[binary]biasstartend`
   Restore the contents of file filename into memory.  The
   `restore` command can automatically recognize any known BFD
   file format, except for raw binary.  To restore a raw binary file you
   must specify the optional keyword `binary` after the filename.

   If bias is non-zero, its value will be added to the addresses
   contained in the file.  Binary files always start at address zero, so
   they will be restored at address bias.  Other bfd files have
   a built-in location; they will be restored at offset bias
   from that location.

   If start and/or end are non-zero, then only data between
   file offset start and file offset end will be restored.
   These offsets are relative to the addresses in the file, before
   the bias argument is applied.


## 10.19 How to Produce a Core File from Your Program

A *core file* or *core dump* is a file that records the memory
image of a running process and its process status (register values
etc.).  Its primary use is post-mortem debugging of a program that
crashed while it ran outside a debugger.  A program that crashes
automatically produces a core file, unless this feature is disabled by
the user.  See [Files](Files.html#Files), for information on invoking GDB in
the post-mortem debugging mode.

Occasionally, you may wish to produce a core file of the program you
are debugging in order to preserve a snapshot of its state.
GDB has a special command for that.

- `generate-core-file [file]``gcore [file]`
   Produce a core dump of the inferior process.  The optional argument
   file specifies the file name where to put the core dump.  If not
   specified, the file name defaults to core.pid, where
   pid is the inferior process ID.

   Note that this command is implemented only for some systems (as of
   this writing, GNU/Linux, FreeBSD, Solaris, and S390).

   On GNU/Linux, this command can take into account the value of the
   file /proc/pid/coredump_filter when generating the core
   dump (see [set use-coredump-filter](#set-use_002dcoredump_002dfilter)), and by default honors the
   `VM_DONTDUMP` flag for mappings where it is present in the file
   /proc/pid/smaps (see [set dump-excluded-mappings](#set-dump_002dexcluded_002dmappings)).

- `set use-coredump-filter on` 
  `set use-coredump-filter off`
   Enable or disable the use of the file
   /proc/pid/coredump_filter when generating core dump
   files.  This file is used by the Linux kernel to decide what types of
   memory mappings will be dumped or ignored when generating a core dump
   file.  pid is the process ID of a currently running process.

   To make use of this feature, you have to write in the
   /proc/pid/coredump_filter file a value, in hexadecimal,
   which is a bit mask representing the memory mapping types.  If a bit
   is set in the bit mask, then the memory mappings of the corresponding
   types will be dumped; otherwise, they will be ignored.  This
   configuration is inherited by child processes.  For more information
   about the bits that can be set in the
   /proc/pid/coredump_filter file, please refer to the
   manpage of `core(5)`.

   By default, this option is `on`.  If this option is turned
   `off`, GDB does not read the coredump_filter file
   and instead uses the same default value as the Linux kernel in order
   to decide which pages will be dumped in the core dump file.  This
   value is currently `0x33`, which means that bits `0`
   (anonymous private mappings), `1` (anonymous shared mappings),
   `4` (ELF headers) and `5` (private huge pages) are active.
   This will cause these memory mappings to be dumped automatically.

- `set dump-excluded-mappings on`
  `set dump-excluded-mappings off`
   If `on` is specified, GDB will dump memory mappings
   marked with the `VM_DONTDUMP` flag.  This flag is represented in
   the file /proc/pid/smaps with the acronym `dd`.

   The default value is `off`.


## 10.20 Character Sets

If the program you are debugging uses a different character set to
represent characters and strings than the one GDB uses itself,
GDB can automatically translate between the character sets for
you.  The character set GDB uses we call the *host
character set*; the one the inferior program uses we call the
*target character set*.

For example, if you are running GDB on a GNU/Linux system, which
uses the ISO Latin 1 character set, but you are using GDB's
remote protocol (see [Remote Debugging](Remote-Debugging.html#Remote-Debugging)) to debug a program
running on an IBM mainframe, which uses the EBCDIC character set,
then the host character set is Latin-1, and the target character set is
EBCDIC.  If you give GDB the command `set target-charset EBCDIC-US`, then GDB translates between
EBCDIC and Latin 1 as you print character or string values, or use
character and string literals in expressions.

GDB has no way to automatically recognize which character set
the inferior program uses; you must tell it, using the `set target-charset`
command, described below.

Here are the commands for controlling GDB's character set
support:

- `set target-charset charset`
   Set the current target character set to charset.  To display the
   list of supported target character sets, type
   set target-charset TABTAB.

- `set host-charset charset`
   Set the current host character set to charset.

   By default, GDB uses a host character set appropriate to the
   system it is running on; you can override that default using the
   `set host-charset` command.  On some systems, GDB cannot
   automatically determine the appropriate host character set.  In this
   case, GDB uses `UTF-8`.

   GDB can only use certain character sets as its host character
   set.  If you type set host-charset TABTAB,
   GDB will list the host character sets it supports.

- `set charset charset`
   Set the current host and target character sets to charset.  As
   above, if you type set charset TABTAB,
   GDB will list the names of the character sets that can be used
   for both host and target.

- `show charset`
   Show the names of the current host and target character sets.

- `show host-charset`
   Show the name of the current host character set.

- `show target-charset`
   Show the name of the current target character set.

- `set target-wide-charset charset`
   Set the current target's wide character set to charset.  This is
   the character set used by the target's `wchar_t` type.  To
   display the list of supported wide character sets, type
   set target-wide-charset TABTAB.

- `show target-wide-charset`
   Show the name of the current target's wide character set.

Here is an example of GDB's character set support in action.
Assume that the following source code has been placed in the file
charset-test.c:

```c
#include <stdio.h>

char ascii_hello[]
  = {72, 101, 108, 108, 111, 44, 32, 119,
     111, 114, 108, 100, 33, 10, 0};
char ibm1047_hello[]
  = {200, 133, 147, 147, 150, 107, 64, 166,
     150, 153, 147, 132, 90, 37, 0};

main ()
{
  printf ("Hello, world!\n");
}
```

In this program, `ascii_hello` and `ibm1047_hello` are arrays
containing the string `Hello, world!` followed by a newline,
encoded in the ASCII and IBM1047 character sets.

We compile the program, and invoke the debugger on it:

    $ gcc -g charset-test.c -o charset-test
    $ gdb -nw charset-test
    GNU gdb 2001-12-19-cvs
    Copyright 2001 Free Software Foundation, Inc.
    &hellip;
    (gdb)

We can use the `show charset` command to see what character sets
GDB is currently using to interpret and display characters and
strings:

    (gdb) show charset
    The current host and target character set is `ISO-8859-1'.
    (gdb)

For the sake of printing this manual, let's use ASCII as our
initial character set:

    (gdb) set charset ASCII
    (gdb) show charset
    The current host and target character set is `ASCII'.
    (gdb)

Let's assume that ASCII is indeed the correct character set for our
host system &mdash; in other words, let's assume that if GDB prints
characters using the ASCII character set, our terminal will display
them properly.  Since our current target character set is also
ASCII, the contents of `ascii_hello` print legibly:

    (gdb) print ascii_hello
    $1 = 0x401698 "Hello, world!\n"
    (gdb) print ascii_hello[0]
    $2 = 72 'H'
    (gdb)

GDB uses the target character set for character and string
literals you use in expressions:

    (gdb) print '+'
    $3 = 43 '+'
    (gdb)

The ASCII character set uses the number 43 to encode the `+` character.

GDB relies on the user to tell it which character set the
target program uses.  If we print `ibm1047_hello` while our target
character set is still ASCII, we get jibberish:

    (gdb) print ibm1047_hello
    $4 = 0x4016a8 "\310\205\223\223\226k@\246\226\231\223\204Z%"
    (gdb) print ibm1047_hello[0]
    $5 = 200 '\310'
    (gdb)

If we invoke the `set target-charset` followed by TABTAB,
GDB tells us the character sets it supports:

    (gdb) set target-charset
    ASCII       EBCDIC-US   IBM1047     ISO-8859-1
    (gdb) set target-charset

We can select IBM1047 as our target character set, and examine the
program's strings again.  Now the ASCII string is wrong, but
GDB translates the contents of `ibm1047_hello` from the
target character set, IBM1047, to the host character set,
ASCII, and they display correctly:

    (gdb) set target-charset IBM1047
    (gdb) show charset
    The current host character set is `ASCII'.
    The current target character set is `IBM1047'.
    (gdb) print ascii_hello
    $6 = 0x401698 "\110\145%%?\054\040\167?\162%\144\041\012"
    (gdb) print ascii_hello[0]
    $7 = 72 '\110'
    (gdb) print ibm1047_hello
    $8 = 0x4016a8 "Hello, world!\n"
    (gdb) print ibm1047_hello[0]
    $9 = 200 'H'
    (gdb)

As above, GDB uses the target character set for character and
string literals you use in expressions:

    (gdb) print '+'
    $10 = 78 '+'
    (gdb)

The IBM1047 character set uses the number 78 to encode the `+` character.


## 10.21 Caching Data of Targets

GDB caches data exchanged between the debugger and a target.
Each cache is associated with the address space of the inferior.
See [Inferiors and Programs](Inferiors-and-Programs.html#Inferiors-and-Programs),
about inferior and address space.
Such caching generally improves performance in remote debugging
(see [Remote Debugging](Remote-Debugging.html#Remote-Debugging)),
because it reduces the overhead of the
remote protocol by bundling memory reads and writes into large chunks.
Unfortunately, simply caching everything would lead to incorrect results,
since GDB does not necessarily know anything about volatile
values, memory-mapped I/O addresses, etc.  Furthermore, in non-stop mode
(see [Non-Stop Mode](Non_002dStop-Mode.html#Non_002dStop-Mode)) 
memory can be changed *while* a gdb command is executing.
Therefore, by default, GDB only caches data
known to be on the stack[12](#FOOT12) or
in the code segment.
Other regions of memory can be explicitly marked as
cacheable; see [Memory Region Attributes](Memory-Region-Attributes.html#Memory-Region-Attributes).

- `set remotecache on`
  `set remotecache off`
   This option no longer does anything; it exists for compatibility
   with old scripts.

- `show remotecache`
   Show the current state of the obsolete remotecache flag.

- `set stack-cache on`
- `set stack-cache off`
   Enable or disable caching of stack accesses.  When `on`, use
   caching.  By default, this option is `on`.

- `show stack-cache`
   Show the current state of data caching for memory accesses.

`set code-cache on``set code-cache off`
   Enable or disable caching of code segment accesses.  When `on`,
   use caching.  By default, this option is `on`.  This improves
   performance of disassembly in remote debugging.

`show code-cache`
   Show the current state of target memory cache for code segment accesses.

- `info dcache [line]`
   Print the information about the performance of data cache of the
   current inferior&rsquo;s address space.  The information displayed
   includes the dcache width and depth, and for each cache line, its
   number, address, and how many times it was referenced.  This
   command is useful for debugging the data cache operation.

   If a line number is specified, the contents of that line will be
   printed in hex.

- `set dcache size size`
   Set maximum number of entries in dcache (dcache depth above).

- `set dcache line-size line-size`
   Set number of bytes each dcache entry caches (dcache width above).
   Must be a power of 2.

- `show dcache size`
   Show maximum number of dcache entries.  See [info dcache](#Caching-Target-Data).

- `show dcache line-size`
   Show default size of dcache lines.

---

#### Footnotes

### [(12)](#DOCF12)

In non-stop mode, it is moderately
rare for a running thread to modify the stack of a stopped thread
in a way that would interfere with a backtrace, and caching of
stack reads provides a significant speed up of remote backtraces.


## 10.22 Search Memory

Memory can be searched for a particular sequence of bytes with the
`find` command.

- `find [/sn]start_addr, +len, val1[, val2, &hellip;]`
  `find [/sn]start_addr, end_addr, val1[, val2, &hellip;]`
   Search memory for the sequence of bytes specified by val1, val2,
   etc.  The search begins at address start_addr and continues for either
   len bytes or through to end_addr inclusive.

   `s` and `n` are optional parameters.
   They may be specified in either order, apart or together.

   `s`, search query size
   The size of each search query value.

- `b`
   bytes

- `h`
   halfwords (two bytes)

- `w`
   words (four bytes)

- `g`
   giant words (eight bytes)

All values are interpreted in the current language.
This means, for example, that if the current source language is C/C++
then searching for the string `hello` includes the trailing `\0`.
The null terminator can be removed from searching by using casts,
e.g.: `{char[5]}"hello"`.

If the value size is not specified, it is taken from the
value's type in the current language.
This is useful when one wants to specify the search
pattern as a mixture of types.
Note that this means, for example, that in the case of C-like languages
a search for an untyped 0x42 will search for `(int) 0x42`
which is typically four bytes.

n, maximum number of finds
The maximum number of matches to print.  The default is to print all finds.

You can use strings as search values.  Quote them with double-quotes
 (`"`).
The string value is copied into the search pattern byte by byte,
regardless of the endianness of the target and the size specification.

The address of each match found is printed as well as a count of the
number of matches found.

The address of the last value found is stored in convenience variable
`$\_`.
A count of the number of matches is stored in `$numfound`.

For example, if stopped at the `printf` in this function:

```c
void hello ()
{
  static char hello[] = "hello-hello";
  static struct { char c; short s; int i; }
    __attribute__ ((packed)) mixed
    = { 'c', 0x1234, 0x87654321 };
  printf ("%s\n", hello);
}
```

you get during debugging:

    (gdb) find &hello[0], +sizeof(hello), "hello"
    0x804956d <hello.1620+6>
    1 pattern found
    (gdb) find &hello[0], +sizeof(hello), 'h', 'e', 'l', 'l', 'o'
    0x8049567 <hello.1620>
    0x804956d <hello.1620+6>
    2 patterns found.
    (gdb) find &hello[0], +sizeof(hello), {char[5]}"hello"
    0x8049567 <hello.1620>
    0x804956d <hello.1620+6>
    2 patterns found.
    (gdb) find /b1 &hello[0], +sizeof(hello), 'h', 0x65, 'l'
    0x8049567 <hello.1620>
    1 pattern found
    (gdb) find &mixed, +sizeof(mixed), (char) 'c', (short) 0x1234, (int) 0x87654321
    0x8049560 <mixed.1625>
    1 pattern found
    (gdb) print $numfound
    $1 = 1
    (gdb) print $_
    $2 = (void *) 0x8049560


## 10.23 Value Sizes

Whenever GDB prints a value memory will be allocated within
GDB to hold the contents of the value.  It is possible in
some languages with dynamic typing systems, that an invalid program
may indicate a value that is incorrectly large, this in turn may cause
GDB to try and allocate an overly large ammount of memory.

- `set max-value-size bytes`
  `set max-value-size unlimited`
   Set the maximum size of memory that GDB will allocate for the
   contents of a value to bytes, trying to display a value that
   requires more memory than that will result in an error.

   Setting this variable does not effect values that have already been
   allocated within GDB, only future allocations.

   There's a minimum size that `max-value-size` can be set to in
   order that GDB can still operate correctly, this minimum is
   currently 16 bytes.

   The limit applies to the results of some subexpressions as well as to
   complete expressions.  For example, an expression denoting a simple
   integer component, such as `x.y.z`, may fail if the size of
   x.y is dynamic and exceeds bytes.  On the other hand,
   GDB is sometimes clever; the expression `A[i]`, where
   A is an array variable with non-constant size, will generally
   succeed regardless of the bounds on A, as long as the component
   size is less than bytes.

   The default value of `max-value-size` is currently 64k.

- `show max-value-size`
   Show the maximum size of memory, in bytes, that GDB will
   allocate for the contents of a value.
