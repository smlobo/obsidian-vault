# Why
- fault tolerance
- performance
- physical separation

# Labs
1. MapReduce
2. Raft
3. KV Server
4. Sharded KV server

# Infrastructure
- Storage
- Communication
- Computation
Abstractions for each of these
## Implementations / Tools
- RPC (Remote Procedure Calls)
- Threads - concurrent operations
	- locks
## Performance
- scalability
## Fault Tolerance
- why
	- scale implies always failures
	- mask failures so application programmers do not have to deal with them
- Availability
	- replication - drift out of sync
- Recoverability
	- non volatile storage - save a checkpoint
## Consistency
- KV store `put(k, v)` & `get(k) -> v` result in consistent results
- strong consistency
	- get sees the value from put
	- needs lots of expensive communication to keep replicas consistent
- weak consistency
	- cheaper to implement 
![[Client-Server-DB]]
# MapReduce
Both simple functions run on different servers.
![[MapReduce]]
## Map Function
```
Map(k,v)
	split v into words
	for each word w
		emit(w, "1")
```
## Reduce Function
```
Reduce(k,v)
	emit(len(v))
```

# Reading
- [[mapreduce-osdi04.pdf]]
- 