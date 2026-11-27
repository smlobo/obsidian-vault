- https://youtu.be/d7nAGI_NZPk?si=PTnzMCwVfo-26KzB

# High level
- consensus is agreeing on 1 _result_
- once a _majority_ agrees on a _proposal_ , that is a _consensus_
- the reached _consensus_ can be _eventually_ known by everyone
- the involved parties want  to agree on _any_ result, not on their proposal
- communication channels may be faulty (messages can get lost)

# Paxos Nodes
- 3 roles
	- proposers
	- acceptors
	- learners
- nodes can take multiple roles
	- even all of them
- must know how many _acceptors_ a majority is
	- 2 majorities will always overlap by 1 node)
- must be persistent - they cannot forget what they accepted

## Paxos run
- aims at reaching a _single consensus_
- once consensus is reached, it _cannot progress_ to another consensus
- in order to reach a different consensus, a different Paxos run must happen

# Paxos Algorithm
- ==🟡Proposer== sends `PREPARE IDp` to >= majority of ==🔴Acceptors==
	- IDp must be unique
- ==🔴Acceptor== receives a `PREPARE IDp`
	- Did it promise to ignore requests with this IDp?
		- Yes -> ignore
		- No ->
			- Has ==🔴Acceptor== ever accepted anything?
				- No -> Reply with `PROMISE IDp` (promise to ignore any request < IDp)
				- Yes -> reply with `PROMISE IDp accepted IDa, <value>`
- ==🟡Proposer== gets majority of`PROMISE IDp`messages, it sends `ACCEPT-REQUEST IDp, <value>` to >= majority of ==🔴Acceptors==
	- Has it got an already accepted `<value>` with the `PROMISE`?
		- No -> it picks any value it wants
		- Yes -> picks the `<value>` with the highest `IDa`
- ==🔴Acceptor== receives an `ACCEPT_REQUEST IDp, <value>`
	- Did it promise to ignore requests with this IDp?
		- Yes -> ignore
		- No -> Reply with `ACCEPT IDp, <value>`; also send to all ==🟢Learners==
- ==🟡Proposer== or ==🟢Learner== get `ACCEPT IDp, <value>`
	- If these get >= majority for specific IDp, they know consensus has been reached for `<value>`
## Key points
- If a majority of ==🔴Acceptors== `PROMISE IDp`, no ID < IDp can make it through
- If a majority of ==🔴Acceptors== `ACCEPT IDp, <value>` , consensus is reached
	- Consensus is always on `<value>`
- If a ==🟡Proposer== / ==🟢Learner== gets a majority of `ACCEPT IDp, <value>`, they know that consensus has been reached for `<value>`

## Examples
- `PROMISE` only
![[promise only.png]]
- `PROMISE` with `accepted`
![[promise with accepted.png]]