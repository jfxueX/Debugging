Edit ~/.gdbinit

```gdb
# configure TUI
tui enable
layout asm
layout regs
set tui border-kind space
set tui border-mode bold
tabset 4
winheight regs -11
winheight asm +8
winheight cmd +3
focus cmd
refresh
```
