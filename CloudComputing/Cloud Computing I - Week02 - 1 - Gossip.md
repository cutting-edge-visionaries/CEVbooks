Cloud Computing I - Week02 - 1 - Gossip
---


### Multicast Problem

#### Multicast
- suppose one node in a group of nodes wants to send the packets to a groups of nodes, it is called as the the Multicast.
- Extreme form of multicast is known as *Broadcast*, where one node sends packets to every other node in the network

#### Fault Tolerance and Scalability
- for this *Multicast Protocols* are used, which should be
	- Fault tolerant
	- scalable
- the protocol sits at the application level and not on the Hardware levels, in routers and switches

#### Centralized
- sends **UDP/TCP** packets
- have sender and a list of receivers
- it is prone to fault tolerant
- it also increases the *latency*
 
#### Tree-Based
- such protocols are developed
- *e.g. App level protocols like RMTP, TRAM, TMTP* etc.
- when node failures happens, nodes may not receive packets for a while
- easy troubleshooting

#### Tree - Based Multicast Protocols
- use spanning tree to disseminate multicasts
- use either *ACKs* or *NAKs* (negative acknowledgements) to repair multicasts not received
- *e.g.* SRM(Scalable Reliable Multicast)
	- uses NAKs
	- but adds random delays, and uses exponential backoffs(*waits everytime for double the time*) to avoid NAK storms
- RMTP (Reliable Multicast Transport Protocol)
	- uses ACKs
	- sometimes there is ACK storms
	- for this ACKs only sent to designates receivers, which then re-transmit missing multicasts
- ***These protocols still cause an O(N) ACK/NAK overhead***

-----


### The Gossip Protocol
#### A third approach
- supposethere is one multicast sender, which chooses *b* random nodes to send the message to, periodically, say 5 seconds
- it uses the connectionless protocol, *UDP*
- the messages are called *Gossip Messages*
- other nodes do same after receiving multicast, periodically, say 5 seconds
- so there is good amount of wastage as well

#### Epidemic
- When a node receives a message, it is said to be *infected* by the message
- others are called *uninfected*


#### PUSH vs PULL
- this is *PUSH* gossip protocol
	- once a node receives multicast you start gossiping
	- in case of multiple messages, 
		- gossip one msg at a time, or a subset of msgs
		- you may also gossip the recently received ones
- other is *PULL* gossip protocol
	- reverse of PUSH protocol
	- queries regarding the messages is periodically is sent to few of the close nodes asking for any messages
- HYBRID


-----

### Gossip Analysis

- lightweight even in large groups
- spreads a multicast quickly
- is highly fault tolerant

#### Ananlysis
- from old maths *Epidemiology*
- classical version has *(n+1)* individuals
- contact rate between any individual pair is $\beta$
- say, 
	- *x* : infected , *y* : uninfected
	- then, `x0 = n`, `y0 = 1`, all the time `x + y = n + 1`
- continous time process
![images02_01_01](/images/images02_01_01.png)

#### Epidemic Multicast Analysis
![images02_01_02](/images/images02_01_02.png)

- set `c`, `b` to be small no. independent of `n`
	- Low latency
	- reliability
	- lightweight
![images02_01_03](/images/images02_01_03.png)

*Note: `log(n)` is very small thus, almost constant*

#### Fault Tolerance
- Packet Loss
	-  analyse with `b -> (b/2)`
	- to achieve same reliability as `0%` packet loss, takes *twice* as many rounds
- Node Failure
	- 50% of nodes fail: analyse with `n -> n/2` and `b -> (b/2)`-
- it is **possible** that the gossip dies out quickly, but it is **improbable**
	- once a few nodes are infected, with high probability, the epidemic will not die out
	- so the analysis we saw in the prev. portion is actually bahviour with high probability

#### PULL Gossipt
- in all forms of gossip, it takes `O(log(N))` rounds before about `N/2` gets the gossip
- PULL faster than PUSH
- *k = no. of gossip pulls per round per process*

![images02_01_04](/images/images02_01_04.png)

***in HYBRID, in first half PUSH is used, in second half PULL is used***

#### Topology-Aware Gossip
- Gossip protocols are not Topology aware
- so they get trafficed when it comes to hierarchical topology
- suppose gossip need to go from one subnet to other
- since each node is selected at random, ***n/2*** will go to another subnet
- probability of going **over network** is 1/n<sub>i</sub>
- prob of going within the network is 1 - (1/n<sub>i</sub>)
- initially rouuter load is O(N), which goes down to O(1), when all the nodes are once infected
- Dissemination TIme : `O(Log(N))`

> In a datacenter topology with top of the rack switches talking to one core switch, there are 2 racks, each with N/2 nodes from the multicast group. The nodes execute push gossip. If each node picks a gossip target outside its rack with probability (1/N), and gossip targets are selected uniformly at random from the target subgroup, the time to disseminate a gossip is:
> - [x] O(log(N))
> - [ ] O(N)
> - [ ] O(log(log(N)))

*Check if your analysis is correct.*

![images02_01_05](/images/images02_01_05.png)

-----

### Gossip Implementation

#### NNTP Inter Server Protocol

![images02_01_06](/images/images02_01_06.png)

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTc2MzU5MDUwMl19
-->