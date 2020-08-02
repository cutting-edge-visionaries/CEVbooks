Cloud Computing I - Week03 - 1 - P2P Systems
---

**Index**

0. [Introdution](#introduction)
1. [P2P Introduction](#p2p-introduction)
2. [Napster](#napster)
3. [Gnutella](#gnutella)
4. [FastTrack and BitTorrent](#fasttrack-and-bittorrent)
5. [Chord](#chord)
6. [Failures in Chord](#failures-in-chord)
7. [Pastry](#pastry)
8. [Kelips](#kelips)


### Introduction
|||
|--|--|
| unstructred p2p Systems	| *e.g. Gnutella, Napster* etc.	|
| structures p2p systems	| *e.g. Chord, Pastry* etc.		|

*Latter is a pre-cursor to key-value/NoSQL storage systems*


### P2P Introduction
- first distributed systems designed to scale

### Napster
![images03_01_01](images03_01_01.png)

#### Napster Operations
- client
	- connects to Napster Server
		- upload the *list of file*
		- server maintains list of `<filename, ip_address, portnum>` tuples. 
		- **Server stores no file**
	- Search
		- send server the keyword
		- server searches the list
		- server returns the list of hosts, `<filename, ip_address, portnum>`
		- client pings each host in the list to find transfer rates
		- client fetches file from best host
	- all communications uses TCP
	- server maintains the direct information using **ternary tree**

#### Joining a P2P network
- there exists some well known URL, *e.g. www.napster,org*
- message routed (after lookup in DNS) to introducer, a well known server that keeps track of some recently joined nodes in p2p systems
- *introducer initialises new peers' neighbor table*

#### Problems
- centralisation server a source of congestion/single point failure
- no security: plaintext messages and pwd
- next system: Gnutella

### Gnutella
- Eliminate the servers, and clients are used to retreive the results

![images03_01_02](/images/images03_01_02.png)

- peers also stores *peer pointers*
- creates a graph among the peers called as an **overlay graph**

#### Searching
- it contains 5 main types of messages
	- **Query** (search)
	- **QueryHit** (response to query)
	- **Ping** (to probe netowork for other peers)
	- **Pong** (reply to ping, contains address of another peer)
	- **Push** (used to initiate file transfer)
- **Message Structure**
	- all fields except IP Address are in ***Little-Endian Format*** (*rather than Big-endian format*)
	- `0x12345678` stored as `0x78` in lowest address byte, than `0x56` in next higher address, and so on.

![images03_01_03](/images/images03_01_03.png)

- **Query** Message

| 0-1 bytes| ... |
|--|--|
|Minimum Speed |Search Criteria(keywords) |

- query's flooded out, ttl-restricted, forwarded only once

<br>

- How does Search results come back?

#### Gnutella Search

- **Query Hit** Message is used for that

![images03_01_04](/images/images03_01_04.png)

- no of hits = no of files that match
- servent_id is sometimes there to store information of the sender of the message
- routed on the *reverse path*

#### Avoiding Excessive Traffic
- to avoid duplicate transmission, each peer maintains a list of recently received messages
- query forwarded to all neighbors except peer from which received 
- each query (identified by *Descriptor's ID*) forwarded only once
- QueryHit routed back only to peer from which Query received with same DescriptorID
- for flooded messages, duplicates with same DescriptorID and Payload descriptor are dropped
- QueryHit with DescriptorID for which Query not seen is dropped
- in case the the overlay graph changes(*maybe when the network changes*) the Query results may not be shown, and dropped, but we hope the results drop will be small

#### After Receiving Query Hit Message
- Requestor chooses **"best"** QueryHit responder, *depending on Bandwidth and download starts*
	- initiates "GET" HTTP request directly to responder's ip+port
![images03_01_05](/images/images03_01_05.png)

	- range field is used to to *partial file transfer*
	- suppose, you wanted to transfer 1MB and connection gets killed at 512KB, then you can set the range to 512KB for partially sending the file
	
- Responder then replies with file packers after this message:

![images03_01_06](/images/images03_01_06.png)

- *What if the responder os behind the Firewall?*
	- that where the **PUSH** message comes into play
	- it responds with PUSH on the onverlay network
	- PUSH message is reverse routed on reverse Query Hit path

#### Dealing with Firewalls

<table>
<tr>
	<th colspan=4> **PUSH (0x040**) </th>
</tr>
<tr>
	<td colspan=2> *same as in received QueryHit msg*</td>
	<td colspan=2> *address at which requestor can accept incoming connections* </td>
</tr>
<tr>
	<td> servent_id </td>
	<td> fileindex </td>
	<td> ip_address </td>
	<td> port </td>
</tr>
</table>

- if the requestor is behind the firewall too
	- Gnutella gives up
	- for that modified version of Gnutella could use the overlay links themselves to transfer the file(*slow though*)

#### Ping - Pong

![images03_01_07](/images/images03_01_07.png)

- *When a Gnutella peer receives a Ping message from one of its neighbors, which of the following actions does it perform?*
	- It forwards it to appropriate neighbors after checking TTL.
	- It creates a Pong message about itself and reverse routes it.
	- If it was the original peer that initiated the Ping, it uses received Pong responses to update its membership lists.
	- It reverse routes any Pong messages it receives.

#### Problems
- Ping Pong consumes 50%  of the traffic
- Modem connected hosts cannot handle Gnutella traffic
- large freeloaders (70% os users in 2000)
- Flooding causes excessive traffic



### FastTrack and BitTorrent

#### FastTrack
- Hybrid bw Gnutella and Napster
- "healthier" node concept
- Underlying tech : *Kazaa, KazaaLite, Grokster*
- Like Gnutella, but some peers as ***supernodes***
- Supernodes can work as servers in napster
- supernode membership changes over time, based on the reputation
- uploading, downloading increases th reputation
- *initial reputation is 10*
- a peer contacts by connecting *directly* to nearby supernode

#### BitTorrent
- have *Trackers* (per file)
	- maintains the list of peers currently trasnfering the files
	- uses Heartbeats for that
- have incentives to encourage peers participate
- peers are of 2 kinds
	- ***seed*** which has full files
	- ***leechers*** has some blocks of file(typically equal sizes)

![images03_01_08](/images/images03_01_08.png)

- block size (`32KB~256KB`)
- uses **Local Rarest First** block policy
	- prefers early download of blocks that are least replicated among neighbors
	- as due to churn the blocks may disappear
	- *exception:* new node allowed to pick one random neighbor, which helps in bootstrapping and it doesnt get stuck to a particular path
- Incentives are provided for Best download rates, *Tit for Tat* bandwidth usage
	- seeds do the same
- concurrent uploads can overload the servers
- for this **Choking** is used.
	- limit no of neighbors to which concurrent uploads <= 5, *i.e.* "best" neighbors
		- everyone else is **choked**
		- periodically re evaluate this set (e.g. every 10s)
		- *Optimistic Choke:* periodically (e.g. ~30s) unchoke a random neighbor helps keep unchoked set fresh
- random choices are used in BitTorrent Choking Policy, to avoid the system from getting stuck where only a few peers receive service


### Chord
- first p2p network launched for academia purpose

#### Distributed Hash Tables
- hash table typically maintained inside one machine, where objects can be deleted or updates using a unique *key*
- hash tables lets you to store these objects in the buckets
- Distributed hash table runs on distributed systems
- **Performance Concerns**
	- *Load Balancing* each node to have same no of objects as other
	- *Fault Tolerance*
	- *Efficiency of lookups and inserts*
	- Locality
- Napsters, Gnutella. FastTrack are all DHTs(sort of)


####  Comparative Perfomance

| | Memory | Lookup Latency | #messages for a lookup |
|--|--|--|--|
| Napster 	| `O(1)` <br> `O(N)`@server | `O(1)` | `O(1)` 	|
| Gnutella 	| `O(N)` 		| `O(N)` 		| `O(N)` 		|
| Chord		| `O(log(N))` 	| `O(log(N))`	| `O(log(N))`	|

- for practical purposes `log(n)` is constant

#### Chord
- intelligent choice of neighbors to reduce latency and message cost of routing (lookups/inserts)
- whereas the Gnutella servers made choices on the basis of KBs sharead etc.
- uses ***consistent hashing*** on  node's address
	- SHA-1 (ip_address.port) -> 160 bit string
	- Truncated to *m bits*, where *m* is a system parameter
	-   which gives you the peer id (*no bw* **0** & **2<sup>m</sup> - 1**) [inclusive]
	- not unique but id conflicts very unlikely
	- can then map peers to one of **2<sup>m</sup>** logical points on a circle

#### Ring of Peers

- types of neighbor used in Chord P2P systems
	- successors
	- Finger Tables
	- Predecessors (if needed)

**first kind of peer pointers**

![images03_01_09](/images/images03_01_09.png)

- **2<sup>m</sup> - 1** points are laid in cicular fashion and each know
	- their next neighbor's IP address, in case of Successors

**2nd type: Finger Table**

![images03_01_10](/images/images03_01_10.png)

- i<sup>th</sup> entry at the peer with id *n* is first peer with 
	- id >= (n + 2<sup>i</sup>) (mod 2<sup>m</sup>)

*e.g. Finger Table entries*

| i | *ft[i]=* (n + 2<sup>i</sup>) 	| node >= value	|
|--|--|--|
| 0 | 80 + 2<sup>0</sup> = 81		| 96	|
| 1 | 80 + 2<sup>1</sup> = 82		| 96	|
| 2 | 80 + 2<sup>2</sup> = 84		| 96	|
| 3 | 80 + 2<sup>3</sup> = 88		| 96	|
| 4 | 80 + 2<sup>4</sup> = 96		| 96	|
| 5 | 80 + 2<sup>5</sup> = 112		| 112	|
| 6 | 80 + 2<sup>6</sup> = 144*		| 16	|

 - *since 144 isn't there so, we need to do (mod 2<sup>m</sup>)
	 - *i.e*  (80 + 2<sup>6</sup>) mod (2<sup>6</sup>)
		 - `144 % 128 = 16`
 
 
 #### What about the Files?
 - Filename also mapped using same consistent hash function
	 - truncated same a peers
	 - file stored in *first peer with id greater than its key*
 - 

> Q. In a Chord ring with m=7, three successive peers have ids 12, 19, 33 (there are other peers in the system too, but not in between 12 and 33). If the number of files is large and a uniform hash function is used, which of the following is true?
> Ans. Peer 33 stores about double the number of files as peer 19.

#### Mapping Files
- 
# *chord portions left

#### Search
 

### Failures in Chord


### Pastry
- assigns ids to nodes, just like Chord(*using virtual ring*), using consistent hashing functions
- **Leaf Set** - Each node knows its *successor* and *predecessors* 

#### Pastry neighbors
- **Routing Tables** based *prefix Matching*
	- think of a *hypercube routing*
	- "*Hypercube networks are a type of network topology used to connect multiple processors with memory modules and accurately route data. Hypercube networks consist of* 2<sup>m</sup> *nodes.* ***These nodes form the vertices of squares to create an internetwork connection.*** *A hypercube is basically a multidimensional mesh network with* ***two nodes in each dimension***. Due to similarity, such topologies are usually grouped into a k-ary d-dimensional mesh topology family where d represents the number of dimensions and k represents the number of nodes in each dimension*"
- routing thus based on prefix matching, and is thus `log(N)`
	- and hops are short(in the underlying network)

#### Pastry Routing
- Consider a peer with if `01110100101`. It maintains a neighbor peer with an id mathching each of the following prefixes.
- when it needs to route to a peer, say `01110111001`, it starts by forwarding to a neighbor with the largest matching prefix, *i.e.* `011101*`
- the order goes like this
	- `*`
	- `0*`
	- `01*`
	- `011*`
	- ....
	- `...0111010010*`
	- it goes to the largest matching prefix

#### Pastry Locality
- for each prefix, say `*011*`, among all potential neighbors with a matching prefix, the neighbor with the shortest round-trip-time is selected
- since shorter prefixes have many more candidates, the neighbors for shortes prefixes are likely to be closer than the neughbors for longer prefixes
- thus, in the prefix routing, early hops are short and later hops are longer
- yet overall "strectch", compared to direct internet path, stays short

### Kelips

<!--stackedit_data:
eyJoaXN0b3J5IjpbNjQ4MTk4Njc2LC0zMDgyMDUzODEsLTk2Mj
IzODU1OSwxMTE4MTA5MTU4LC0xODA2NzUzNjIsLTY2NzQ0Mjks
NjY5MzU3NDM5LDI0NTE1NjEwNSwtMTIwNzAyOTc0MywxOTMzMD
M1NTgzLDU3ODU0MzE0MCwtMTEzMDg3NDU3MSw0OTcyNzIxNjIs
MTE1MjQyODc1NSwtMTgxNzY5MTc4LDg0NzU4MTU3LC00MDI5Mz
E1NzgsMjAzNTczNTQ3Miw2NjY0NTQwMDFdfQ==
-->