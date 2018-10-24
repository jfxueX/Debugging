# 25 GDB Text User Interface


* [25.1 TUI Overview](#251-tui-overview)
* [25.2 TUI Key Bindings](#252-tui-key-bindings)
* [25.3 TUI Single Key Mode](#253-tui-single-key-mode)
* [25.4 TUI-specific Commands](#254-tui-specific-commands)
* [25.5 TUI Configuration Variables](#255-tui-configuration-variables)

The GDB Text User Interface (TUI) is a terminal
interface which uses the `curses` library to show the source
file, the assembly output, the program registers and GDB
commands in separate text windows.  The TUI mode is supported only
on platforms where a suitable version of the `curses` library
is available.

The TUI mode is enabled by default when you invoke GDB as
&lsquo;gdb -tui&rsquo;.
You can also switch in and out of TUI mode while GDB runs by
using various TUI commands and key bindings, such as `tui enable` 
or `C-x` `C-a`.  See [TUI Commands](#254-tui-specific-commands), and
[TUI Key Bindings](#252-tui-key-bindings).


## 25.1 TUI Overview

In TUI mode, GDB can display several text windows:

- *command*  
This window is the GDB command window with the GDB
prompt and the GDB output. The GDB input is still
managed using readline.

- *source*  
The source window shows the source file of the program. The current
line and active breakpoints are displayed in this window.

- *assembly*  
The assembly window shows the disassembly output of the program.

- *register*  
This window shows the processor registers. Registers are highlighted
when their values change.

The source and assembly windows show the current program position
by highlighting the current line and marking it with a &lsquo;>&rsquo; marker.
Breakpoints are indicated with two markers. The first marker
indicates the breakpoint type:

- `B`  
Breakpoint which was hit at least once.

- `b`  
Breakpoint which was never hit.

- `H`  
Hardware breakpoint which was hit at least once.

- `h`  
Hardware breakpoint which was never hit.

The second marker indicates whether the breakpoint is enabled or not:

- `+`  
Breakpoint is enabled.

- `-`  
Breakpoint is disabled.

The source, assembly and register windows are updated when the current
thread changes, when the frame changes, or when the program counter
changes.

**These windows are not all visible at the same time.** The command
window is always visible. The others can be arranged in several
layouts:

-  source only,

-  assembly only,

-  source and assembly,

-  source and registers, or

-  assembly and registers.

A status line above the command window shows the following information:

- *target*  
Indicates the current GDB target.
(see [Specifying a Debugging Target](Targets.html#Targets)).

- *process*  
Gives the current process or thread number.
When no process is being debugged, this field is set to `No process`.

- *function*  
Gives the current function name for the selected frame.
The name is demangled if demangling is turned on (see [Print Settings](Print-Settings.html#Print-Settings)).
When there is no symbol corresponding to the current program counter,
the string `??` is displayed.

- *line*  
Indicates the current line number for the selected frame.
When the current line number is not known, the string `??` is displayed.

- *pc*  
Indicates the current program counter address.


## 25.2 TUI Key Bindings

The TUI installs several key bindings in the readline keymaps
(see [Command Line Editing](Command-Line-Editing.html#Command-Line-Editing)).
The following key bindings are installed for both TUI mode and the
GDB standard mode.

- `C-x C-a`  
`C-x a`  
`C-x A`  
Enter or leave the TUI mode.  When leaving the TUI mode,
the curses window management stops and GDB operates using
its standard mode, writing on the terminal directly.  When reentering
the TUI mode, control is given back to the curses windows.
The screen is then refreshed.

- `C-x 1`  
Use a TUI layout with only one window.  The layout will
either be &lsquo;source&rsquo; or &lsquo;assembly&rsquo;.  When the TUI mode
is not active, it will switch to the TUI mode.  
Think of this key binding as the Emacs `C-x 1` binding.

- `C-x 2`  
Use a TUI layout with at least two windows.  When the current
layout already has two windows, the next layout with two windows is used.
When a new layout is chosen, one window will always be common to the
previous layout and the new one.  
Think of it as the Emacs `C-x 2` binding.

- `C-x o`  
Change the active window.  The TUI associates several key bindings
(like scrolling and arrow keys) with the active window.  This command
gives the focus to the next TUI window.  
Think of it as the Emacs `C-x o` binding.

- `C-x s`  
Switch in and out of the TUI SingleKey mode that binds single
keys to GDB commands (see [TUI Single Key Mode](#253-tui-single-key-mode)).  
The following key bindings only work in the TUI mode:

- `PgUp`  
Scroll the active window one page up.

- `PgDn`  
Scroll the active window one page down.

- `Up`  
Scroll the active window one line up.

- `Down`  
Scroll the active window one line down.

- `Left`  
Scroll the active window one column left.

- `Right`  
Scroll the active window one column right.

- `C-L`  
Refresh the screen.

Because the arrow keys scroll the active window in the TUI mode, they
are not available for their normal use by readline unless the command
window has the focus.  When another window is active, you must use
other readline key bindings such as `C-p`, `C-n`, `C-b`
and `C-f` to control the command window.


## 25.3 TUI Single Key Mode

The TUI also provides a *SingleKey* mode, which binds several
frequently used GDB commands to single keys.  **Type `C-x s` to
switch into this mode**, where the following key bindings are used:

- `c`  
continue

- `d`  
down

- `f`  
finish

- `n`  
next

- `o`  
nexti.  The shortcut letter &lsquo;o&rsquo; stands for &ldquo;step `O`ver&rdquo;.

- `q`  
exit the SingleKey mode.

- `r`  
run

- `s`  
step

- `i`  
stepi.  The shortcut letter &lsquo;i&rsquo; stands for &ldquo;step `I`nto&rdquo;.

- `u`  
up

- `v`  
info locals

- `w`  
where

Other keys temporarily switch to the GDB command prompt.
The key that was pressed is inserted in the editing buffer so that
it is possible to type most GDB commands without interaction
with the TUI SingleKey mode.  Once the command is entered the TUI
SingleKey mode is restored.  The only way to permanently leave
this mode is by typing `q` or `C-x s`.


## 25.4 TUI-specific Commands

The TUI has specific commands to control the text windows.
These commands are always available, even when GDB is not in
the TUI mode.  When GDB is in the standard mode, most
of these commands will automatically switch to the TUI mode.

Note that if GDB&rsquo;s `stdout` is not connected to a
terminal, or GDB has been started with the machine interface
interpreter (see [The GDB/MI Interface](GDB_002fMI.html#GDB_002fMI)), most of
these commands will fail with an error, because it would not be
possible or desirable to enable curses window management.

- `tui enable`  
Activate TUI mode.  The last active TUI window layout will be used if
TUI mode has prevsiouly been used in the current debugging session,
otherwise a default layout is used.

- `tui disable`  
Disable TUI mode, returning to the console interpreter.

- `info win`  
List and give the size of all displayed windows.

- `layout name`  
Changes which TUI windows are displayed.  In each layout the command
window is always displayed, the name parameter controls which
additional windows are displayed, and can be any of the following:

    - `next`  
    Display the next layout.

    - `prev`  
    Display the previous layout.

    - `src`  
    Display the source and command windows.

    - `asm`  
    Display the assembly and command windows.

    - `split`  
    Display the source, assembly, and command windows.

    - `regs`  
When in `src` layout display the register, source, and command
windows.  When in `asm` or `split` layout display the
register, assembler, and command windows.

    - `focus name`  
    Changes which TUI window is currently active for scrolling.  The
name parameter can be any of the following:

    - `next`  
    Make the next window active for scrolling.

    - `prev`  
    Make the previous window active for scrolling.

    - `src`  
    Make the source window active for scrolling.

    - `asm`  
    Make the assembly window active for scrolling.

    - `regs`  
    Make the register window active for scrolling.

    - `cmd`  
    Make the command window active for scrolling.

- `refresh`  
Refresh the screen.  This is similar to typing C-L.

- `tui reg group`  
Changes the register group displayed in the tui register window to
group.  If the register window is not currently displayed this
command will cause the register window to be displayed.  The list of
register groups, as well as their order is target specific. The
following groups are available on most targets:

    - `next`  
    Repeatedly selecting this group will cause the display to cycle
through all of the available register groups.

    - `prev`  
    Repeatedly selecting this group will cause the display to cycle
through all of the available register groups in the reverse order to
next.

    - `general`  
    Display the general registers.

    - `float`  
    Display the floating point registers.

    - `system`  
    Display the system registers.

    - `vector`  
    Display the vector registers.

    - `all`  
    Display all registers.

- `update`  
Update the source window and the current execution point.

- `winheight name +count`  
`winheight name -count`  
Change the height of the window name by count
lines.  Positive counts increase the height, while negative counts
decrease it.  The name parameter can be one of `src` (the
source window), `cmd` (the command window), `asm` (the
disassembly window), or `regs` (the register display window).

- `tabset nchars`  
Set the width of tab stops to be nchars characters.  This
setting affects the display of TAB characters in the source and
assembly windows.


## 25.5 TUI Configuration Variables

Several configuration variables control the appearance of TUI windows.

- `set tui border-kind kind`  
Select the border appearance for the source, assembly and register windows.
The possible values are the following:

    - `space`  
    Use a space character to draw the border.

    - `ascii`  
    Use ASCII characters &lsquo;+&rsquo;, &lsquo;-&rsquo; and &lsquo;|&rsquo; to draw the border.

    - `acs`  
    Use the Alternate Character Set to draw the border.  The border is
drawn using character line graphics if the terminal supports them.

- `set tui border-mode mode`   
- `set tui active-border-mode mode`  
Select the display attributes for the borders of the inactive windows
or the active window.  The mode can be one of the following:

    - `normal`  
    Use normal attributes to display the border.

    - `standout`  
    Use standout mode.

    - `reverse`  
    Use reverse video mode.

    - `half`  
    Use half bright mode.

    - `half-standout`  
    Use half bright and standout mode.

    - `bold`  
    Use extra bright or bold mode.

    - `bold-standout`  
    Use extra bright or bold and standout mode.
