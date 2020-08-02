Cloud Computing I - Week02 - 2 - Membership
---


### What is Group Membership List?
- Mean Time to Failure (MTTF)

#### Target Settings
- Process 'group' based systems
	- clouds/datacenters
	- replicated servers
	- Distributed DBs
- crash-stop/fail-stop process failures

#### Group Membership Service

![images02_02_01](/images/images02_02_01.png)

- **Membership List** is the list of all the processes that are currently running.
- All the application queries, *e.g. gossip, overlays, DHTs etc.* keep in sync with this list
- **Membership Protocol** governs the Membership list. 
- *one of the challenges is that this membership protocol has to communicate, over* **unreliable medium** *which can drop or delay the packets.*

![images02_02_02](/images/images02_02_02.png)

- Strongly Consistent Membership, *e.g. computer synchrony*, a well known distributed computing paradigm relies on this
- Partial Consistent List
- Weakly Consistent
- **Failure Detectors** + **Dissemination**


------

### Failure Detectors

#### Disitributed Failure Detectors: Properties
- **Completeness**
	- the failure should be detected eventually (*that means, there is no time bound*)
- **Accuracy**
	- there should be no false positive
- **Speed**
- **Scale**
- Completeness and Accuracy is **impossible** together in *lossy networks.*

#### Failure Detector Properties
| | |
| -- | -- |
|Completeness	|*Guranteed(almost always 100%)*		|
|Accuracy		|*Partial/Probabilistic gurantee(<100%)*|
|Speed			|*Time*| 
|Scale 			|*No Bottlenecks/Single Point of Failure* <br> * Equal Load on each member <br> * Network Message Load|

 #### Centralised Heartbeating
 - p<sub>i</sub> sends periodic heartbeat signals to p<sub>j</sub>
 - Heartbeat is a no. containing sequence no.
 
 ![images02_02_04](/images/images02_02_04.png)

#### Ring Hearbeat
- sends heartbeats to both the left and the right neighbors
- quality of heartbeat is same , *sequence no.*
- ***Failure Condition***
	- if there are multiple failures they may go undetected

![images02_02_05](/images/images02_02_05.png)

#### All-to-All Heartbeat
- heartbeat is sent to all the processes
- equal load per member
- it is complete
- *problem:* 	
	- suppose there is one node p<sub>j</sub>, which is slow,
	- it may mark all the nodes as failed
  
![images02_02_06](/images/images02_02_06.png)

### Gossip-Style Membership
- a variant to all to all heartbeating, just more *robust*
- it has good accuracy properties

![images02_02_07](/images/images02_02_07.png) 

#### Gossip Style Failure Detection
- if the heartbeat has not increased for more than **T<sub>fail</sub>** seconds, the member is considered failed
- and after **T<sub>cleanup</sub>** seconds, it will delete the member from the list
- *Why do we have 2 times?*
	- because it is possible, than one node has deleted it's entry while other hasn't
	- so, that deleted entry may get added again

#### Ananlysis/Discussion
- *What happens if gossip period* **T<sub>gossip</sub>** *is decreased?*
- A single heartbeat takes `O(log(N))` time to propagate

![images02_02_08](/images/images02_02_08.png) 

-----

### Which is the best failure detector?
| | |
| -- | -- |
|Completeness	|*Guranteed always*		|
|Accuracy		| *Probability* **PM(T)** |
|Speed			| **T** *Time units*| 
|Scale <br> * Equal Load on each member <br> * **Network Message Load**	| <br><br> **N*L** compare this across platforms |

#### All to all heartbeating
![images02_02_09](/images/images02_02_09.png) 
*in case of NORMAL ALL-to-ALL HEARTBEATING*


![images02_02_10](/images/images02_02_10.png) 
*in case of* **Gossip-Based** *ALL-to-ALL HEARTBEATING*

- Gossip has higher load than the normal one

#### The best/optimal we can do!
- worst case load **L*** (per member), as a function of *T*, *PM(T)*, *N*
- Independent Message Loss Probabiliity *p<sub>ml</sub>*
<br>

![images02_02_11](/images/images02_02_11.png) 
*not dependent on* **N**

<br>

- The problem is that, Gossip based is trying to do both *Failure Detection* and *Dissemination** together.
- So, the **KEY** is
	- Separate the 2 components
	- Use a non heartbeat-based Failure Detection Component
- 

---

### Another Probabilistic Failure Detector


---

### Dissemination and suspicion

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI4MTAwODE1MywtOTQ1MDY2MDA5LC0yMD
U5MDE1ODQzLDE3NTU4OTc5ODcsLTMwNDAwNDkzMywtMTMxOTk2
NzU3MV19
-->