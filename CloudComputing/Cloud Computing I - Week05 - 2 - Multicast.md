Multicast
--

### Multicast ordering

- used by storage systems like Cassandara or a Database
	- Replica servers for a key: writes/reads to the key are multicast within the replica group
	- All servers: membership inforation (e.g. Heartbeats) is multicast across all servers in cluster
- Online Scoreboards
- Stock Exchanges
	- Group is a set of Broker computers
	- Groups of computers for High Freq Trading
- Air Traffic Control System
	- all controllers need to receive the same updates in the same order 

#### FIFO Ordering

- only cares about message sent by one process and doesn't care about other processes

#### Causal Ordering

- Multicasts whose send events ae causally related, must be received in the same causality-obeying order at all receivers
- Formally
	- *if multicast(g, m) -> multicast(g, m'), then any correct process that delivers m' would already have delivered m*
	- *(-> is Lamport's happens-before)*
- Causal Ordering => FIFO
- 


> Which of the following are true of a causal multicast protocol? (select multiple correct answers)
> - [x] If two multicast sends are causally related, then they are delivered in the order satisfying causality.
> - [x] It satisfies FIFO ordering.
> - [x] It may deliver two multicasts in different orders at recipients.
> - [ ] It always delivers all multicasts in the same order at all recipients.

#### Total Ordering (Atomic Broadcast)

- doesn't pay attention to order of multicast
- Ensures all receivers receive all the multicasts in the same order
- Formally
	- *If a correct process P delivers message m before m' (independent of the senders), then any other correct process P' that delivers m' would already have delivered m*

#### Hybrid Variants

- Since FIFO/Causal are orthogonal o Total, can have hybrid ordering protocols too
	- FIFO-total hybrid protocol satisfies both FIFO and total orders
	- Causal-total hybrid protocol satisfies both Causal and Total order

----

### Implementing Multicast Ordering 1

#### FIFO: 

**Data Structure**
- Each receiver maintains a per-sender sequence number (integers)
	- Processes *P1* through  *PN*
	- P<sub>i</sub> maintains a vector of sequence numbers P<sub>i</sub>[1..N] (initially all zeroes)
	- P<sub>i</sub>[j] is the latest sequence number P<sub>i</sub> has received from P<sub>j</sub>

**Updating Rules**
- Send multicast at process P<sub>j</sub>:
	- Set P<sub>j</sub>[j] = P<sub>j</sub>[j] + 1
	- Include new P<sub>j</sub>[j] in multicast message as its sequence no
- Receive Multicast: if P<sub>i</sub> receives a multicast from P<sub>j</sub> with sequence no. *S* in message 
	- if (S == P<sub>i</sub>[j] + 1) then
		- deliver msg to application
		- Set P<sub>i</sub>[j] = P<sub>i</sub>[j] + 1
	- else buffer this multicast until above condition is true

#### Total Ordering

**Sequencer-Based Approach**
- Special process elected as leader or sequencer
- Send multicast at process P<sub>i</sub>:
	- Send multicast msg M to group and sequencer
- Sequencer:
	- Maintains a global sequence number S (initially 0)
	- When it receives a multicast message M, it set S = S + 1, and multicasts <M,S>
- Receive multicast at process P<sub>i</sub>:
	- P<sub>j</sub> maintains a local received global sequence no. S<sub>i</sub> (initially 0)
	- if P<sub>i</sub>receives a multicast M from P<sub>j</sub>, it buffer it until both 
		- 1. P<sub>i</sub> receives <M, S(M)> from sequencer, and 
		- 2. S<sub>i</sub> + 1 = S(M)
		- Then deliver it msg to app and set S<sub>i</sub> = S<sub>i</sub> 1

> Having the sequencer send an <M, S(M)> message is wasteful because M is effectively sent twice (once by sender, once by sequencer). Someone proposes a protocol where the sender of multicast M maintains a local sequence number F(M) (like FIFO) and includes this with the message M. Thereafter, the sequencer only sends <sender-process-id(M), F(M), S(M)> instead of <M, S(M)>, and recipient processes check this. This protocol:

> - [x] Satisfies total ordering
> - [ ] Does not satisfy total ordering

----

### Implementing Multicast Ordering 2

**Data Structure**
- Each receiver maintains a vector of per-sender sequence no
	- Similar to FIFO 
	- Updating rules are different

**Updating Rules**

![images05_02_01.png](/images/images05_02_01.png)



----

### Reliable Multicast

- loosely says that every process in the group receives all multicasts
	- Reliability is orthogonal to ordering 
	- Can Implement Reliable-FIFO, or Reliable-Causal, or Reliable-Total, or Reliable_hybrid protocols
- But! What happens to the failed processes?
	- *Need all correct (i.e. non-faulty) processes to receive the same set of muticasts as all other correct processes*
	- Faulty Processes are unpredictable, so we won't worry about them

#### Implementing a Reliable multicast
- Let's assume we have reliable unicast (TCP based) available to use
- First-Cut: Sender process (of each multicast M) sequentially sends a reliable unicast message to all group recipients
- First-Cut protocol does not satisfy reliability
	- If sender fails, some correct processes might receive multicast M, while other correct processes might not receive M

#### REALLY implementing Reliable Multicast 
- Trick: Have receviers help the sender
	- 1. Sender process (of each multicast M) sequentially sends a reliable unicast message to all group recipients
	- 2. When receiver receives multicast M, it also sequentially sends M to all the group's processes
- Not Efficient, but Reliable


----

### Virtual Synchrony

- also known as view synchrony
- attempts to preserve multicast ordering and reliability in spite of failures
- combines a membership protocol with a multicast protocol
- Systems that implemented it(like Isis) have been used in BYSE, French Air Traffic Control System, Swiss Stock Exchange

#### Views
- Each process maintains a membership list
- The membership list is called a *VIEW*
- An update to the membership list is called a *View Change*
	- Process Joins, leave, or failure
- Virtual synchrony gaurantees that all *view change are delivered in the same order at all correct processes*
	- if a correct P1 process receives views, say {p1}, {p1, p2, p3}, {p1, p2}, {p1, p2, p4} then
	- Any other correct process receives the *same sequence* of view changes (after it joins the group)
		- p2 receives views {p1, p2, p3}, {p1, p2], {p1, p2, p3}
- Views may be delivered at different physical times at processes, but they are delivered in the same order


#### VSync Multicasts

- multicast M is said to be "delivered in a view V at process P<sub>i</sub>" if 
	- P<sub>i</sub> receives view V, and then sometime before P<sub>i</sub> receives the next view it delivers mutlicast M
- VSync ensures that
	- 1. the set of Ms delivered in a given view is the same set at all correct processes that were in that view
		- what happens in a View, stays in that view
	- 2. The sender of the multicast msgs also belongs to that view
	- 3. if a process P<sub>i</sub> does not deliver a M in view V while other processes in the view V delivered M in V, then P<sub>i</sub> will be *forcibly removed* from the next view delivered after V at the other processes

> Which of the following does view synchrony/virtual synchrony NOT care about?
> - [ ] Delivering the same multicasts within each view
> - [ ] Ensuring that when a multicast is delivered, its sender is in the same view
> - [x] Ensuring that if a process does not deliver a multicast in a view (when others have delivered it), it has another chance to deliver the multicast in the next view


#### About the Name

- called 'Virtual Sync' since in spite of running on an async net, it gives the appearance of a sync net underneath that obeys the same ordering at all processes
- So can this virtually sync system be used to implement consensus?
- NO!! VSync groups are susceptible to partitioning
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMzQ4Njg4ODIsLTE5NDc0Nzk1OTksMT
k1MDQ5MTczNCwxMjczOTY5NzM1LDE0MDU4NzcyODMsLTIwNTI4
MzAzNTksLTExMzM0MjIxMTgsLTEwMzU3ODM4MTQsMTQzNTU4ND
A2MiwtNDkyNzE2NDc5LDY3ODMxNjI1MF19
-->