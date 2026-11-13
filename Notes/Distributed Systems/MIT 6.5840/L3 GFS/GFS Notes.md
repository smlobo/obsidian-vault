# Design Overview
## Interface
- usual operations
	- create, delete, open, close, read, write
- snapshot
	- copy of file or dir tree at low cost
- record append
	- multiple clients to append data to the same file
# Architecture
- single `master`
- multiple `chunkservers`
- files are fixed size `chunks` (64MB)
- identified by immutable & globally unique 64-bit `chunk handle`
- `chunks` replicated on multiple `chunkservers`
- `master` maintains all file system metadata
	- namespace
	- access control
	- file to chunk mapping
	- current location of chunks
- `master` controls system wide activities
	- `chunk` lease mgmt
	- garbage collection of orphan `chunks
	- `chunk` migration
- `master` periodically communicates to `chunkservers`
	- `heartbeat`
		- give instructions
		- collect state
![[GFS.png]]
## Metadata
- in memory data structures on `master`
	- fast
- chunk locations are polled from `chunkservers`
- operation log
	- used to recover file system state by replaying
	- log is kept small
	- `master` checkpoints its state
	- recovery needs latest checkpoint + subsequent log files
## Consistency Model
- relaxed consistency
# System Interaction
## Leases & Mutation Order
![[GFS Write Order.png]]
## Data Flow
## Atomic Record Appends
## Snapshot
# Master Operation
## Namespace Management & Locking
## Replica Placement
## Creation, Re-replication, Rebalancing
## Garbage Collection
## Stale Replica Detection
# Fault Tolerance
## High Availability
## Data Integrity
