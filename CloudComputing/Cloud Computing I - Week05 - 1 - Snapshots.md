Snapshots
---

-   **[Google Chubby](http://research.google.com/archive/chubby.html)**
-   **[Apache Zookeeper](http://zookeeper.apache.org/)**


-----

### What is Global Snapshot?

- suppose on cloud an app is running on multiple servers
- Servers handling concurrent events and interacting with each other
- Uses of Global Snap
	- *Checkpointing*: on failure the distributed app can be restarted
	- *Garbage collection of objects:* Garbage are the objects to the servers that dont have any other objects(in any server) with pointers to them
	- *Deadlock Detection:* useful in DB txns systems
	- *Termination of computation* for Batch computing systems, *e.g. Folding@Home, SETI@Home*
- **Global Snapshot == Global State**
	- Individual state of each (*process in the Distributed System + Communication channel in the distributed system*)
- Capture the **Instantaneous** state of each process
- and the instantaneous *state* of each communication channel, i.e., *messages* in transit on the channels


#### Obvious solution

- sync the clocks of all the processes
- ask all of the processes to record their states at known time *t*
- but, Time sync is not good, it has errors
- Thus, sync is not required, CAUSALITY is enough!
- State-to-state movement obeys causality
- 

----


### Global Snapshot Algorithm

- ***System Model***
	- `N` processes in the system
	- There are uni-directional communication channels between each ordered process pair: 
		- `P<sub>j</sub> → P<sub>i</sub>`
	- Communication channels are FIFO-ordered
		- First in First out
	- No failure
	- All msgs arrive intact, and are not duplicated 
		- *other papers later relaxed some of these assumptions*
- **Requirements**
	- Snaps should not interfere with any apps in any way
	- Each process should be able to record its own state
		- *Process state*: App-defined state or, in the case: *its heap, registers, program counter, code etc. (essentially the core dump)*
	- Global state is collected in a distributed manner
	- Any process may initiate the snaps usually distinguished by using different IDs


#### Chandy-Lamport Global SnapShot Algo

- **Initiator** P<sub>i</sub> records its own state
- Initiator Process creates special msgs called "MARKER" msgs
	- Not an app msg, does not interfere with app msgs
	- but when sent on channel, they are ordered with app msgs
- for  `j= 1 to N` except `i` 
	- P<sub>i</sub> sends out MARKER msg on outgoing channel C<sub>ij</sub>
	-  `(N-1)` channels
- *Starts recording* the incoming msgs on each of the incoming channels at 
	- P<sub>i</sub> : C<sub>ji</sub> (for `j = 1 to N` except `i`)
	- 
- **When process**  P<sub>i</sub> **receives the MARKER**
	- if (this is the first MARKER P<sub>i</sub> is seeing)
		- P<sub>i</sub> records its own state first
		- Marks the state of channel C<sub>ij</sub> as "empty"
		- for `j = 1 to N` except `i`
			- P<sub>i</sub> send out a MARKER msg on outgoing channel C<sub>ij</sub>
		- *Starts Recording* the incoming msgs on each of the incoming channels  P<sub>i</sub> : C<sub>ji</sub> (for `j=1 to N` except `i` and `k`)
	- else `||` already seen a MARKER msgs
		- MARK the state of channel  C<sub>ki</sub> as all the msgs that have arrived on it *since recording was turned on for* C<sub>ki</sub>
- ***The algo terminates when**
	- All processes have received a MARKER
		- to record their own state 
	- All processes have received a MARKER on all the `(N-1)` incoming channels at each
			- To record the state of all channels

***Then, if needed, a central server collects all these partial state pieces to obtain the full global snapshot***


----

### Consistent Cuts

- **CUT**
	- time frontier at each process and at each channel
	- Events at the process/channel that happen before the cut are ***"in the cut"*** 
	- after the cut are ***"out the cut:***
- **Consistent Cut**
	- a cut obeys causality
	- a cut C is a consistent cut if and only if:
		- for (each pair of events e, f in the system)
			- Such that event e is in the cut C, and if **f → e** (f happens before e)
				- THEN: event f is also in the cut C

![images05_01_01.png](/images/images05_01_01.png)

*"Any run of the Chandy-Lamport Global Snapshot algorithm creates a consistent cut"*

-----

### Safety and Liveness

- Liveness: Something GOOD will happen eventually
- Safety:  Something BAD will never happen in the system
- Chandy Lamport can be used to detect global properties that are stable
	- STABLE = once true, stays true forever afterwards (*i.e. for infinite time afterwards*)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzMzM5NzY2OTMsMTk5NDk0NTMwOSwxNT
cyMzE0MzQzLDExNDM1OTgxNTksMjE0NDc5ODk3LC0xNzgzNzg3
NjMyLC0xNDUwOTc5NDYzLDE4MDI3Mjk0NTAsLTUzMjQ1ODY4Ni
wtNzgwMjgxOTk1XX0=
-->