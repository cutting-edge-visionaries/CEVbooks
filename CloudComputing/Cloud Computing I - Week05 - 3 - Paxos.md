Paxos
---

### The consensus Problem

- consensus is impossible to solve under certain sytems
- A group of servers attempting to do
	- Reliable Multicast
	- Membership/Failure Detection (list of each other)
	- Leader Election
	- Mutual Exclusion of Critical Resources
 
#### What is Consensus?

Formal problem statements
- N processes
- Each process p has 
	- I/p variable Xp : initially 0 or 1
	- o/p variable Yp : initially b (can be changed only once)
- *Consensus Problem:* design a protocol so that at the end, either:
	- 1. All processes set their o/p variables to 0 (all-0's)
	- 2. or all processes set their o/p variables to 1 (all-1's)
- Other Constraints
	- Validity
	- Integrity
	- Non-Triviality: at least one initial system state that leads to each of the all-0's or all-1's outcomes
- **2 different models of distributed systems** (*understand this stuff for any prob given to you*)
	- Synchronous & Asynchronous
	- ***Sync. Distributed Systems***
		- Each msg is received witihin bounded time 
		- Drift of each process' loca clock has a known bound
		- Each step in a process takes lb < time < ub
		- e.g. a collection of processors connected by a communication bus, e.g. a CRAY supercomputer or a multicore machine
		- `Consensus is solvable`
	- ***Async Distributed Systems***
		- No bounds on process execution
		- The drift rate of a clock is arbitrary
		- No bounds on msg transmission delays
		- e.g. Internet, ad-hoc, sensor networks etc
		- ***More difficult to tackle***
		- if protocol works for async, it will also work for sync.  systems
		- `Consensus is impossible to solve`


---

### Consensus in Synchronous Systems
- First know what is system model
- *Sync system* has bounds on 
	- message delays
	- upper bound on clock drift rates
	- Max time for each process step
- For a system at most *f* processes crashing 
	- all processes are synced and operate in "rounds" of time
	- the algo proceeds *f+1* rounds (with timeout), using reliable communication to all members
	- Values<sup>r</sup><sub>i</sub> *the set of proposed value known to* p<sub>i</sub> *at the beginning of rounds* r 

![images05_03_01.png](/images/images05_03_01.png)


#### Why does the Algorithms work?

- After `f + 1` rounds, all non-faulty processes would have received the same set of Values. Proof by contradiction
- Assume that two non-faulty processes, say p<sub>i</sub> and p<sub>j</sub>, differ in their final set of values (i.e., after `f + 1` rounds)
- Assume that p<sub>i</sub> possesses a value *v* that p<sub>j</sub> does not possess.

![images05_03_02.png](/images/images05_03_02.png)



---

### Paxos, Simply

- used by Zookeeper (Yahoo!), Google Chubby
- invented by Leslie Lamport
- FLP still applies : Paxos is not guaranteed to reach the consensus


#### how does it work
- it has rounds, with unique Ballot id
- it is async
	- i.e. Time sync is not required
	- if you are in round *j* and hear a message from round *j+1* , **abort everything** and move over to round *j+1*
	- use timeouts
- **3 Phases** *Election, Bill & Law*
- **Phase I: ELECTION**
	- potential leader chooses (highest) unique ballot id
	- PL needs to get votes from others
	- so they send out the ballot id to everyone
	- the other members vote for the highet ballot id they see
	- if PL sees the higher ballot id then it refuses to be the leader
	- When PL reaches the quorom of votes it becomes leader
	- *Error* 
		- it is possible nonne reaches majority
		- in corner cases it is possible the tht multiple leader are selected
	- the ballot ids are also logged
	- in case of no leader, next round is started
- **Phase II: BILL**
	- leader sends a proposed value to all
	- if it knows the value from the prev round, it uses that value
	- if no value is decided in the prev round, it can use its own decided value
	- Recipients logs in the disk
- **Phase |||:LAW**
	- when Leader reaches the quorom 
	- it tells it to everyone


#### Which is the point of NoReturn?
- when the consensus is reached in the system?
- it is reached at the middle of the BILL phase, where the processes have logged on it
- processes dnt know it yet, but it is decided and cannot be changed now
- even if the leader fails, the rounds are continued until a value is reached

#### How Safety works?
- **Proof**
	- PL waits for majority of OKs in phase I
	- At least one will contain v' (bcz 2 quoroms always intersect)
	- it will chose to send out v' in Phase 2

#### What could go wrong?
- Process fails & Majority does not include it
	- when process restarts, it tries to look at the log
- Leader Fails
	- another round started
- Msgs dropped
	- another round started
- New round can be started at any point of time
- and **EVENTUALLY** it may never end thus ***EVENTUAL LIVENESS***
- 

---

### The FLP Proof [Optional]

- *Key to the Proof*: It is impossible to distinguish a failed process from one that is just very very (very) slow. Hence the rest of the alive processses may stay ambivalent (forever) when it comes to deciding.
<!--stackedit_data:
eyJoaXN0b3J5IjpbOTcxNDA4NTc4LDMzNjA0MzIzMywtNDkyMz
E5MTMxLC0yMDI1NDAzMzg3LDE1OTA5OTc1MTAsLTUxODcwMTI2
MCwtMjY3MDQ4NDg0LDEzNDgyMjk0NzAsMjEwMzc5NzgyMCwtMz
Y5Mzc2MzMzXX0=
-->