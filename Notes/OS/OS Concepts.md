## User mode
A user-mode program generally cannot:

- Directly access hardware
- Modify page tables
- Read another process’s memory
- Disable interrupts
- Execute privileged CPU instructions
- Access kernel memory
- Perform raw disk or network operations directly
## Kernel mode
The operating-system kernel and many device drivers execute in kernel mode. Kernel code has access to privileged instructions and protected resources.

It can:

- Configure memory mappings
- Schedule threads
- Handle interrupts
- Communicate with hardware
- Manage filesystems
- Process network packets
- Control storage devices
- Access any process’s address space, subject to OS logic
The CPU can enter kernel mode for several reasons:

- **System call:** an application intentionally requests an OS service.
- **Exception:** the current instruction causes an event such as a page fault or divide-by-zero.
- **Hardware interrupt:** a device signals completion or requires attention.

| User mode                              | Kernel mode                                    |
| -------------------------------------- | ---------------------------------------------- |
| Restricted privileges                  | Full hardware privileges                       |
| Isolated process address space         | Access to kernel and system resources          |
| Hardware accessed through system calls | Hardware accessed through drivers              |
| Crash usually affects one process      | Bug can crash or corrupt the system            |
| Easier and safer to develop            | Requires strict validation and synchronization |
User mode is the restricted execution environment used by applications, while kernel mode is the privileged environment used by the OS and drivers. Applications enter the kernel through controlled mechanisms such as system calls, exceptions, and interrupts. The separation provides security and fault isolation: an application failure normally affects one process, while a kernel failure can affect the entire machine.

## System calls

Applications cannot directly access protected hardware or kernel data, so they request services through system calls.

Common Linux examples include:

```
read(fd, buf, size);
write(fd, buf, size);
open(path, flags);
mmap(addr, length, protection, flags, fd, offset);
fork();
execve(...);
```

Conceptually, a system call works like this:

```
Application in user mode
        │
        │ syscall instruction
        ▼
Kernel mode, same thread
        │
        │ kernel performs operation
        ▼
Application resumes in user mode
```

The steps are approximately:

1. The application places the syscall number and arguments in registers.
2. It executes a special CPU instruction such as `syscall`.
3. The CPU changes privilege level and enters a predefined kernel entry point.
4. The kernel saves enough user state to return later.
5. It validates pointers, permissions, file descriptors, and arguments.
6. It performs or starts the operation.
7. It places the result or error in a register.
8. It restores user state and returns to user mode.

A normal function call only changes the instruction pointer and stack according to the calling convention. A system call crosses a protection boundary.
## Context switches

A context switch occurs when the scheduler changes which thread is running on a CPU:

```
Thread A → Thread B
```

The operating system must save Thread A’s execution state and restore Thread B’s state. That state can include:

- Program counter
- Stack pointer
- General-purpose registers
- CPU flags
- Scheduling information
- Floating-point/SIMD state when necessary
- Address-space information when switching processes

A context switch can happen when:

- A thread blocks on I/O
- A thread waits on a mutex or condition variable
- Its time slice expires
- A higher-priority thread becomes runnable
- The thread voluntarily yields
- An interrupt causes the scheduler to reconsider the running thread

## Blocking system-call example

Suppose Thread A calls `read()` and the requested data is not available:

```
Thread A, user mode
    │ read()
    ▼
Thread A, kernel mode
    │ data unavailable
    │ A becomes blocked
    ▼
Context switch
    ▼
Thread B runs
```

Later, the storage device completes the request:

```
Device interrupt
    │
Kernel marks Thread A runnable
    │
Scheduler eventually selects A
    ▼
Context switch to Thread A
    │
read() returns
    ▼
Thread A resumes in user mode
```
This system call caused context switches because it blocked. A nonblocking or immediately satisfied `read()` might not.

## Thread versus process context switches

A switch between threads of the same process is generally cheaper because they share an address space.

A switch between different processes may require changing the active virtual-memory mappings. This can create additional costs involving:

- Page-table state
- TLB behavior
- CPU caches
- Branch predictors
- Kernel bookkeeping

