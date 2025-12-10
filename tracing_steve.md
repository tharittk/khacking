# Learning the Linux Kernel with Tracing (Steven Rosetedt)
[https://www.youtube.com/watch?v=JRyrhsx-L5Y]

- [hardware] [kernel including BIOS] [library] [application]
- kernel knows to parse ELF file, put them in where they belong in the memory, and jump to the location it needs to.
- printf -> compile opt -> puts (you see this is objdump -d as there is no need for parsing, it is a static string "hello")
- printf("hello world %p \n", &main) - if you look at the assembly, main is at says 0x135
- but you see 0x55513135 <- kernel security feature that add the offset (randomized) to your elf file when loaded
- address space is not only the memory ! it may include your other devices.
- cr3 register stores page table root entry
- vaddr is broken into multi-level pagetable (PGD -> PUD -> PMD-> PTE) many hops
- 0x0000 0000 -> 0x7FFF FFFF is [user] after that to 0xFFFF FFFF is [kernel] space
- app1, app2 shares libc.so (mapped to each app virtual address) <-> kernel is the service provider through system call interface
- strace: system call tracer (using ptrace) -> you hook it to application and it becomes the parent
- `gdb` is done via ptrace, actually. People try to move to `perf`
- ftrace: official, busybox
- /sys/kernel/tracing <- psuedo directory if ftrace is enabled.
- when you read, it is pausing the tracing. If you want producer-consumer, use trace pipe
- trace-cmd: trace-cmd start -p function; trace-cmd show; (take the mounting for etc. from ftrace)
- ftrace does not need user space (while strace does)
- `$ trace-cmd record -p function_graph --max-graph-depth 1 -e syscalls -F ./hello`
- `$ trace-cmd report`
- You see: Program does not load everything in ELF file at once. It does dynamically via page_fault
- `$ trace-cmd record -p function_graph --max-graph-depth 2 -e syscalls -F ./hello`
- to graph one function only (dig deeper)
- `$ trace-cmd record -p function_graph -g __x64_sys_write -F ./hello`
- You can alslo trace when something gets schedule in / out 
- Too Much Info: people built the way to visualize it: KernelShark
