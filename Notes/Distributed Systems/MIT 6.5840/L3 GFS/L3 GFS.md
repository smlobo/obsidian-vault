# Big Storage
## Why Hard
- Want Performance
	- many servers for parallel access (sharding)
- many servers -> faults
	- need tolerance
- tolerance -> replication
- replication -> in consistency
- consistency -> low performance
# Consistency
## Strong Consistency
```
C1 W X=1
C2 W X=2
C3       R X=2
C4             R X=2
```
# GFS
- big
- fast
- global
- sharding
- automatic recovery
- big sequential access
- 1 master
	- file names -> location of chunks
- many chunk servers
- master data
	- filename -> array of chunk handles
	- chunk handle 
		- list of chunk servers
		- version #
		- primary
		- lease expiration
	- log + checkpoint on disk
## Read
- C (file name, offset) -> M
- M -> handle, list of CS
	- cached
- C -> CS
- CS -> data
## Write (record append)
- No primary
	- M:
		- find up to date replicas
		- pick CS P & S
		- increment V#
		- reply with P, S, V# + Lease (60s)
	- C:
		- appends to P, S
	- P: 
		- picks offset
		- all S told to write at offset
		- `yes` from all S -> P; P -> C `success`
		- `no` from any S -> P; P -> C `fail`
	- C:
		- on `fail` reissues append -> M
- Example of replicas _not_ in sync
	- append(A); `success`
	- append(B); `fail` (partial success to some P + S)
	- append(C); `success`
	- reissued append(B); `success`