Modern CPUs and operating systems reduce some of these costs, but cache disruption often matters more than merely saving registers.

A system call is a controlled transition from user mode into kernel mode to request an OS service. A context switch is a scheduler operation that replaces the currently running thread with another thread. Every system call crosses the user-kernel privilege boundary, but it causes a context switch only if the caller blocks, is preempted, or the scheduler chooses another runnable thread.

| Event                          | Mode switch?       | Context switch? |
| ------------------------------ | ------------------ | --------------- |
| Immediate `getpid()`           | Yes                | Usually no      |
| Cached `read()`                | Yes                | Usually no      |
| Blocking disk `read()`         | Yes                | Usually yes     |
| Time slice expires             | Kernel is involved | Yes             |
| Interrupt, same thread resumes | Yes temporarily    | No              |
| Mutex wait under contention    | Yes                | Usually yes     |
# Virtual memory, page tables, TLBs, and page faults
```
Virtual address generated by CPU
             │
             ▼
        Check the TLB
        ┌────┴────┐
      hit        miss
       │           │
       │      Walk page tables
       │           │
       └─────┬─────┘
             ▼
     Physical address found?
        ┌────┴────┐
       yes         no
        │          │
   Access RAM   Page fault
```
## Pages and frames

Memory is divided into fixed-size units:

- A **virtual page** is a unit of virtual address space.
- A **physical frame** is a corresponding unit of RAM.

Common page sizes include 4 KiB, 2 MiB, and 1 GiB, depending on architecture and configuration.

For a 4 KiB page:

```
Virtual address = virtual page number + 12-bit offset
```

The offset remains unchanged during translation:

```
Virtual page 42, offset 100
        ↓
Physical frame 900, offset 100
```

Larger pages reduce translation overhead but can waste memory and increase allocation or fragmentation challenges.

## Page tables

Page tables are data structures maintained by the operating system and used by the CPU’s memory-management unit, or MMU.

A page-table entry can contain:

- Physical frame number
- Present/valid bit
- Read/write permission
- User/kernel permission
- Executable or non-executable permission
- Accessed bit
- Dirty bit
- Cache-control information

Conceptually:

```
Virtual page number → page-table entry → physical frame number
```

Modern address spaces are enormous, so a process does not use one giant flat array. Architectures normally use multilevel page tables:

```
Virtual address
   │
   ├── level-1 index
   ├── level-2 index
   ├── level-3 index
   ├── level-4 index
   └── page offset
```

Only the portions needed by the process must be allocated.

### Why permissions live in page tables

Suppose a process writes to a read-only page. The page-table entry denies that operation, and the CPU raises a fault. This lets the OS implement:

- Read-only code
- Non-executable data
- Kernel-memory protection
- Copy-on-write
- Guard pages around stacks

## TLB

The Translation Lookaside Buffer is a small, fast CPU cache of recent virtual-to-physical translations.

Without it, nearly every memory access could require several additional memory accesses to walk multilevel page tables.

```
Virtual page 42 → physical frame 900
```

If that translation is already in the TLB, the CPU can use it quickly.

### TLB hit

The translation and permissions are cached:

```
Virtual address → TLB → physical address → data
```

This is the fast path.

### TLB miss

The translation is not cached. The CPU, sometimes with OS assistance depending on the architecture, walks the page tables.

If a valid page-table entry exists:

1. The translation is loaded into the TLB.
2. The original instruction is retried.
3. Execution continues.

A TLB miss is not the same as a page fault. The page can already be resident in RAM even though its translation is absent from the TLB.

Virtual memory gives each process an isolated address space divided into pages. Page tables map virtual pages to physical frames and store protection information. The TLB caches recent translations to avoid expensive page-table walks. A TLB miss means the translation is not cached, while a page fault means the current mapping cannot satisfy the access. The kernel may resolve a valid fault through demand paging or copy-on-write, or reject an invalid access with an error such as `SIGSEGV`.

# Page Cache & memory mapped IO
### Read path

For:

```
read(fd, buffer, 4096);
```

the kernel:

