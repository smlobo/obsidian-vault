## `ps -eo pid,state,wchan:32,comm,args`

Displays every process with fields useful for identifying blocked tasks:

```
ps -eo pid,state,wchan:32,comm,args
```

Fields:

|Field|Meaning|
|---|---|
|`pid`|Process ID|
|`state`|Current process state|
|`wchan:32`|Kernel function where the process is sleeping|
|`comm`|Executable name|
|`args`|Complete command line|

Important states include:

|State|Meaning|
|---|---|
|`R`|Running or runnable|
|`S`|Interruptible sleep, often waiting normally|
|`D`|Uninterruptible sleep, frequently waiting for I/O|
|`T`|Stopped or being traced|
|`Z`|Zombie|
|`I`|Idle kernel thread|

Example:

```
PID   S WCHAN                            COMMAND  COMMAND
1842  D io_schedule                      database /usr/bin/database
```

A persistent `D` state suggests the process is blocked inside the kernel, often waiting for storage, NFS, or a device driver.

`wchan` is meaningful primarily when a task is sleeping. It may show `0` if symbols or permission are unavailable.

## `cat /proc/<pid>/stack`

Displays the process’s current kernel stack:

```
sudo cat /proc/1842/stack
```

Example:

```
[<0>] io_schedule+0x...
[<0>] wait_on_page_bit_common+0x...
[<0>] filemap_fault+0x...
```

This shows how the task reached its current sleeping point and provides more context than `wchan`.

For a multithreaded process, inspect each individual thread:

```
ls /proc/1842/task
sudo cat /proc/1842/task/<tid>/stack
```

Access may require root privileges or suitable `ptrace` permissions.

A kernel stack does not show the application’s user-space call stack. Use GDB, `pstack`, or a core dump for that.

## `cat /proc/<pid>/wchan`

Displays only the kernel function in which the process is waiting:

```
cat /proc/1842/wchan
```

Example:

```
futex_wait_queue
```

Common values include:

```
futex_wait_queue    waiting on a userspace mutex or condition variable
do_wait             waiting for a child process
ep_poll             waiting in epoll
io_schedule         waiting for I/O
pipe_read           waiting for pipe data
```

`wchan` is a quick summary; `/proc/<pid>/stack` provides the surrounding call chain.

## `dmesg -T`

Displays the kernel message ring buffer with human-readable timestamps:

```
sudo dmesg -T
```

It is useful for finding:

- Storage timeouts
- SCSI errors
- Filesystem errors
- Out-of-memory kills
- Device resets
- Kernel warnings and crashes
- Network-driver problems

For example:

```
sd 2:0:0:0: timing out command
blk_update_request: I/O error
EXT4-fs error
Out of memory: Killed process 1842
```

Useful filtering:

```
dmesg -T | grep -Ei 'error|fail|timeout|reset|oom|scsi|nvme'
```

The human-readable timestamps are reconstructed from kernel uptime and can be inaccurate after suspend or wall-clock changes. `journalctl -k` is often another convenient way to view kernel messages on systemd machines.

## `iostat -xz 1`

Reports extended storage-device statistics once per second:

```
iostat -xz 1
```

Options:

- `-x`: Show extended device statistics.
- `-z`: Hide devices with no activity.
- `1`: Refresh every second.

Useful fields vary somewhat by version but commonly include:

|Field|Meaning|
|---|---|
|`r/s`, `w/s`|Read and write operations per second|
|`rkB/s`, `wkB/s`|Read and write throughput|
|`r_await`, `w_await`|Average read/write latency in milliseconds|
|`aqu-sz`|Average number of queued I/O requests|
|`%util`|Percentage of time the device had outstanding I/O|

Warning signs include:

- Increasing `await`: storage requests are taking longer.
- Large `aqu-sz`: requests are accumulating.
- Sustained high `%util`: device is continuously busy.
- Low throughput with high latency: possible storage or path problem.

The first report normally contains averages since boot; subsequent reports represent each one-second interval.

For modern parallel devices such as NVMe and storage arrays, `%util == 100` does not necessarily mean all internal capacity is exhausted. Latency and queue depth are often more informative.

`iostat` is usually supplied by the `sysstat` package.

## `vmstat 1`

Displays system-wide CPU, memory, paging, and scheduling activity once per second:

```
vmstat 1
```

Important fields include:

|Field|Meaning|
|---|---|
|`r`|Runnable tasks waiting for CPU|
|`b`|Tasks blocked in uninterruptible sleep|
|`swpd`|Swap currently used|
|`free`|Free memory|
|`buff`, `cache`|Buffer and filesystem cache|
|`si`, `so`|Swap read/write activity|
|`bi`, `bo`|Blocks read from/written to devices|
|`in`|Interrupts per second|
|`cs`|Context switches per second|
|`us`|User CPU percentage|
|`sy`|Kernel CPU percentage|
|`id`|Idle CPU percentage|
|`wa`|I/O-wait percentage|
|`st`|CPU time stolen by a hypervisor|

Warning patterns:

```
High r + low id       CPU contention
High b + high wa      tasks blocked on I/O
Sustained si/so       active swapping or memory pressure
High sy               heavy kernel activity
High st               virtual machine deprived of physical CPU
```

As with `iostat`, the first line generally represents averages since boot; later lines represent each interval.

## Typical investigation flow

For a slow or hung application:

```
1. ps        Find blocked processes and their wait locations.
2. wchan     Quickly identify the immediate kernel wait.
3. stack     Inspect the complete kernel-side call chain.
4. vmstat    Determine whether the system has CPU, I/O, or memory pressure.
5. iostat    Identify slow or overloaded storage devices.
6. dmesg     Look for device, filesystem, driver, or OOM errors.
```

For example, persistent `D` state, an `io_schedule` stack, high `iostat` latency, and SCSI timeout messages in `dmesg` together strongly suggest a storage-path or device problem.