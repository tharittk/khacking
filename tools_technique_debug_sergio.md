# Tools and Techniques to Debug an Embedded Linux System
[https://www.linuxfoundation.org/webinars/tools-and-techniques-to-debug-an-embedded-linux-system]
- understand the problem (have to be able to read and understand the error)
- reproduce the problem
- identify the root cause
- types: crash, lockup/hang, logic/implementation, resource leakage, lack of performance
- tools: our brain, post mortem (log, memory dump) - extracted from the device, tracing/profiling (add print to the code), interactive debugging (gdb), debugging frameworks (Valgrind)

### Post Mortem Analysis
- Logs: from kernel, user space
- Memory dump
- example: insert USB stick and kernel crashes
- "unable to handle kernel NULL pointer dereference at virtual address 0000 0000"
- PC is at storage_probe + 0x60/0x270 <- location of the crash i.e. function
- and then we have the back trace (from start -> ... -> storage_probe)
- you can use GDB (needed vmlinux <the debug symbols so you can convert the addr -> symbol>)
- the other one is `$ arm-linux addr2line -f -e vmlinux <addr like 0xc06c5eb0>`
- now you get, storage_probe, and file : line no.

- example: user space, core dump analysis
- `$ fping -f 3 192.168.0.1` <- application that is crashing
- `ulimit -c unlimited` <- used to limit resource like threads, file opened, etc.
- core dumped
- `$ arm-linux-gdb src/fping -c core (application and core)`
- then it will show the line of code it crashes
- tui mode in gdb will also give the view of the source code
- print option, print *option, print option->some_ptr
- this is not the interactive
- compiler optimization may reduce the binary size and therefore the info for debugging

### Tracing
- print ~ add static tracing point
- dynamic tracing in linux -> add trace point at run-time
- kernel hacking -> enable tracing (f trace)
- `$ mount -t tracefs tracefs /sys/kernel/tracing`
- `$ echo function_graph > current_tracer`
- `$ trace-cmd record -p function_graph -F echo l ...` - see Steven's presentation
- You don't have to recompile the kernel to trace arbitary points (assuming you have CONFIG_FTRACE already)
- ftrace instrumented code (~ instead of jumping to original function pointer, you jump to instrumented function pointer - which is created during runtime -- invoked by user)
- the cousin is the `strace`: for user application, `ltrace` is for library call

### Interactive Debugging (gdb)
- for kernel space, you need client-server
- server: device (running software), CONFIG_KGDB=y, CONFIG_KGDB_SERIAL=y
- client: your host (binary, source)
- agent-proxy (to console or serial)
- put the kernel (device) in the debugged mode
- then `$ arm-linux-gdb vmlinux -tui`, connect to device `target remote localhost:5551` <- your gdb port of the device
- (gdb) add break point to some functions
- (gdb) c
- now you run something that contains breakpointed function on the device
- it will stop at that point (device stopped)
- then in the host gdb console, you can do interactively (backtrace etc.)
- `$ gdbserver localhost:1234 tree var` <- gdb, please open up the port 1234 to debug this application tree
- then in host `$ arm-linux-gdb tree -tui` <- tree is for debug symbol (check with `file tree`)
- (gdb) target remote 192.168.0.2:1234 <- ethernet connection

### Debugging framework (Valgrind)
- Let's say the kernel hangs from `$ cat /proc/uptime`
- You will see kernel oops (if you CONFIG_ENABLE_HANG sth).
- `$ valgrind --leak-check=full <your binary>`