1. Checks whether the requested file pages are in the page cache.
2. On a cache hit, copies data from the page cache into the user buffer.
3. On a cache miss, starts storage I/O.
4. Places the resulting data in the page cache.
5. Copies it into the user buffer.
### Write path

For:

```
write(fd, buffer, 4096);
```

the kernel commonly:

1. Copies data from the user buffer into page-cache pages.
2. Marks those pages dirty.
3. Returns before the data is necessarily on persistent storage.
4. Writes dirty pages back later through background writeback.
## Memory-mapped I/O

`mmap()` maps a file region into a process’s virtual address space:

```
void *p = mmap(NULL, length, PROT_READ | PROT_WRITE,
               MAP_SHARED, fd, 0);
```

The application can then access the file using normal loads and stores:

```
char value = ((char *)p)[100];
((char *)p)[200] = 'X';
```

The mapping connects virtual pages to the file’s page-cache pages:

```
Process virtual address
          │
       page table
          │
          ▼
Physical page in page cache
          │
          ▼
Corresponding file offset
```
## Important interview pitfalls

- `write()` does not necessarily mean durable.
- `mmap()` does not necessarily mean data is already in RAM.
- A page fault from a mapping may trigger storage I/O.
- `msync()` concerns mapped pages; durability requirements can involve additional filesystem metadata.
- Accessing beyond the valid portion of a mapped file can generate `SIGBUS`.
- If another thread or process truncates a mapped file, existing accesses may fault.
- `MAP_PRIVATE` modifications do not update the file.
- Mapping does not eliminate page-table, cache, or fault costs.
- `mmap()` is often described as “zero-copy,” but that only refers to avoiding the extra page-cache-to-user-buffer copy; the device still transfers data into RAM.

The page cache is the kernel’s cache of file-backed data in RAM. Buffered `read()` and `write()` normally move data between user buffers and that cache, while `mmap()` maps the cached pages directly into a process’s virtual address space. Missing mapped pages are loaded through page faults, and modified shared pages become dirty and are written back later. Neither `write()` nor a memory store alone guarantees persistence; explicit synchronization such as `fsync()` or `msync()` is needed according to the required durability semantics.

# Interrupts, Deferred Work, & DMA
```
CPU configures device and DMA buffers
              │
              ▼
Device transfers data directly to/from RAM
              │
              ▼
Device raises an interrupt
              │
              ▼
Small interrupt handler acknowledges completion
              │
              ▼
Deferred work performs heavier processing
              │
              ▼
Waiting application is awakened
```
```
1. Application calls read()
2. Kernel determines data is not cached
3. Block layer creates an I/O request
4. Driver builds DMA descriptors
5. Driver places request in hardware submission queue
6. Device retrieves descriptors
7. Controller transfers data into RAM using DMA
8. Device posts a completion
9. Device raises an interrupt
10. Interrupt handler acknowledges it
11. Deferred processing completes the block request
12. Kernel marks the data available
13. Waiting application becomes runnable
14. Scheduler eventually runs the application
15. read() returns
```
An interrupt lets a device asynchronously notify the CPU of an event such as I/O completion. The immediate interrupt handler should do minimal work because it runs in a restricted context, so heavier processing is moved to deferred mechanisms such as softirqs, workqueues, or threaded interrupts. DMA allows the device to transfer data directly between itself and RAM after the driver configures buffers and descriptors. On completion, the device posts status and may raise an interrupt. Correct implementations must handle DMA mapping, cache coherency, memory ordering, buffer lifetime, interrupt affinity, and the throughput-versus-latency tradeoff of batching or interrupt coalescing.
# Kernel threads & scheduling
## Kernel threads

A kernel thread performs operating-system work that should run asynchronously or may need to block.

Linux examples include work for:

- Dirty-page writeback
- Memory reclamation
- Deferred interrupt processing
## Task states

A schedulable task can be in states such as:

- **Running:** currently executing on a CPU
- **Runnable:** ready to execute but waiting in a run queue
- **Sleeping:** waiting for an event or resource
- **Stopped:** deliberately suspended
- **Terminated:** finished execution

