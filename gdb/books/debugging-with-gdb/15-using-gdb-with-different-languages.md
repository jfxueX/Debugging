# 15 Using GDB with Different Languages

Although programming languages generally have common aspects, they are
rarely expressed in the same manner.  For instance, in ANSI C,
dereferencing a pointer `p` is accomplished by `*p`, but in
Modula-2, it is accomplished by `p^`.  Values can also be
represented (and displayed) differently.  Hex numbers in C appear as
&lsquo;0x1ae&rsquo;, while in Modula-2 they appear as &lsquo;1AEH&rsquo;.

Language-specific information is built into GDB for some languages,
allowing you to express operations like the above in your program&rsquo;s
native language, and allowing GDB to output values in a manner
consistent with the syntax of your program&rsquo;s native language.  The
language you use to build expressions is called the *working language*.


* [15.1 Switching Between Source Languages](#151-switching-between-source-languages)
    * [15.1.1 List of Filename Extensions and Languages](#1511-list-of-filename-extensions-and-languages)
    * [15.1.2 Setting the Working Language](#1512-setting-the-working-language)
    * [15.1.3 Having GDB Infer the Source Language](#1513-having-gdb-infer-the-source-language)
* [15.2 Displaying the Language](#152-displaying-the-language)
* [15.3 Type and Range Checking](#153-type-and-range-checking)
    * [15.3.1 An Overview of Type Checking](#1531-an-overview-of-type-checking)
    * [15.3.2 An Overview of Range Checking](#1532-an-overview-of-range-checking)
* [15.4 Supported Languages](#154-supported-languages)
    * [15.4.1 C and C++](#1541-c-and-c)
        * [15.4.1.1 C and C++ Operators](#15411-c-and-c-operators)
        * [15.4.1.2 C and C++ Constants](#15412-c-and-c-constants)
        * [15.4.1.3 C++ Expressions](#15413-c-expressions)
        * [15.4.1.4 C and C++ Defaults](#15414-c-and-c-defaults)
        * [15.4.1.5 C and C++ Type and Range Checks](#15415-c-and-c-type-and-range-checks)
        * [15.4.1.6 GDB and C](#15416-gdb-and-c)
        * [15.4.1.7 GDB Features for C++](#15417-gdb-features-for-c)
        * [15.4.1.8 Decimal Floating Point format](#15418-decimal-floating-point-format)
    * [15.4.2(TODO)](#1542todo)
    * [15.4.3 Go](#1543-go)
    * [15.4.4 Objective-C](#1544-objective-c)
        * [15.4.4.1 Method Names in Commands](#15441-method-names-in-commands)
        * [15.4.4.2 The Print Command With Objective-C](#15442-the-print-command-with-objective-c)
    * [15.4.5 ~ 10(TODO)](#1545--10todo)
* [15.5 Unsupported Languages](#155-unsupported-languages)


## 15.1 Switching Between Source Languages

There are two ways to control the working language&mdash;either have GDB
set it automatically, or select it manually yourself.  You can use the
`set language` command for either purpose.  On startup, GDB
defaults to setting the language automatically.  The working language is
used to determine how expressions you type are interpreted, how values
are printed, etc.

In addition to the working language, every source file that
GDB knows about has its own working language.  For some object
file formats, the compiler might indicate which language a particular
source file is in.  However, most of the time GDB infers the
language from the name of the file.  The language of a source file
controls whether C++ names are demangled&mdash;this way `backtrace` can
show each frame appropriately for its own language.  There is no way to
set the language of a source file from within GDB, but you can
set the language associated with a filename extension.  See [Displaying the Language](Show.html#Show).

This is most commonly a problem when you use a program, such
as `cfront` or `f2c`, that generates C but is written in
another language.  In that case, make the
program use `#line` directives in its C output; that way
GDB will know the correct language of the source code of the original
program, and will display that source code, not the generated C code.


### 15.1.1 List of Filename Extensions and Languages

If a source file name ends in one of the following extensions, then
GDB infers that its language is the one indicated.

- `.ada`  
`.ads`  
`.adb`  
`.a`  
Ada source file.

- `.c`  
C source file

- `.C`  
`.cc`  
`.cp`  
`.cpp`  
`.cxx`  
`.c++`  
C++ source file

- `.d`  
D source file

- `.m`  
Objective-C source file

- `.f`  
.F    
Fortran source file

- `.mod`  
Modula-2 source file

- `.s`  
.S  
Assembler source file.  This actually behaves almost like C, but
GDB does not skip over function prologues when stepping.

In addition, you may set the language associated with a filename
extension.  See [Displaying the Language](Show.html#Show).


### 15.1.2 Setting the Working Language

If you allow GDB to set the language automatically,
expressions are interpreted the same way in your debugging session and
your program.

If you wish, you may set the language manually.  To do this, issue the
command &lsquo;set language lang&rsquo;, where lang is the name of
a language, such as
`c` or `modula-2`.
For a list of the supported languages, type &lsquo;set language&rsquo;.

Setting the language manually prevents GDB from updating the working
language automatically.  This can lead to confusion if you try
to debug a program when the working language is not the same as the
source language, when an expression is acceptable to both
languages&mdash;but means different things.  For instance, if the current
source file were written in C, and GDB was parsing Modula-2, a
command such as:

    print a = b + c
    

might not have the effect you intended.  In C, this means to add
`b` and `c` and place the result in `a`.  The result
printed would be the value of `a`.  In Modula-2, this means to compare
`a` to the result of `b+c`, yielding a `BOOLEAN` value.


### 15.1.3 Having GDB Infer the Source Language

To have GDB set the working language automatically, use
&lsquo;set language local&rsquo; or &lsquo;set language auto&rsquo;.  GDB
then infers the working language.  That is, when your program stops in a
frame (usually by encountering a breakpoint), GDB sets the
working language to the language recorded for the function in that
frame.  If the language for a frame is unknown (that is, if the function
or block corresponding to the frame was defined in a source file that
does not have a recognized extension), the current working language is
not changed, and GDB issues a warning.

This may not seem necessary for most programs, which are written
entirely in one source language.  However, program modules and libraries
written in one source language can be used by a main program written in
a different source language.  Using &lsquo;set language auto&rsquo; in this
case frees you from having to set the working language manually.

## 15.2 Displaying the Language

The following commands help you find out which language is the
working language, and also what language source files were written in.

- `show language`  
Display the current working language.  This is the
language you can use with commands such as `print` to
build and compute expressions that may involve variables in your program.

- `info frame`  
Display the source language for this frame.  This language becomes the
working language if you use an identifier from this frame.
See [Information about a Frame](Frame-Info.html#Frame-Info), to identify the other
information listed here.

- `info source`  
Display the source language of this source file.
See [Examining the Symbol Table](Symbols.html#Symbols), to identify the other
information listed here.

In unusual circumstances, you may have source files with extensions
not in the standard list.  You can then set the extension associated
with a language explicitly:

- `set extension-language extlanguage`  
Tell GDB that source files with extension ext are to be
assumed as written in the source language language.

- `info extensions`  
List all the filename extensions and the associated languages.

## 15.3 Type and Range Checking

Some languages are designed to guard you against making seemingly common
errors through a series of compile- and run-time checks.  These include
checking the type of arguments to functions and operators and making
sure mathematical overflows are caught at run time.  Checks such as
these help to ensure a program&rsquo;s correctness once it has been compiled
by eliminating type mismatches and providing active checks for range
errors when your program is running.

By default GDB checks for these errors according to the
rules of the current source language.  Although GDB does not check
the statements in your program, it can check expressions entered directly
into GDB for evaluation via the `print` command, for example.

### 15.3.1 An Overview of Type Checking

Some languages, such as C and C++, are strongly typed, meaning that the
arguments to operators and functions have to be of the correct type,
otherwise an error occurs.  These checks prevent type mismatch
errors from ever causing any run-time problems.  For example,

<pre>
int klass::my_method(char \*b) { return  b ? 1 : 2; }

(gdb) <b>print obj.my_method (0)</b>
$1 = 2


but


(gdb) <b>print obj.my_method (0x1234)</b>
Cannot resolve method klass::my_method to any overloaded instance
</pre>
    

The second example fails because in C++ the integer constant
&lsquo;0x1234&rsquo; is not type-compatible with the pointer parameter type.

For the expressions you use in GDB commands, you can tell
GDB to not enforce strict type checking or
to treat any mismatches as errors and abandon the expression;
When type checking is disabled, GDB successfully evaluates
expressions like the second example above.

Even if type checking is off, there may be other reasons
related to type that prevent GDB from evaluating an expression.
For instance, GDB does not know how to add an `int` and
a `struct foo`.  These particular type errors have nothing to do
with the language in use and usually arise from expressions which make
little sense to evaluate anyway.

GDB provides some additional commands for controlling type checking:

- `set check type on`   
`set check type off`  
Set strict type checking on or off.  If any type mismatches occur in
evaluating an expression while type checking is on, GDB prints a
message and aborts evaluation of the expression.

- `show check type`  
Show the current setting of type checking and whether GDB
is enforcing strict type checking rules.


### 15.3.2 An Overview of Range Checking

In some languages (such as Modula-2), it is an error to exceed the
bounds of a type; this is enforced with run-time checks.  Such range
checking is meant to ensure program correctness by making sure
computations do not overflow, or indices on an array element access do
not exceed the bounds of the array.

For expressions you use in GDB commands, you can tell
GDB to treat range errors in one of three ways: ignore them,
always treat them as errors and abandon the expression, or issue
warnings but evaluate the expression anyway.

A range error can result from numerical overflow, from exceeding an
array index bound, or when you type a constant that is not a member
of any type.  Some languages, however, do not treat overflows as an
error.  In many implementations of C, mathematical overflow causes the
result to &ldquo;wrap around&rdquo; to lower values&mdash;for example, if m is
the largest integer value, and s is the smallest, then

    m + 1 &rArr; s

This, too, is specific to individual languages, and in some cases
specific to individual compilers or machines.  See [Supported Languages](Supported-Languages.html#Supported-Languages), for further details on specific languages.

GDB provides some additional commands for controlling the range checker:

- `set check range auto`  
Set range checking on or off based on the current working language.
See [Supported Languages](Supported-Languages.html#Supported-Languages), for the default settings for
each language.

- `set check range on`  
`set check range off`  
Set range checking on or off, overriding the default setting for the
current working language.  A warning is issued if the setting does not
match the language default.  If a range error occurs and range checking is on,
then a message is printed and evaluation of the expression is aborted.

- `set check range warn`  
Output messages when the GDB range checker detects a range error,
but attempt to evaluate the expression anyway.  Evaluating the
expression may still be impossible for other reasons, such as accessing
memory that the process does not own (a typical example from many Unix
systems).

- `show range`  
Show the current setting of the range checker, and whether or not it is
being set automatically by GDB.

## 15.4 Supported Languages

GDB supports C, C++, D, Go, Objective-C, Fortran,
OpenCL C, Pascal, Rust, assembly, Modula-2, and Ada.
Some GDB features may be used in expressions regardless of the
language you use: the GDB`@` and `::` operators,
and the &lsquo;{type}addr&rsquo; construct (see [Expressions](Expressions.html#Expressions)) can be used with the constructs of any supported
language.

The following sections detail to what degree each source language is
supported by GDB.  These sections are not meant to be language
tutorials or references, but serve only as a reference guide to what the
GDB expression parser accepts, and what input and output
formats should look like for different languages.  There are many good
books written on each of these languages; please look to these for a
language reference or tutorial.

### 15.4.1 C and C++

Since C and C++ are so closely related, many features of GDB apply
to both languages.  Whenever this is the case, we discuss those languages
together.

The C++ debugging facilities are jointly implemented by the C++
compiler and GDB.  Therefore, to debug your C++ code
effectively, you must compile your C++ programs with a supported
C++ compiler, such as GNU`g++`, or the HP ANSI C++
compiler (`aCC`).

#### 15.4.1.1 C and C++ Operators

Operators must be defined on values of specific types.  For instance,
`+` is defined on numbers, but not on structures.  Operators are
often defined on groups of types.

For the purposes of C and C++, the following definitions hold:

- *Integral types* include `int` with any of its storage-class
specifiers; `char`; `enum`; and, for C++, `bool`.

- *Floating-point types* include `float`, `double`, and
`long double` (if supported by the target platform).

- *Pointer types* include all types defined as `(type *)`.

- *Scalar types* include all of the above.

The following operators are supported.  They are listed here
in order of increasing precedence:

- `,`  
The comma or sequencing operator.  Expressions in a comma-separated list
are evaluated from left to right, with the result of the entire
expression being the last expression evaluated.

- `=`  
Assignment.  The value of an assignment expression is the value
assigned.  Defined on scalar types.

- `op=`  
Used in an expression of the form `a op= b`,
and translated to `a = a op b`.
`op=` and `=` have the same precedence.  The operator
op is any one of the operators `|`, `^`, `&`,
`<<`, `>>`, `+`, `-`, `*`, `/`, `%`.

- `?:`  
The ternary operator.  `a ? b : c` can be thought
of as:  if a then b else c.  The argument a
should be of an integral type.

- `||`  
Logical OR.  Defined on integral types.

- `&&`  
Logical AND.  Defined on integral types.

- `|`  
Bitwise OR.  Defined on integral types.

- `^`  
Bitwise exclusive-OR.  Defined on integral types.

- `&`  
Bitwise AND.  Defined on integral types.

- `==, !=`  
Equality and inequality.  Defined on scalar types.  The value of these
expressions is 0 for false and non-zero for true.

- `<, >, <=, >=`  
Less than, greater than, less than or equal, greater than or equal.
Defined on scalar types.  The value of these expressions is 0 for false
and non-zero for true.

- `<<, >>`  
left shift, and right shift.  Defined on integral types.

- `@`  
The GDB &ldquo;artificial array&rdquo; operator (see [Expressions](Expressions.html#Expressions)).

- `+, -`  
Addition and subtraction.  Defined on integral types, floating-point types and
pointer types.

- `*, /, %`  
Multiplication, division, and modulus.  Multiplication and division are
defined on integral and floating-point types.  Modulus is defined on
integral types.

- `++, --`  
Increment and decrement.  When appearing before a variable, the
operation is performed before the variable is used in an expression;
when appearing after it, the variable&rsquo;s value is used before the
operation takes place.

- `*`  
Pointer dereferencing.  Defined on pointer types.  Same precedence as
`++`.

- `&`  
Address operator.  Defined on variables.  Same precedence as `++`.  
For debugging C++, GDB implements a use of &lsquo;&&rsquo; beyond what is
allowed in the C++ language itself: you can use &lsquo;&(&ref)&rsquo;
to examine the address

    where a C++ reference variable (declared with &lsquo;&ref&rsquo;) is
stored.

- `-`  
Negative.  Defined on integral and floating-point types.  Same
precedence as `++`.

- `!`  
Logical negation.  Defined on integral types.  Same precedence as
`++`.

- `~`  
Bitwise complement operator.  Defined on integral types.  Same precedence as
`++`.

- `., ->`  
Structure member, and pointer-to-structure member.  For convenience,
GDB regards the two as equivalent, choosing whether to dereference a
pointer based on the stored type information.
Defined on `struct` and `union` data.

- `.*, ->*`  
Dereferences of pointers to members.

- `[]`  
Array indexing.  `a[i]` is defined as
`*(a+i)`.  Same precedence as `->`.

- `()`  
Function parameter list.  Same precedence as `->`.

- `::`  
C++ scope resolution operator.  Defined on `struct`, `union`,
and `class` types.

- `::`  
Doubled colons also represent the GDB scope operator
(see [Expressions](Expressions.html#Expressions)).  Same precedence as `::`,
above.

If an operator is redefined in the user code, GDB usually
attempts to invoke the redefined version instead of using the operator&rsquo;s
predefined meaning.

#### 15.4.1.2 C and C++ Constants

GDB allows you to express the constants of C and C++ in the
following ways:

-  Integer constants are a sequence of digits.  Octal constants are
specified by a leading &lsquo;0&rsquo; (i.e. zero), and hexadecimal constants
by a leading &lsquo;0x&rsquo; or &lsquo;0X&rsquo;.  Constants may also end with a letter
&lsquo;l&rsquo;, specifying that the constant should be treated as a
`long` value.

-  Floating point constants are a sequence of digits, followed by a decimal
point, followed by a sequence of digits, and optionally followed by an
exponent.  An exponent is of the form:
&lsquo;e[[+]|-]nnn&rsquo;, where nnn is another
sequence of digits.  The &lsquo;+&rsquo; is optional for positive exponents.
A floating-point constant may also end with a letter &lsquo;f&rsquo; or
&lsquo;F&rsquo;, specifying that the constant should be treated as being of
the `float` (as opposed to the default `double`) type; or with
a letter &lsquo;l&rsquo; or &lsquo;L&rsquo;, which specifies a `long double`
constant.

-  Enumerated constants consist of enumerated identifiers, or their
integral equivalents.

-  Character constants are a single character surrounded by single quotes
(`'`), or a number&mdash;the ordinal value of the corresponding character
(usually its ASCII value).  Within quotes, the single character may
be represented by a letter or by *escape sequences*, which are of
the form &lsquo;\nnn&rsquo;, where nnn is the octal representation
of the character&rsquo;s ordinal value; or of the form &lsquo;\x&rsquo;, where
&lsquo;x&rsquo; is a predefined special character&mdash;for example,
&lsquo;\n&rsquo; for newline.  

    Wide character constants can be written by prefixing a character
constant with &lsquo;L&rsquo;, as in C.  For example, &lsquo;L'x'&rsquo; is the wide
form of &lsquo;x&rsquo;.  The target wide character set is used when
computing the value of this constant (see [Character Sets](Character-Sets.html#Character-Sets)).

-  String constants are a sequence of character constants surrounded by
double quotes (`"`).  Any valid character constant (as described
above) may appear.  Double quotes within the string must be preceded by
a backslash, so for instance &lsquo;"a\"b'c"&rsquo; is a string of five
characters.  

    Wide string constants can be written by prefixing a string constant
with &lsquo;L&rsquo;, as in C.  The target wide character set is used when
computing the value of this constant (see [Character Sets](Character-Sets.html#Character-Sets)).

-  Pointer constants are an integral value.  You can also write pointers
to constants using the C operator &lsquo;&&rsquo;.

-  Array constants are comma-separated lists surrounded by braces &lsquo;{&rsquo;
and &lsquo;}&rsquo;; for example, &lsquo;{1,2,3}&rsquo; is a three-element array of
integers, &lsquo;{{1,2}, {3,4}, {5,6}}&rsquo; is a three-by-two array,
and &lsquo;{&"hi", &"there", &"fred"}&rsquo; is a three-element array of pointers.


#### 15.4.1.3 C++ Expressions

GDB expression handling can interpret most C++ expressions.

> *Warning:*GDB can only debug C++ code if you use
> the proper compiler and the proper debug format.  Currently,
> GDB works best when debugging C++ code that is compiled
> with the most recent version of GCC possible.  The DWARF
> debugging format is preferred; GCC defaults to this on most
> popular platforms.  Other compilers and/or debug formats are likely to
> work badly or not at all when using GDB to debug C++
> code.  See [Compilation](Compilation.html#Compilation).

1.  Member function calls are allowed; you can use expressions like  
    <pre>
count = aml->GetOriginal(x, y)
</pre>
    

2.  While a member function is active (in the selected stack frame), your
expressions have the same namespace available as the member function;
that is, GDB allows implicit references to the class instance
pointer `this` following the same rules as C++.  `using`
declarations in the current scope are also respected by GDB.

3.  You can call overloaded functions; GDB resolves the function
call to the right definition, with some restrictions.  GDB does not
perform overload resolution involving user-defined type conversions,
calls to constructors, or instantiations of templates that do not exist
in the program.  It also cannot handle ellipsis argument lists or
default arguments.  

    It does perform integral conversions and promotions, floating-point
promotions, arithmetic conversions, pointer conversions, conversions of
class objects to base classes, and standard conversions such as those of
functions or arrays to pointers; it requires an exact match on the
number of function arguments.  

    Overload resolution is always performed, unless you have specified
`set overload-resolution off`.  See [GDB Features for C++](Debugging-C-Plus-Plus.html#Debugging-C-Plus-Plus).  
You must specify `set overload-resolution off` in order to use an
explicit function signature to call an overloaded function, as in  
    <pre>p 'foo(char,int)'('x', 13)</pre>  

    The GDB command-completion facility can simplify this;
see [Command Completion](Completion.html#Completion).

4. GDB understands variables declared as C++ lvalue or rvalue
references; you can use them in expressions just as you do in C++
source&mdash;they are automatically dereferenced.  

    In the parameter list shown when GDB displays a frame, the values of
reference variables are not displayed (unlike other variables); this
avoids clutter, since references are often used for large structures.
The *address* of a reference variable is always shown, unless
you have specified &lsquo;set print address off&rsquo;.

5. GDB supports the C++ name resolution operator `::`&mdash;your
expressions can use it just as expressions in your program do.  Since
one scope may be defined in another, you can use `::` repeatedly if
necessary, for example in an expression like
&lsquo;scope1::scope2::name&rsquo;.  GDB also allows
resolving name scope by reference to source files, in both C and C++
debugging (see [Program Variables](Variables.html#Variables)).

6. GDB performs argument-dependent lookup, following the C++
specification.


#### 15.4.1.4 C and C++ Defaults

If you allow GDB to set range checking automatically, it
defaults to `off` whenever the working language changes to
C or C++.  This happens regardless of whether you or GDB
selects the working language.

If you allow GDB to set the language automatically, it
recognizes source files whose names end with .c, .C, or
.cc, etc, and when GDB enters code compiled from one of
these files, it sets the working language to C or C++.
See [Having GDB Infer the Source Language](Automatically.html#Automatically),
for further details.

#### 15.4.1.5 C and C++ Type and Range Checks

By default, when GDB parses C or C++ expressions, strict type
checking is used.  However, if you turn type checking off, GDB
will allow certain non-standard conversions, such as promoting integer
constants to pointers.

Range checking, if turned on, is done on mathematical operations.  Array
indices are not checked, since they are often used to index a pointer
that is not itself an array.


#### 15.4.1.6 GDB and C

The `set print union` and `show print union` commands apply to
the `union` type.  When set to &lsquo;on&rsquo;, any `union` that is
inside a `struct` or `class` is also printed.  Otherwise, it
appears as &lsquo;{...}&rsquo;.

The `@` operator aids in the debugging of dynamic arrays, formed
with pointers and a memory allocation function.  See [Expressions](Expressions.html#Expressions).


#### 15.4.1.7 GDB Features for C++

Some GDB commands are particularly useful with C++, and some are
designed specifically for use with C++.  Here is a summary:

`breakpoint menus`
When you want a breakpoint in a function whose name is overloaded,
GDB has the capability to display a menu of possible breakpoint
locations to help you specify which function definition you want.
See [Ambiguous Expressions](Ambiguous-Expressions.html#Ambiguous-Expressions).

- `rbreak regex`  
Setting breakpoints using regular expressions is helpful for setting
breakpoints on overloaded functions that are not members of any special
classes.

    See [Setting Breakpoints](Set-Breaks.html#Set-Breaks).

- `catch throw`   
`catch rethrow`  
`catch catch`  
Debug C++ exception handling using these commands.  See [Setting Catchpoints](Set-Catchpoints.html#Set-Catchpoints).

- `ptype typename`  
Print inheritance relationships as well as other information for type
typename.  

    See [Examining the Symbol Table](Symbols.html#Symbols).

- `info vtbl expression.`  
The `info vtbl` command can be used to display the virtual
method tables of the object computed by expression.  This shows
one entry per virtual table; there may be multiple virtual tables when
multiple inheritance is in use.

- `demangle name`  
Demangle name.  
See [Symbols](Symbols.html#Symbols), for a more complete description of the `demangle` command.

- `set print demangle`   
`show print demangle`   
`set print asm-demangle`   
`show print asm-demangle`  
Control whether C++ symbols display in their source form, both when
displaying code as C++ source and when displaying disassemblies.
See [Print Settings](Print-Settings.html#Print-Settings).

- `set print object`   
`show print object`  
Choose whether to print derived (actual) or declared types of objects.
See [Print Settings](Print-Settings.html#Print-Settings).

- `set print vtbl`   
`show print vtbl`  
Control the format for printing virtual function tables.
See [Print Settings](Print-Settings.html#Print-Settings).
(The `vtbl` commands do not work on programs compiled with the HP
ANSI C++ compiler (`aCC`).)

- `set overload-resolution on`  
Enable overload resolution for C++ expression evaluation.  The default
is on.  For overloaded functions, GDB evaluates the arguments
and searches for a function whose signature matches the argument types,
using the standard C++ conversion rules (see [C++ Expressions](C-Plus-Plus-Expressions.html#C-Plus-Plus-Expressions), for details).
If it cannot find a match, it emits a message.

- `set overload-resolution off`  
Disable overload resolution for C++ expression evaluation.  For
overloaded functions that are not class member functions, GDB
chooses the first function of the specified name that it finds in the
symbol table, whether or not its arguments are of the correct type.  For
overloaded functions that are class member functions, GDB
searches for a function whose signature *exactly* matches the
argument types.

- `show overload-resolution`  
Show the current setting of overload resolution.

- `Overloaded symbol names`  
You can specify a particular definition of an overloaded symbol, using
the same notation that is used to declare such symbols in C++: type
`symbol(types)` rather than just symbol.  You can
also use the GDB command-line word completion facilities to list the
available choices, or to finish the type list for you.
See [Command Completion](Completion.html#Completion), for details on how to do this.

- `Breakpoints in functions with ABI tags`  
The GNU C++ compiler introduced the notion of ABI &ldquo;tags&rdquo;, which
correspond to changes in the ABI of a type, function, or variable that
would not otherwise be reflected in a mangled name.  See
[https://developers.redhat.com/blog/2015/02/05/gcc5-and-the-c11-abi/](https://developers.redhat.com/blog/2015/02/05/gcc5-and-the-c11-abi/)
for more detail.

The ABI tags are visible in C++ demangled names.  For example, a
function that returns a std::string:

    std::string function(int);
    

when compiled for the C++11 ABI is marked with the `cxx11` ABI
tag, and GDB displays the symbol like this:

    function[abi:cxx11](int)
    

You can set a breakpoint on such functions simply as if they had no
tag.  For example:

<pre>
(gdb) <b>b function(int)</b>
Breakpoint 2 at 0x40060d: file main.cc, line 10.
(gdb) <b>info breakpoints</b>
Num     Type           Disp Enb Address    What
1       breakpoint     keep y   0x0040060d in function[abi:cxx11](int)
                                           at main.cc:10
</pre>
    

On the rare occasion you need to disambiguate between different ABI
tags, you can do so by simply including the ABI tag in the function
name, like:

    (gdb) b ambiguous[abi:other_tag](int)


#### 15.4.1.8 Decimal Floating Point format

GDB can examine, set and perform computations with numbers in
decimal floating point format, which in the C language correspond to the
`_Decimal32`, `_Decimal64` and `_Decimal128` types as
specified by the extension to support decimal floating-point arithmetic.

There are two encodings in use, depending on the architecture: BID (Binary
Integer Decimal) for x86 and x86-64, and DPD (Densely Packed Decimal) for
PowerPC and S/390.  GDB will use the appropriate encoding for the
configured target.

Because of a limitation in libdecnumber, the library used by GDB
to manipulate decimal floating point numbers, it is not possible to convert
(using a cast, for example) integers wider than 32-bit to decimal float.

In addition, in order to imitate GDB&rsquo;s behaviour with binary floating
point computations, error checking in decimal float operations ignores
underflow, overflow and divide by zero exceptions.

In the PowerPC architecture, GDB provides a set of pseudo-registers
to inspect `_Decimal128` values stored in floating point registers.
See [PowerPC](PowerPC.html#PowerPC) for more details.


### 15.4.2(TODO)

### 15.4.3 Go

GDB can be used to debug programs written in Go and compiled with
gccgo or 6g compilers.

Here is a summary of the Go-specific features and restrictions:

- `The current Go package`  
The name of the current package does not need to be specified when
specifying global variables and functions.

For example, given the program:

```go
package main
var myglob = "Shall we?"
func main () {
  // ...
}
```
    

When stopped inside `main` either of these work:

    (gdb) p myglob
    (gdb) p main.myglob
    

- `Builtin Go types`  
The `string` type is recognized by GDB and is printed
as a string.

- `Builtin Go functions`  
The GDB expression parser recognizes the `unsafe.Sizeof`
function and handles it internally.

- `Restrictions on Go expressions`  
All Go operators are supported except `&^`.
The Go `_` &ldquo;blank identifier&rdquo; is not supported.
Automatic dereferencing of pointers is not supported.

### 15.4.4 Objective-C

This section provides information about some commands and command
options that are useful for debugging Objective-C code.  See also
[info classes](Symbols.html#Symbols), and [info selectors](Symbols.html#Symbols), for a
few more commands specific to Objective-C support.

#### 15.4.4.1 Method Names in Commands

The following commands have been extended to accept Objective-C method
names as line specifications:

- `clear`
- `break`
- `info line`
- `jump`
- `list`

A fully qualified Objective-C method name is specified as

    -[ClassmethodName]
    

where the minus sign is used to indicate an instance method and a
plus sign (not shown) is used to indicate a class method.  The class
name Class and method name methodName are enclosed in
brackets, similar to the way messages are specified in Objective-C
source code.  For example, to set a breakpoint at the `create`
instance method of class `Fruit` in the program currently being
debugged, enter:

    break -[Fruit create]
    

To list ten program lines around the `initialize` class method,
enter:

    list +[NSText initialize]
    

In the current version of GDB, the plus or minus sign is
required.  In future versions of GDB, the plus or minus
sign will be optional, but you can use it to narrow the search.  It
is also possible to specify just a method name:

    break create
    

You must specify the complete method name, including any colons.  If
your program&rsquo;s source files contain more than one `create` method,
you&rsquo;ll be presented with a numbered list of classes that implement that
method.  Indicate your choice by number, or type &lsquo;0&rsquo; to exit if
none apply.

As another example, to clear a breakpoint established at the
`makeKeyAndOrderFront:` method of the `NSWindow` class, enter:

    clear -[NSWindow makeKeyAndOrderFront:]

#### 15.4.4.2 The Print Command With Objective-C

The print command has also been extended to accept methods.  For example:

    print -[object hash]
    

will tell GDB to send the `hash` message to object
and print the result.  Also, an additional command has been added,
`print-object` or `po` for short, which is meant to print
the description of an object.  However, this command may only work
with certain Objective-C libraries that have a particular hook
function, `_NSPrintForDebugger`, defined.

### 15.4.5 ~ 10(TODO)

## 15.5 Unsupported Languages

In addition to the other fully-supported programming languages,
GDB also provides a pseudo-language, called `minimal`.
It does not represent a real programming language, but provides a set
of capabilities close to what the C or assembly languages provide.
This should allow most simple operations to be performed while debugging
an application that uses a language currently not supported by GDB.

If the language is set to `auto`, GDB will automatically
select this language if the current frame corresponds to an unsupported
language.
