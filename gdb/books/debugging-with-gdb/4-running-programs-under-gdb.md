[]()

# 4 Running Programs Under GDB

When you run a program under GDB, you must first generate debugging information when you compile it.

You may start GDB with its arguments, if any, in an environment of your choice. If you are doing 
native debugging, you may redirect your program’s input and output, debug an already running 
process, or kill a child process.


### 4.1 Compiling for Debugging

In order to debug a program effectively, you need to generate debugging information when you compile 
it. This debugging information is stored in the object file; it describes the data type of each 
variable or function and the correspondence between source line numbers and addresses in the 
executable code.

To request debugging information, specify the ‘-g’ option when you run the compiler.

Programs that are to be shipped to your customers are compiled with optimizations, using the ‘-O’ 
compiler option. However, some compilers are unable to handle the ‘-g’ and ‘-O’ options together. 
Using those compilers, you cannot generate optimized executables containing debugging information.

GCC, the GNU C/C`++` compiler, supports ‘-g’ with or without ‘-O’, making it possible to debug 
optimized code. We recommend that you *always* use ‘-g’ whenever you compile a program. You may 
think your program is correct, but there is no sense in pushing your luck. For more information, see 
[Optimized Code](Optimized-Code.html#Optimized-Code).

Older versions of the GNU C compiler permitted a variant option ‘<span class="nolinebreak">-gg</
span>’ for debugging information. GDB no longer supports this format; if your GNU C compiler has 
this option, do not use it.

GDB knows about preprocessor macros and can show you their expansion (see [Macros]
(Macros.html#Macros)). Most compilers do not include information about preprocessor macros in the 
debugging information if you specify the -g flag alone. Version 3.1 and later of GCC, the GNU C 
compiler, provides macro information if you are using the DWARF debugging format, and specify the 
option -g3.

See [Options for Debugging Your Program or GCC](http://gcc.gnu.org/onlinedocs/gcc/Debugging-Options.html#Debugging-Options) 
in Using the GNU Compiler Collection (GCC), for more information on GCC options affecting debug 
information.

You will have the best debugging experience if you use the latest version of the DWARF debugging 
format that your compiler supports. DWARF is currently the most expressive and best supported 
debugging format in GDB.


## 4.2 Starting your Program

-   `run`
    `r`
    Use the `run` command to start your program under GDB. You must first specify the program name 
with an argument to GDB (see [Getting In and Out of GDB](Invocation.html#Invocation)), or by using 
the `file` or `exec-file` command (see [Commands to Specify Files](Files.html#Files)).

    If you are running your program in an execution environment that supports processes, `run` creates 
an inferior process and makes that process run your program. In some environments without processes, 
`run` jumps to the start of your program. Other targets, like ‘remote’, are always running. If you 
get an error message like this one:

    ``` smallexample
    The "remote" target does not support "run".
    Try "help target" or "continue".
    ```

    then use `continue` to run your program. You may need `load` first (see [load](Target-
Commands.html#load)).

    The execution of a program is affected by certain information it receives from its superior. GDB 
provides ways to specify this information, which you must do *before* starting your program. (You 
can change it after starting your program, but such changes only affect your program the next time 
you start it.) This information may be divided into four categories:

    The *arguments.*  
    Specify the arguments to give your program as the arguments of the `run` command. If a shell is 
available on your target, the shell is used to pass the arguments, so that you may use normal 
conventions (such as wildcard expansion or variable substitution) in describing the arguments. In 
Unix systems, you can control which shell is used with the `SHELL` environment variable. If you do 
not define `SHELL`, GDB uses the default shell (/bin/sh). You can disable use of any shell with the 
`set startup-with-shell` command (see below for details).

    The *environment.*  
    Your program normally inherits its environment from GDB, but you can use the GDB commands `set 
environment` and `unset environment` to change parts of the environment that affect your program. 
See [Your Program’s Environment](Environment.html#Environment).

    The *working directory.*  
    You can set your program’s working directory with the command set cwd. If you do not set any 
working directory with this command, your program will inherit GDB’s working directory if native 
debugging, or the remote server’s working directory if remote debugging. See [Your Program’s Working 
Directory](Working-Directory.html#Working-Directory).

    The *standard input and output.*  
    Your program normally uses the same device for standard input and standard output as GDB is 
using. You can redirect input and output in the `run` command line, or you can use the `tty` command 
to set a different device for your program. See [Your Program’s Input and Output]
(Input_002fOutput.html#Input_002fOutput).

    *Warning:* While input and output redirection work, you cannot use pipes to pass the output of 
the program you are debugging to another program; if you attempt this, GDB is likely to wind up 
debugging the wrong program.

    When you issue the `run` command, your program begins to execute immediately. See [Stopping and 
Continuing](Stopping.html#Stopping), for discussion of how to arrange for your program to stop. Once 
your program has stopped, you may call functions in your program, using the `print` or `call` 
commands. See [Examining Data](Data.html#Data).

    If the modification time of your symbol file has changed since the last time GDB read its 
symbols, GDB discards its symbol table, and reads it again. When it does this, GDB tries to retain 
your current breakpoints.

-   `start`
    The name of the main procedure can vary from language to language. With C or C`++`, the main 
procedure name is always `main`, but other languages such as Ada do not require a specific name for 
their main procedure. The debugger provides a convenient way to start the execution of the program 
and to stop at the beginning of the main procedure, depending on the language used.

    The ‘start’ command does the equivalent of setting a temporary breakpoint at the beginning of 
the main procedure and then invoking the ‘run’ command.

    Some programs contain an *elaboration* phase where some startup code is executed before the main 
procedure is called. This depends on the languages used to write your program. In C`++`, for 
instance, constructors for static and global objects are executed before `main` is called. It is 
therefore possible that the debugger stops before reaching the main procedure. However, the 
temporary breakpoint will remain to halt execution.

    Specify the arguments to give to your program as arguments to the ‘start’ command. These 
arguments will be given verbatim to the underlying ‘run’ command. Note that the same arguments will 
be reused if no argument is provided during subsequent calls to ‘start’ or ‘run’.

    It is sometimes necessary to debug the program during elaboration. In these cases, using the 
`start` command would stop the execution of your program too late, as the program would have already 
completed the elaboration phase. Under these circumstances, either insert breakpoints in your 
elaboration code before running your program or use the `starti` command.

-   `starti`
    The ‘starti’ command does the equivalent of setting a temporary breakpoint at the first 
instruction of a program’s execution and then invoking the ‘run’ command. For programs containing an 
elaboration phase, the `starti` command will stop execution at the start of the elaboration phase.

-   `set exec-wrapper wrapper`
    `show exec-wrapper`
    `unset exec-wrapper`

    When ‘exec-wrapper’ is set, the specified wrapper is used to launch programs for debugging. GDB 
    starts your program with a shell command of the form exec wrapper program. Quoting is added to 
    program and its arguments, but not to wrapper, so you should add quotes if appropriate for your 
    shell. The wrapper runs until it executes your program, and then GDB takes control.
    
    You can use any program that eventually calls `execve` with its arguments as a wrapper. Several 
    standard Unix utilities do this, e.g. `env` and `nohup`. Any Unix shell script ending with `exec 
    "$@"` will also work.
    
    For example, you can use `env` to pass an environment variable to the debugged program, without 
    setting the variable in your shell’s environment:

    ``` smallexample
    (gdb) set exec-wrapper env 'LD_PRELOAD=libtest.so'
    (gdb) run
    ```
    This command is available when debugging locally on most targets, excluding DJGPP, Cygwin, MS Windows, and QNX Neutrino.

-   `set startup-with-shell`
    `set startup-with-shell on`
    `set startup-with-shell off`
    `show startup-with-shell`
    On Unix systems, by default, if a shell is available on your target, GDB) uses it to start your 
    program. Arguments of the `run` command are passed to the shell, which does variable substitution, 
    expands wildcard characters and performs redirection of I/O. In some circumstances, it may be useful 
    to disable such use of a shell, for example, when debugging the shell itself or diagnosing startup 
    failures such as:

    ``` smallexample
    (gdb) run
    Starting program: ./a.out
    During startup program terminated with signal SIGSEGV, Segmentation fault.
    ```

    which indicates the shell or the wrapper specified with ‘exec-wrapper’ crashed, not your 
    program. Most often, this is caused by something odd in your shell’s non-interactive mode 
    initialization file—such as .cshrc for C-shell, $.zshenv for the Z shell, or the file specified in 
    the ‘BASH\_ENV’ environment variable for BASH.

-   `set auto-connect-native-target`
    `set auto-connect-native-target on`
    `set auto-connect-native-target off`
    `show auto-connect-native-target`
    By default, if not connected to any target yet (e.g., with `target remote`), the `run` command 
    starts your program as a native process under GDB, on your local machine. If you’re sure you don’t 
    want to debug programs on your local machine, you can tell GDB to not connect to the native target 
    automatically with the `set auto-connect-native-target off` command.
    
    If `on`, which is the default, and if GDB is not connected to a target already, the `run` command 
    automaticaly connects to the native target, if one is available.
    
    If `off`, and if GDB is not connected to a target already, the `run` command fails with an error:

    ``` smallexample
    (gdb) run
    Don't know how to run.  Try "help target".
    ```
    If GDB is already connected to a target, GDB always uses it with the `run` command.
    
    In any case, you can explicitly connect to the native target with the `target native` command. For 
    example,

    ``` smallexample
    (gdb) set auto-connect-native-target off
    (gdb) run
    Don't know how to run.  Try "help target".
    (gdb) target native
    (gdb) run
    Starting program: ./a.out
    [Inferior 1 (process 10421) exited normally]
    ```
    In case you connected explicitly to the `native` target, GDB remains connected even if all inferiors 
    exit, ready for the next `run` command. Use the `disconnect` command to disconnect.
    
    Examples of other commands that likewise respect the `auto-connect-native-target` setting: `attach`, 
    `info proc`, `info os`.

-   `set disable-randomization`
    `set disable-randomization on`
    This option (enabled by default in GDB) will turn off the native randomization of the virtual 
    address space of the started program. This option is useful for multiple debugging sessions to make 
    the execution better reproducible and memory addresses reusable across debugging sessions.
    
    This feature is implemented only on certain targets, including GNU/Linux. On GNU/Linux you can get 
    the same behavior using
    
    ``` smallexample
    (gdb) set exec-wrapper setarch `uname -m` -R
    ```

-   `set disable-randomization off`

    Leave the behavior of the started executable unchanged. Some bugs rear their ugly heads only when 
    the program is loaded at certain addresses. If your bug disappears when you run the program under 
    GDB, that might be because GDB by default disables the address randomization on platforms, such as 
    GNU/Linux, which do that for stand-alone programs. Use set disable-randomization off to try to 
    reproduce such elusive bugs.
    
    On targets where it is available, virtual address space randomization protects the programs against 
    certain kinds of security attacks. In these cases the attacker needs to know the exact location of a 
    concrete executable code. Randomizing its location makes it impossible to inject jumps misusing a 
    code at its expected addresses.
    
    Prelinking shared libraries provides a startup performance advantage but it makes addresses in these 
    libraries predictable for privileged processes by having just unprivileged access at the target 
    system. Reading the shared library binary gives enough information for assembling the malicious code 
    misusing it. Still even a prelinked shared library can get loaded at a new random address just 
    requiring the regular relocation process during the startup. Shared libraries not already prelinked 
    are always loaded at a randomly chosen address.
    
    Position independent executables (PIE) contain position independent code similar to the shared 
    libraries and therefore such executables get loaded at a randomly chosen address upon startup. PIE 
    executables always load even already prelinked shared libraries at a random address. You can build 
    such executable using `gcc -fPIE -pie`.
    
    Heap (malloc storage), stack and custom mmap areas are always placed randomly (as long as the 
    randomization is enabled).

-   `show disable-randomization`
    Show the current setting of the explicit disable of the native randomization of the virtual address 
    space of the started program.