Only one thread runs on a logical CPU at a time:

```
CPU 0: Thread A running
Run queue: Thread B, Kernel worker K, Thread C
```

On a multicore system, several threads can run simultaneously—one per logical CPU.
## Scheduling

The scheduler selects a runnable thread for each CPU.

A scheduling decision can happen when:

- The current thread blocks
- Its scheduling time expires
- A higher-priority task becomes runnable
- A task voluntarily yields
- An interrupt wakes another task
- The current task exits
- CPU-balancing logic migrates work

The decision may produce a context switch:

```
Save Thread A state
→ choose Thread B
→ restore Thread B state
→ run Thread B
```
## Run queues

The kernel tracks runnable threads in scheduling data structures, commonly organized per CPU:

```
CPU 0 run queue          CPU 1 run queue
- Thread A               - Thread C
- Worker K1              - Thread D
- Thread B               - Worker K2
```
## Common interview pitfalls

- Entering kernel mode does not make a user thread a kernel thread.
- Waking a thread does not guarantee immediate execution.
- Runnable is different from running.
- An interrupt is not itself a kernel thread.
- Kernel threads can usually sleep; interrupt handlers generally cannot.
- More worker threads do not always improve throughput—contention and context-switch overhead can make performance worse.
- A context switch can disrupt caches even when register-save overhead is small.
A kernel thread is an independently schedulable task that runs kernel code, generally without a user address space. It is useful for background or deferred work that may block, such as memory reclamation, writeback, or I/O completion processing. The scheduler chooses among runnable user and kernel threads using priorities, scheduling policies, CPU affinity, and load balancing. A kernel thread can normally sleep because it runs in process context, unlike a hard-interrupt handler. Scheduling delays, CPU migration, contention, and priority inversion can all materially affect storage tail latency.
# Preemption & Interrupt COntext
## Common interview traps

- Kernel mode does not automatically mean interrupt context.
- A user thread executing a system call is still in thread context.
- Disabling preemption does not disable hardware interrupts.
- Disabling interrupts locally does not stop other CPUs.
- Spinlocks must not be held while sleeping.
- Softirq context is deferred but still usually non-sleepable.
- A workqueue runs in thread context and may sleep.
- An interrupt does not necessarily produce a context switch.

Preemption allows the scheduler to pause a runnable task and execute another, including while a task is in preemptible kernel code. It must be disabled in short critical regions where switching tasks would violate invariants. Interrupt context is different: it is asynchronous kernel execution caused by hardware and is not an ordinary schedulable thread, so it cannot sleep or take blocking locks. Hard-interrupt handlers should acknowledge the event, capture minimal state, and defer heavier work to softirqs or thread-based mechanisms. An interrupt may wake a higher-priority thread and thereby lead to preemption, but the interrupt and the context switch are separate events.
# Kernel Locks
## Choosing a kernel lock

|Situation|Typical mechanism|
|---|---|
|Very short section used by interrupt handler|Spinlock|
|Thread-only section that may wait|Mutex|
|Counted pool of resources|Semaphore|
|Many readers, infrequent writers|RW lock/RW semaphore or RCU|
|Simple counter/state flag|Atomic operation|
|CPU-local statistics|Per-CPU data|
|Read-mostly object lookup|RCU, when appropriate|

## Common interview traps

- A spinlock does not make slow work acceptable.
- Interrupts disabled does not protect against other CPUs.
- A spinlock alone may not protect against same-CPU interrupt reentry.
- A mutex cannot normally be acquired in interrupt context.
- Sleeping while holding a mutex is allowed but may be poor design.
- `volatile` is not synchronization.
- Reference counting protects lifetime, not concurrent field modification.
- Never call an unknown callback while holding a low-level lock unless its behavior is strictly defined.
- Lock acquisition must respect a consistent global order.

