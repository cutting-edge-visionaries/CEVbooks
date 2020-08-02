Cloud Computing I - Week04 - 2 - Time and Ordering
---


### Introduction and Basics

#### Why is it challenging?

- End hosts in cloud
	- each have thier own clocks
	- Unlike CPUs within one server or workstations which share a ssytem clock
- Processes in internet based systems follow an asyn system model
	- No bounds on
		- message delays
		- processing delays
	- unlike multi processor (or parallel) systems which follow a sync system model

#### Clock SKew vs Clock Drift
- Each process has its own clock
- When comparing 2 clocks at 2 processes:
	- **Clock Skew** relative difference in clock values of 2 processes
		- like distance between 2 vehicles on a road
	- **Clock Drift** rela. diff in clock *freq* of two processes
		- like diff in speeds of 2 vehicles
- a non zero clock skew implies clocks are not synced
- a non zero clock drift causes skew to increase (eventually)

#### How often to Sync?
- **Max drift rate** of a clock
- absolute MDR is defined relative to Coordinated Universal Time (UTC).
	- MDR of a process depends on the env
- MDR btw 2 clocks with similar MDR is `2*MDR`
- Given a max acceptable skew M btw any pair of clocks, need to sync at least once every: `M/(2*MDR)` time units
	- Since time = dist/Speed

--- 

### Cristian's Algorithm

- for external sync if clocks
- all proesses P sync with a time server S
- **Wrong**
	- by the time response msg is received at P, time has moved on
	- P's time set to `t` is inaccurate.
	- Inaccuracy is a function of msg latencies
	- Since latencies unbounded in an async system the inaccuracy cannot be bounded
-  **Cristian's algo**
	- it measures ROUND TRIP TIME, and knows the error
	- suppose message transfer latencies are P -> S min1 and S -> P  min2
	- The Actual time at P when it receives the response is between `[t+min2, t+RTT-min1]`
	- **P sets its time to halfwat through this interval**
		- To: t + (RTT + min1 - min1) / 2
	- Error is at most ( RTT - min2 - min1) / 2
		- Bounded!

#### Gotchas

- only allowed to increase clock balue but **should never decrease clock value**
	- may violate ordering of events within the same process
- Allowed to increase or decrease speed of Clock
- If error is too high, take multiple readings and average them

--- 

### NTP

- Network Time Protocol
- Each Client = Leaf of the Tree
- Each node syncs with its tree parent

| Child | | Parent |
|--|--|--|
| | --- Let's start the protocol -->> | |
| Msg 1 recv time `tr1` | <<-- Message 1 -- | Msg 1 send time `ts1` |
| msg 2 send time `ts2` | -- Message 2 -->> | msg 2 rec time `tr2`  |
| | `ts1` , `tr2` | Time |

- Offset `o = (tr1 - tr2 + ts2 -ts1)/2`
- let's calculate the error 
- Suppose real offset is oreal
	- child is ahead of parent by oreal 
	- Parent is ahead of child by -oreal
- Suppose one-way latency of message 1 is L1 (L2 for msg 2)
- No one knows L1 or L2
- Then 
	- `tr1 = ts1 + L1 + oreal`
	- `tr2 = ts2 + L2 - oreal`
- on substracting 
	- `oreal = (tr1 - tr2 + ts2 -ts1)/2 + (L2 - L1)/2`
	- `oreal = o + (L2 - L1)/2`
	- `|oreal - o| < |(L2 - L1)/2| < |(L2 + L1)/2|`
- thus, the error is bounded by the round-trip-time
- We still have a no zero error
- we just cant seem to get rid of the error
	- Cant as long as the msg latencies are non zero
- There are methos where we can work without syncing the clock
- 

--- 

### Lamport Timestamps

- causality is necessary, i.e. one event leads to other
- used by almost all distributed systems 


#### Logical or Lamport Ordering
- define a logical relation *Happens-Before* (denoted as → ) among pairs of events
- Three rules
	- On the same process *a → b*, if *time(a) < time(b)* [using local clock]
	- if P1 sends m to P2 : *send(m) → receive(m)*
	- (Transitivity) If *a → b* and *b → c* then *a → c*
- Creates a "partial order" among events
	- Not all events related to each other via →


#### In practice
- Rules
	- each process uses a local counter (clock) which is an integer
		- initial value of counter is zero
		- A process increments its counter when a *send* or an *instruction* happens at it. The counter is assigned to the event as its timestamp.
		- *A send(message)* event carries its timestamp
		- For *a receive(message)* event the counter is updated by 
			- *max(local clock, message timestamp) + 1*
- we'll say that with this method the casality is followed
-  concurrent events, no causal paths
- Lamport timestamps not guaranteed to be ordered or unequal for *concurrent event*
- 
- 
 
## little confusions in identifyinf concurrent events

--- 

### Vector Clocks

- Used in key value stores like Riak
- Each process uses a vector of integer clocks
- suppose there are N processes in the group `1...N`
- Each vector has N elements
- Process *I maintains vector* `V<sub>i</sub>[1...N]`
- *j* th element of vector clock at process *i* , `V<sub>i</sub>[j]` , is *i*'s knowledge of latest events at process *j*


#### updating the timestamps

![images04_02_01.png](/images/images04_02_01.png)

![images04_02_02.png](/images/images04_02_02.png)


![images04_02_03.png](/images/images04_02_03.png)

| Lamport Timestamps | Vector Timestamps |
| -- | -- |
| Int clocks assigned to eventts | - |
| Obeys Causality | Obey Causality |
| Cannot distinguish concurrent events | By using more space, can also identify concurrent events |

 


 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNjQxNjkyNiwtMTIzMzk1MDUxNywtMT
U4MzU2MzE4OSwtMTM4ODA5NjUxMCwtMTQwNTMyNzg2NSwtNjgw
NzAwOTM4LC0xMDcyMTI4NDEwLDEyMDExNTY5MTgsLTExODEwOD
ExMTYsMzkzMjYxOTc2LC05OTQ3ODQxMTMsLTEyODI4NDc0NDEs
LTgxNjE3NDI3NiwtOTgyMjExNDIwLC0xOTc0MDIxODcwLDM5MT
U3MTk0NSwxMTgyMDc3NDgwLC03NDY4OTYwODhdfQ==
-->