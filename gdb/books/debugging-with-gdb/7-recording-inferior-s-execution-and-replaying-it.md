# 7 Recording Inferior’s Execution and Replaying It

On some platforms, GDB provides a special *process record and replay* target 
that can record a log of the process execution, and replay it later with both 
forward and reverse execution commands.

When this target is in use, if the execution log includes the record for the 
next instruction, GDB will debug in *replay mode*. In the replay mode, the 
inferior does not really execute code instructions. Instead, all the events that 
normally happen during code execution are taken from the execution log. While 
code is not really executed in replay mode, the values of registers (including 
the program counter register) and the memory of the inferior are still changed 
as they normally would. Their contents are taken from the execution log.

If the record for the next instruction is not in the execution log, GDB will 
debug in *record mode*. In this mode, the inferior executes normally, and GDB 
records the execution log for future replay.

The process record and replay target supports reverse execution (see 
[Reverse Execution](Reverse-Execution.html#Reverse-Execution)),
even if the platform on which the inferior runs does not. However, the reverse 
execution is limited in this case by the range of the instructions recorded in 
the execution log. In other words, reverse execution on platforms that don’t 
support it directly can only be done in the replay mode.

When debugging in the reverse direction, GDB will work in replay mode as long as 
the execution log includes the record for the previous instruction; otherwise, 
it will work in record mode, if the platform supports reverse execution, or stop 
if not.

For architecture environments that support process record and replay, GDB 
provides the following commands:

- `record method`  
   This command starts the process record and replay target. 

   The recording method can be specified as parameter. Without a parameter the 
   command uses the `full` recording method. The following recording methods are 
   available:

   - `full`  
      Full record/replay recording using GDB’s software record and replay 
      implementation. This method allows replaying and reverse execution.
   
   - `btrace format`  
      Hardware-supported instruction recording. 
   
      This method does not record data.  Further, the data is collected in a ring 
      buffer so old data will be overwritten when the buffer is full. It allows 
      limited reverse execution. Variables and registers are not available during 
      reverse execution. In remote debugging, recording continues on disconnect. 
      Recorded data can be inspected after reconnecting. The recording may be stopped 
      using `record stop`.  
   
      The recording format can be specified as parameter. Without a parameter the 
      command chooses the recording format. The following recording formats are 
      available:  
   
      - `bts`  
         Use the *Branch Trace Store* (BTS) recording format.  
         
         In this format, the processor stores a from/to record for each executed 
         branch in the btrace ring buffer.  
      
      - `pt`  
         Use the *Intel Processor Trace* recording format.  
      
         In this format, the processor stores the execution trace in a compressed form 
         that is afterwards decoded by GDB.  
      
         The trace can be recorded with very low overhead. The compressed trace format 
         also allows small trace buffers to already contain a big number of instructions 
         compared to BTS.  
      
         Decoding the recorded execution trace, on the other hand, is more expensive 
         than decoding BTS trace. This is mostly due to the increased number of 
         instructions to process. You should increase the buffer-size with care.
      
      Not all recording formats may be available on all processors.

   The process record and replay target can only debug a process that is already 
   running. Therefore, you need first to start the process with the `run` or `start` 
   commands, and then start the recording with the `record method` command.

   Displaced stepping (see [displaced stepping](Maintenance-Commands.html#Maintenance-Commands)) 
   will be automatically disabled when process record and replay target is started. 
   That’s because the process record and replay target doesn’t support displaced 
   stepping.

   If the inferior is in the non-stop mode (see [Non-Stop Mode](Non_002dStop-Mode.html#Non_002dStop-Mode)) 
   or in the asynchronous execution mode (see [Background Execution](Background-Execution.html#Background-Execution)), 
   not all recording methods are available. The `full` recording method does not 
   support these two modes.

- `record stop`  
   Stop the process record and replay target. 

   When process record and replay target stops, the entire execution log will be 
   deleted and the inferior will either be terminated, or will remain in its final 
   state.

   When you stop the process record and replay target in record mode (at the end 
   of the execution log), the inferior will be stopped at the next instruction that 
   would have been recorded. In other words, if you record for a while and then 
   stop recording, the inferior process will be left in the same state as if the 
   recording never happened.

   On the other hand, if the process record and replay target is stopped while 
   in replay mode (that is, not at the end of the execution log, but at some 
   earlier point), the inferior process will become “live” at that earlier state, 
   and it will then be possible to continue the usual “live” debugging of the 
   process from that state.

   When the inferior process exits, or GDB detaches from it, process record and 
   replay target will automatically stop itself.

- `record goto`  
   Go to a specific location in the execution log. 

   There are several ways to specify the location to go to:

   - `record goto begin`  
     `record goto start`  
      Go to the beginning of the execution log.

   - `record goto end`  
      Go to the end of the execution log.

   - `record goto n`  
      Go to instruction number n in the execution log.

- <code>record save <i>filename</i></code>  
   Save the execution log to a file `filename`. 

   Default filename is `gdb_record.process_id`, where `process_id` is the process 
   ID of the inferior.

   This command may not be available for all recording methods.

- <code>record restore <i>filename</i></code>  
   Restore the execution log from a file *filename*. 

   File must have been created with `record save`.

- <code>set record full insn-number-max <i>limit</i></code>  
  <code>set record full insn-number-max unlimited</code>  
   Set the limit of instructions to be recorded for the `full` recording method. 

   Default value is 200000.

   If *limit* is a positive number, then GDB will start deleting instructions from 
   the log once the number of the record instructions becomes greater than *limit*. 
   For every new recorded instruction, GDB will delete the earliest recorded 
   instruction to keep the number of recorded instructions at the limit. (Since 
   deleting recorded instructions loses information, GDB lets you control what 
   happens when the limit is reached, by means of the `stop-at-limit` option, 
   described below.)

   If *limit* is `unlimited` or zero, GDB will never delete recorded instructions 
   from the execution log. The number of recorded instructions is limited only by 
   the available memory.

- `show record full insn-number-max`  
   Show the limit of instructions to be recorded with the `full` recording method.

- `set record full stop-at-limit`  
   Control the behavior of the `full` recording method when the number of 
   recorded instructions reaches the limit. 

   If ON (the default), GDB will stop when the limit is reached for the first 
   time and ask you whether you want to stop the inferior or continue running it 
   and recording the execution log. If you decide to continue recording, each new 
   recorded instruction will cause the oldest one to be deleted.

   If this option is OFF, GDB will automatically delete the oldest record to 
   make room for each new one, without asking.

- `show record full stop-at-limit`  
   Show the current setting of `stop-at-limit`.

- `set record full memory-query`  
   Control the behavior when GDB is unable to record memory changes caused by an 
   instruction for the `full` recording method. 

   If ON, GDB will query whether to stop the inferior in that case.

   If this option is OFF (the default), GDB will automatically ignore the effect 
   of such instructions on memory. Later, when GDB replays this execution log, it 
   will mark the log of this instruction as not accessible, and it will not affect 
   the replay results.

- `show record full memory-query`  
   Show the current setting of `memory-query`.

   The `btrace` record target does not trace data. As a convenience, when 
   replaying, GDB reads read-only memory off the live program directly, assuming 
   that the addresses of the read-only areas don’t change. This for example makes 
   it possible to disassemble code while replaying, but not to print variables. In 
   some cases, being able to inspect variables might be useful. You can use the 
   following command for that:

- `set record btrace replay-memory-access`  
   Control the behavior of the `btrace` recording method when accessing memory 
   during replay. 

   If `read-only` (the default), GDB will only allow accesses to read-only 
   memory. If `read-write`, GDB will allow accesses to read-only and to read-write 
   memory. Beware that the accessed memory corresponds to the live target and not 
   necessarily to the current replay position.

- `set record btrace cpu identifier`  
   Set the processor to be used for enabling workarounds for processor errata 
   when decoding the trace.

   Processor errata are defects in processor operation, caused by its design or 
   manufacture. They can cause a trace not to match the specification. This, in 
   turn, may cause trace decode to fail. GDB can detect erroneous trace packets and 
   correct them, thus avoiding the decoding failures. These corrections are known 
   as *errata workarounds*, and are enabled based on the processor on which the 
   trace was recorded.

   By default, GDB attempts to detect the processor automatically, and apply the 
   necessary workarounds for it. However, you may need to specify the processor if 
   GDB does not yet support it. This command allows you to do that, and also allows 
   to disable the workarounds.

   The argument *identifier* identifies the CPU and is of the form: 
   `vendor:procesor identifier`. In addition, there are two special identifiers, 
   `none` and `auto` (default).

   The following vendor identifiers and corresponding processor identifiers are 
   currently supported:

   `intel` *family|model[|stepping]*

   On GNU/Linux systems, the processor *family*, *model*, and *stepping* can be 
   obtained from `/proc/cpuinfo`.

   If *identifier* is `auto`, enable errata workarounds for the processor on which 
   the trace was recorded. If *identifier* is `none`, errata workarounds are 
   disabled.

   For example, when using an old GDB on a new system, decode may fail because 
   GDB does not support the new processor. It often suffices to specify an older 
   processor that GDB supports.

   ```gdb
   (gdb) info record
   Active record target: record-btrace
   Recording format: Intel Processor Trace.
   Buffer size: 16kB.
   Failed to configure the Intel Processor Trace decoder: unknown cpu.
   (gdb) set record btrace cpu intel:6/158
   (gdb) info record
   Active record target: record-btrace
   Recording format: Intel Processor Trace.
   Buffer size: 16kB.
   Recorded 84872 instructions in 3189 functions (0 gaps) for thread 1 (...).
   ```
- `show record btrace replay-memory-access`  
   Show the current setting of `replay-memory-access`.

- `show record btrace cpu`  
   Show the processor to be used for enabling trace decode errata workarounds.

- `set record btrace bts buffer-size size`  
  `set record btrace bts buffer-size unlimited`  
   Set the requested ring buffer size for branch tracing in BTS format. Default 
   is 64KB.

   If *size* is a positive number, then GDB will try to allocate a buffer of at 
   least *size* bytes for each new thread that uses the btrace recording method and 
   the BTS format. The actually obtained buffer size may differ from the requested 
   size. Use the `info record` command to see the actual buffer size for each 
   thread that uses the btrace recording method and the BTS format.

   If *limit* is `unlimited` or zero, GDB will try to allocate a buffer of 4MB.

   Bigger buffers mean longer traces. On the other hand, GDB will also need 
   longer to process the branch trace data before it can be used.

- `show record btrace bts buffer-size size`  
   Show the current setting of the requested ring buffer size for branch tracing 
   in BTS format.

- <code>set record btrace pt buffer-size <i>size</i></code>  
  <code>set record btrace pt buffer-size unlimited</code>  
   Set the requested ring buffer size for branch tracing in Intel Processor 
   Trace format. Default is 16KB.

   If *size* is a positive number, then GDB will try to allocate a buffer of at 
   least *size* bytes for each new thread that uses the btrace recording method and 
   the Intel Processor Trace format. The actually obtained buffer size may differ 
   from the requested *size*. Use the `info record` command to see the actual buffer 
   size for each thread.

   If *limit* is `unlimited` or zero, GDB will try to allocate a buffer of 4MB.
   
   Bigger buffers mean longer traces. On the other hand, GDB will also need 
   longer to process the branch trace data before it can be used.

- <code>show record btrace pt buffer-size <i>size</i></code>  
   Show the current setting of the requested ring buffer size for branch tracing 
   in Intel Processor Trace format.

- `info record`  
   Show various statistics about the recording depending on the recording method:

   - `full`  
      For the `full` recording method, it shows the state of process record and its 
      in-memory execution log buffer, including:  

      -  Whether in record mode or replay mode.
      -  Lowest recorded instruction number (counting from when the current execution 
         log started recording instructions).
      -  Highest recorded instruction number.
      -  Current instruction about to be replayed (if in replay mode).
      -  Number of instructions contained in the execution log.
      -  Maximum number of instructions that may be contained in the execution log.

   - `btrace`  
      For the `btrace` recording method, it shows:

      -  Recording format.
      -  Number of instructions that have been recorded.
      -  Number of blocks of sequential control-flow formed by the recorded instructions.
      -  Whether in record mode or replay mode.

      For the `bts` recording format, it also shows:  

      -  Size of the perf ring buffer.

      For the `pt` recording format, it also shows:  

      -  Size of the perf ring buffer.

- `record delete`  
   When record target runs in replay mode (“in the past”), delete the subsequent 
   execution log and begin to record a new execution log starting from the current 
   address. 

   This means you will abandon the previously recorded “future” and begin 
   recording a new “future”.

- `record instruction-history`  
   Disassembles instructions from the recorded execution log. 

   By default, ten instructions are disassembled. This can be changed using the 
   ‘`set record instruction-history-size`’ command. Instructions are printed in 
   execution order.

   It can also print mixed source+disassembly if you specify the the `/m` or 
   `/s` modifier, and print the raw instructions in hex as well as in symbolic form 
   by specifying the `/r` modifier.

   The current position marker is printed for the instruction at the current 
   program counter value. This instruction can appear multiple times in the trace 
   and the current position marker will be printed every time. To omit the current 
   position marker, specify the `/p` modifier.

   To better align the printed instructions when the trace contains instructions 
   from more than one function, the function name may be omitted by specifying the 
   `/f` modifier.

   Speculatively executed instructions are prefixed with ‘?’. This feature is 
   not available for all recording formats.

   There are several ways to specify what part of the execution log to 
   disassemble:

   - `record instruction-history insn`  
      Disassembles ten instructions starting from instruction number insn.

   - `record instruction-history insn, +/-n`  
      Disassembles n instructions around instruction number insn. If n is preceded 
      with `+`, disassembles n instructions after instruction number insn. If n is 
      preceded with `-`, disassembles n instructions before instruction number insn.

   - `record instruction-history`  
      Disassembles ten more instructions after the last disassembly.

   - `record instruction-history -`  
      Disassembles ten more instructions before the last disassembly.

   - `record instruction-history begin, end`  
      Disassembles instructions beginning with instruction number begin until 
      instruction number end. The instruction number end is included.

   This command may not be available for all recording methods.


-  <code>set record instruction-history-size <i>size</i></code>  
   <code>set record instruction-history-size unlimited</code>  
   Define how many instructions to disassemble in the `record instruction-history` command. 

   The default value is 10. A size of `unlimited` means unlimited instructions.

- `show record instruction-history-size`  
   Show how many instructions to disassemble in the `record instruction-history` command.

- `record function-call-history`  
   Prints the execution history at function granularity. It prints one line for 
   each sequence of instructions that belong to the same function giving the name 
   of that function, the source lines for this instruction sequence (if the `/l` 
   modifier is specified), and the instructions numbers that form the sequence (if 
   the `/i` modifier is specified). The function names are indented to reflect the 
   call stack depth if the `/c` modifier is specified. The `/l`, `/i`, and `/c` 
   modifiers can be given together.

   ```gdb
   (gdb) list 1, 10
   1   void foo (void)
   2   {
   3   }
   4
   5   void bar (void)
   6   {
   7     ...
   8     foo ();
   9     ...
   10  }
   (gdb) record function-call-history /ilc
   1  bar     inst 1,4     at foo.c:6,8
   2    foo   inst 5,10    at foo.c:2,3
   3  bar     inst 11,13   at foo.c:9,10
   ```

   By default, ten lines are printed. This can be changed using the ‘`set record 
   function-call-history-size`’ command. Functions are printed in execution order. 
   There are several ways to specify what to print:

   - `record function-call-history func`  
      Prints ten functions starting from function number *func*.

   - `record function-call-history func, +/-n`  
      Prints n functions around function number *func*. If *n* is preceded with `+`, 
      prints n functions after function number *func*. If *n* is preceded with `-`, prints 
      *n* functions before function number *func*.

   - `record function-call-history`  
      Prints ten more functions after the last ten-line print.

   - `record function-call-history -`  
      Prints ten more functions before the last ten-line print.

   - `record function-call-history begin, end`  
      Prints functions beginning with function number *begin* until function number 
      *end*. The function number *end* is included.

   This command may not be available for all recording methods.

- `set record function-call-history-size size`  
  `set record function-call-history-size unlimited`  
   Define how many lines to print in the `record function-call-history` command. 

   The default value is 10. A size of `unlimited` means unlimited lines.

- `show record function-call-history-size`  

   Show how many lines to print in the `record function-call-history` command.

------------------------------------------------------------------------

Next: [Stack](Stack.html#Stack), Previous: [Reverse Execution](Reverse-Execution.html#Reverse-Execution), Up: [Top](index.html#Top)   \[[Contents](index.html#SEC_Contents "Table of contents")\]\[[Index](Concept-Index.html#Concept-Index "Index")\]