Spinlocks are used for short critical sections where the current context cannot sleep, including data shared with interrupt handlers. Sleeping while holding one is illegal because spinlock acquisition establishes an atomic context: other CPUs may spin indefinitely, interrupt context cannot be rescheduled, and the owner might not run to release the lock. Mutexes, by contrast, are sleepable locks used in thread context; a waiter can be descheduled. Sleeping while holding a mutex is possible but should be minimized because it expands lock hold time and can cause convoys, priority inversion, or deadlock. The correct choice depends on access context, expected hold time, and whether protected operations can block.
# Reference counting and object lifetimes
## Practical review checklist

For every reference acquisition, ask:

- Who owns the new reference?
- What event releases it?
- What happens on every error path?
- What happens if work is canceled?
- What synchronization makes acquisition safe?
- Can the counter already be zero?
- Can an asynchronous callback outlive the initiator?
- Are object fields separately synchronized?
- Can ownership form a cycle?
- What exactly happens during final destruction?

Reference counting protects object lifetime by keeping an object alive while at least one owning reference exists. Every successful `get` must have exactly one matching `put`, and the final `put` performs destruction. Atomic increments alone are insufficient because acquiring a reference from a shared lookup can race with removal and freeing; the reference must be obtained while a lock, RCU, weak-reference protocol, or another lifetime guarantee protects the pointer. Reference counts protect existence, not concurrent access to the object’s fields. The design must also handle asynchronous work, cancellation races, zero-count resurrection, overflow, error paths, and ownership cycles.
# Core files
## Examining a core with GDB

Typical usage:

```
gdb /path/to/executable /path/to/core
```

Useful commands:

```
info threads
thread apply all bt
thread apply all bt full
frame 3
info locals
info args
info registers
x/16gx address
disassemble /m function_name
```

A disciplined first pass:

1. Identify the fatal signal and faulting instruction.
2. Inspect the crashing thread.
3. Capture all thread stacks.
4. Examine registers and pointer values.
5. Check whether other threads hold relevant locks.
6. Inspect object state and ownership.
7. Correlate findings with logs and source.
8. Determine whether the crash site is cause or consequence.

## Example: null-pointer failure

Suppose:

```
request->queue->tail = entry;
```

The crash dump shows:

```
request = valid
request->queue = NULL
```

The next questions are not merely “why did it crash?” but:

- Was `queue` never initialized?
- Was it cleared concurrently?
- Was `request` partially constructed?
- Did an error path violate an invariant?
- Is this really a stale or corrupted request?
- Which thread last modified it?

## Example: use-after-free

Possible clues include:

- Pointer refers to allocator poison data.
- Object fields contain unrelated values.
- Reference count is invalid.
- Vtable/function pointer is corrupted.
- Memory now belongs to a different object.
- One thread is destroying the object while another uses it.

A core captures only the final state. Tools such as AddressSanitizer, allocator quarantine, watchpoints, tracing, or reference-lifetime logging may be needed to find the earlier invalid release.

A stack trace reconstructs the active call chain from saved stack and register state. Symbols map machine addresses to functions, source lines, types, and variables, and they must exactly match the crashed build. A core file captures the state of a failed user process, while a kernel crash dump such as a Linux `vmcore` captures kernel and broader system memory after a panic. I would begin by confirming the build and symbols, identify the faulting instruction, inspect all thread or CPU stacks, registers, locks, and relevant objects, then correlate that snapshot with logs. I would also remember that the crash site may only be where earlier memory corruption or a lifetime bug became visible.

# Debug IO hang
I would first determine whether the process is truly waiting for I/O rather than an application lock. I’d inspect all thread stacks, wait channels, process state, and current system calls, using GDB only when appropriate. Next, I’d identify the exact file descriptor and map it through the filesystem, block device, multipath layer, transport, and target. At each boundary I’d compare submitted and completed requests, queue depth, latency, timeouts, and errors. If the block layer has not dispatched the request, I’d focus on the filesystem or host queueing. If it was dispatched but never completed, I’d investigate the driver, fabric, and target. If the target received but did not complete it, I’d examine target QoS, controller pressure, failover, and media latency. If the target completed it but the host did not observe completion, I’d examine the fabric, driver, interrupts, and completion queues. I’d correlate all evidence by timestamp and compare the affected path with a healthy one before forming a root-cause hypothesis.