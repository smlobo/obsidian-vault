# Write path
1. Application `write()`
2. System call
3. Filesystem
4. Page cache
5. Durability request `fsync()`
6. Filesystem submits block IO
7. IO scheduler
8. Device driver
	1. creates SCSI/NVMe commands
	2. mapping memory for DMA
9. Protocol initiator
	1. FC, iSCSI, NVMe/FC, NVME/TCP
10. Network or fabric
11. Storage target
12. Controller cache or NVRAM
13. Persistent media
# Concurrent Queues
```
Application submission queue
  → filesystem/block queue
  → multipath queue
  → driver queue
  → HBA/NIC queue
  → network/switch queues
  → target-port queue
  → controller QoS queue
  → RAID/media queue
```
# Saturation
```
Low depth:
Low throughput, low latency

Increasing depth:
Throughput rises, latency grows slowly

Saturation point:
Throughput approaches maximum

Beyond saturation:
Throughput stays roughly flat,
but latency rises sharply
```
```
Throughput
    |
max |              ───────────
    |           /
    |        /
    |_____ /________________ Queue depth

Latency
    |
    |                    /
    |                 /
    |______________ /_____ Queue depth
```
# Sequential IO
Sequential workloads are often limited by bandwidth:

```
MiB/s or GiB/s
```

rather than maximum IOPS.
# Random IO
Random workloads are often measured primarily in:

```
IOPS and latency
```

# Misc
Fairness may need to consider:

- IOPS
- Bytes per second
- Request size
- Latency objectives
- Backend cost
- Read versus write
- Sequential versus random behavior
## Amplification across multiple layers

Amplification can compound.

Suppose the host writes 4 KiB:

```
Host write:             4 KiB
Filesystem journal:     additional metadata
Copy-on-write layer:    new data and tree nodes
RAID mirror:            two copies
SSD garbage collection: relocates valid pages
Replication:            sends another copy
```

Each layer may be behaving correctly, but the total physical and network work can be much greater than 4 KiB.

This is why measuring only host-visible IOPS can hide backend saturation.

## Example latency distribution

Consider one million operations:

```
p50:   1 ms
p95:   3 ms
p99:  20 ms
p99.9: 400 ms
average: 2.5 ms
```

The average appears healthy, but one in every thousand operations takes at least hundreds of milliseconds. At 100,000 IOPS:

```
100,000 × 0.001 = 100 very slow operations per second
```

A small percentage can still represent many affected requests.
# RAID
Techniques:
- striping
- mirroring
- parity
Recovery:
```
Missing B = A XOR C XOR P
```

```
Snapshot    → “Return this data to an earlier state.”
Backup      → “Recover data even if the primary system is lost.”
Replication → “Keep another system ready with recent data.”
```
```
Production data
  ├── Frequent local snapshots
  │     Fast recovery from recent mistakes
  │
  ├── Replication to secondary site
  │     Fast failover after system/site failure
  │
  └── Independent immutable backups
        Long-term recovery from deletion,
        corruption, or security compromise
```
```
Snapshots:         every 15 minutes, retain 48 hours
Replication:       asynchronous, target lag under 5 minutes
Daily backups:     retain 30 days
Monthly backups:   retain 1 year
Immutable copy:    separate credentials and failure domain
Recovery tests:    scheduled regularly
```

```
For each eligible destination:

predicted utilization =
    (current demand + workload demand + reservations) / usable capacity

score =
    weighted resource utilizations
    + penalty for the most saturated resource
    + migration cost, if applicable

Choose the lowest score.
```
