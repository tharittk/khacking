# Introduction to Memory Management Linux
[https://www.youtube.com/watch?v=7aONIVSXiJ8&list=PLbzoR-pLrL6pRFP6SOywVJWdEHlmQE51q]

- single address space: memory and peripheral share the same space (they map to a different part)
- limitation: portable C program expects flat memory (have to manage and not let 2 programs overlap) -> need specialized knowledge of the platform
- Then people introduced Virtual Memory
- One process's RAM is inaccessible to other processes.
- Kernel RAM is invisible to user space process
- You can do shared memory by having to process VA maps to same PA
- PA: DMA, peripheral
- VA: other stuffs (load, store)
- The Mapping is done in hardware (MMU) - sit between CPU core <-- and --> RAM
- MMU also handles exception generating and page fault
- TLB
- Page Fault = invalid virtual address access i.e., not in the TLB
- not in TLB = not mapped yet, swapped out
- Kernel Virtual Address: In VA, KVA is at high (0xFFFF FFFF) (4GB) down to CONFIG PAGE_OFFSET
- default at 0xC000 0000 (3GB)
- 3 kinds of VA: Kernel Logical Address, Kernel Virtual Address, User Virtual Address
### Kernel Logical Address
- `kmalloc()` = give kernel logical address. "Fixed offset" from the physical address
- Virt: 0xc000 0000 -> phys 0x0000 0000. Easy mapping (add/subtract offset)
- kernel stack also sits here. 
- this logical memory cannot be swapped out
- they are all physically contiguous -> it is used by DMA
- KLA (High in VA) maps to Low address in PA (bottom)
- there is macro _pa(x) and _va(x) to convert

### Kernel Virtual Address
- "non-contiguous in physical" (but contiguous in virtual - so you can do linear ptr access)
- so you cannot rely on it for DMA
- usually it is too large of memory to find 1 contiguous physical chunks
- `vmalloc()`
- also above 3GB mark in VA

### User Virtual Address
- below the 3 GB mark (in 32-bit)
- non-contiguous physically, cannot DMA
- swappable
- every process has its own mapping `struct_mm` in `task_struct` which is changed when you do context switch

### MMU (and TLB)
- MMU works on unit of a page
- TLB ~ 16 entries in embedded stuff (a lot of churns). You get lots of "cache missed" early on after context switching
- Request (CPU-> MMU)
- MMU checks if VA in TLB
- Y: easy translate quickly
- N: MMU walks the page table (multi-level bit matching stuff to find PTE/Page Frame) in RAM, find the address, update TLB

### Shared Memory for IPC
- 2 VAs (from different processes) -> same physical frame/address

### Lazy Allocation
- the kernel will not allocate pages requested immediately (your malloc returns ok but nothing happens physically), if you do malloc() but never uses it, nothing get allocate
- when you allocate, it creates the request in the table
- when that address is touched, page fault, in the hanlder, it will allocate the page physically

`struct mm` and `struct vm_area`

### Swapping
- Your RAM is full, content of the page frame (physical) -> storage, now that VA entry can be replaced
- now if you ask for that page back (use the same virtual address) but you actually access different physical page frame (completely different backing) - no consistent physical backing

- `mmap()` (MAP_ANYMOUS, MAP_SHARED <- for IPC ) reduce kernel buffer -> user space copy
- with `mmap()`, the kernel bring the request data to RAM and modify the user's Page Table so that there is a pointer point to that physical address the kernel just brought data to.
- `read()` Disk -> Kernel Buffer -> User-buffer
- `mmap()` Disk -> Physical RAM. Physical RAM <- User pointer
- actually, malloc() maybe implemented through brk or mmap
- if stack is beyond the limit, it will trigger the page fault
