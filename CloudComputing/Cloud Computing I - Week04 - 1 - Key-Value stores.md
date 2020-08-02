Cloud Computing I - Week04 - 1 - Key-Value stores
---

#### Reading Resources

-   **[Cassandra](http://www.datastax.com/documentation/cassandra/2.0/cassandra/gettingStartedCassandraIntro.html "Link: http://www.datastax.com/documentation/cassandra/2.0/cassandra/gettingStartedCassandraIntro.html")**
-   **[HBase](http://hbase.apache.org/)**
-   [**Cassandra 2.0 paper**](http://docs.datastax.com/en/articles/cassandra/cassandrathenandnow.html)
-   [**Cassandra NoSQL Presentation**](http://www.slideshare.net/Eweaver/cassandra-presentation-at-nosql)
-   [**Cassandra 1.0 documentation at datastax.com**](http://docs.datastax.com/en/archived/cassandra/1.0/docs/)
-   [**Cassandra Apache wiki**](http://wiki.apache.org/cassandra/ArchitectureOverview)
-   [**MongoDB**](https://www.mongodb.com/)


----

### Why Key-Value/NoSQL?

#### Key value Abstraction
- `Key` -> `Value`
- dictionary DS
- sort of a Database
- RDBMS is 
	- Schema based - Structured Table
	- each row has  a unique primary key 
	- Queried using SQL (Structured Query Language)
	- Supports Joins
	- may have foreign keys, which are used to lookup and relate to another table

#### Bad things about RDBMS kind DBs
- Data large and unstructured
- lots of random reads and writes are present
- sometimes write-heavy 
- foreign keys are rarely needed
- joins infrequent
- 


#### Today's needs
- speed
- avoid SPoF
- Low Total Cost of Operation/Ownership
- Fewer System Admins
- incremental Scalability
- Scale out, Not Scale up

#### Scale out, Not Scale up
- **Scale up** grow your cluster capacity by replacing with more powerful machines
	- traditional approach
	- not cost effective, as you are buyin above the sweet spot on the price curve
	- And you need to replace machines often
	
- **Scale out** incrementally grow your cluster capacity by adding more COTS(*Components Off the Shelf*)
	- cheaper
	- over a long duration, phase in a few newer(faster) machines as you phasse out a few older machines
	- used by most companies who run datacentres and clouds today

#### Key-value/NoSQL Data Model
- NoSQL = "Not only SQL"
- apis `get()` `put()`
- some powerful operations, *e.g. "CQL in Cassandra Key-Value Store"*
- **Tables**
	- "Column Families" in *Cassandara*
	- "Table" in HBase
	- "Collection" in MongoDB
- *They are tables in RBDMS but,
	- may be unstructed: May not have schema, i.e. some things may get going missing
	- Don't always support joins or have foreign keys
	- Can have index tables, just like RDBMS

![images04_01_01.png](/images/images04_01_01.png)

> Which of the following may NOT be present in key-value/NoSQL stores? 
> - [x] Joins
> - [x] Foreign Keys
> - [ ] Tables
> - [ ] Rows
> - [x] Same Columns in Some rows

#### Column Oriented Storage
![images04_01_02.png](/images/images04_01_02.png)

> Why is column-oriented storage useful? (select all correct answers)

> - [x] Queries that touch only a few columns are faster than those in a row-oriented storage.
> - [x] It enables faster range searches using one column.
> - [ ] It leads to lower total storage for the table.
> - [x] It prevents the entire table from being read for a query.


----

### Cassandara
- distributed key-value store
- intended to run in a datacenter
- designed by FB
- Open Source
- used by IBM, Adobe, HP, eBay, Ericsson, twitter, spotify
- Netflix uses to keep track of your current position in the video you're watching


#### Design (Key -> server mapping)
- similar to DHT without finger table or routing
- client sends the queries to a server known as Coordinator
- key -> server mapping is the *"Partitioner"*

#### Data Placement Strategies
1. **Simple strategy**
	- uses the partitioner
		- *random partitioner* : chord like hash partitioning
		- *ByteOrderedPartitioner*: assigns ranges of keys to servers
			- easier for *range queries* (e.g. get me all twitter users starting from ..)
			- if you use hash based, then you may need to ask every server
2. **Network Topology Strategy**
	- for multi-DC deployments
	- 2 replicas per DC
	- 3 replicas per DC
	- per DC
		- first replica placed according to partitioner
		- then go clockwise around ring until you hit different rack

#### Snitches
- to map IP to racks and DCs
- configured in `cassandara.yaml`
- Some Options
	- **SimpleSnitch** unware of topology (Rack-Unaware)
	- **RackInferring** assumes topology of network by octet of server's IP address
		- `101.102.103.104` = `x.<DC Octet>.<rack octet>.<node octet>`
		- **PropertyFileSnitch** uses a config file
		- **EC2Snitch** uses EC2
			- EC2 Region = DC
			- Availaility Zone = rack
	- more options are available

#### Writes

- need to be lock free and fast
- client sends write to one coordinator node in Cassandara cluster
	- Coordinator may be per-key, per-client, or per-query
	- per-key coordinator ensures writes for the key are serialised
- Coordinator uses Partitioner to send query to all replica nodes responsible for key
- when X replicas repond, coordinator returns an ack to the client
	- X?  *for now assume it to be a small no.*
- Keys should always be writable: (**Hinted Handoff Mechanism**)
	- if any replica is down, the coordinator writes to all other replicas, and keeps the write locally until down replica comes back up
	- when all replicas are down, the Coordinator(front end) buffers writes (for upto few hours)
- One ring per Dc
	- perDC coordinator elected to coordinate with other DCs
	- Election done via ***ZooKeeper***, which runs a ***PAXOS(consensus)*** variant
	- Zookeper is open source Apache server system


#### Writes at the Replica node
On receiving a write

- Log it in disk commit log (for failure recovery)
- Make changes to appropriate memtables
	- **Memtables** = in memory representation of multiple key-value pairs
	- Cache that can be searched by key
	- Write-back cache as opposed to write-through


Later, when the memtables is full or old, flush to disk

- **Data File** an SSTable (Sorted String Table) -- list of key-value pairs, sorted by key
- **Index File** an SSTable of (Key, position in data sstable)pairs
	- binary search here leads to a large overhead
	- for this bloom filter is used
- and a Bloom filter (for efficient search) - next Slide

#### Bloom Filter
- compact way of representing a set of items
- one of the operations could be to check whether a certain key is there or not
- other operations could be inserting in the set of keys
- some prob of False Positive: an item not in set may check true as being in set
- never False negative

![images04_01_03.png](/images/images04_01_03.png)

- it a very large bit map (128 bits shown) intially all 0
- Hash 1-k is used on Key K
- 

## bloom filter still a doubt


#### Compaction

- Data updates accumulate over time and SStables and logs need to be compacted
	- The process of compaction merges SSTables, i.e., by merging updates for a key
	- run periodically and locally at each server


#### Delete
- dont delete 
- instead mark **tombstone** to the log
- eventually, when compaction encounter s tomstone it will delete the item

#### Reads
- Similar to writes
- Coordinator can contact X replicas
	- coordinator sends read to replicas that hace responded quickest in past
	- when X replicas respond, coordinator returns the latest-timestamped value from among those X
	- X? 

![images04_01_04.png](/images/images04_01_04.png)

#### MemberShip
- any server in cluster could be the coordinator
- so every server needs to maintain a list of all the other servers that are currently in the server
- List needs to be updated automatically as servers join, leave, and fail

![images04_01_05.png](/images/images04_01_05.png)

#### Suspicion Mechanism
- to adaptively set the timeout based on undelying network and failure behaviour
- accrual detector:
	- failure detector outputs a value (PHI) representing suspicion
	- Apps set an appropriate threshold
	- PHI calculation for a member
		- Inter-arrival times for Gossip messages
		- PHI(t) = `-log(CDF or Prob(t_now - t_last))/log 10`
	-  PHI basically determines the detection timeout, but takes into account historical inter-arrival time variations for gossiped heartbeats
- in Practice, PHI = 5 => 10-15sec detection time


#### Cassandara vs RDBMS

|  			| Writes	| Reads		| 
| -- | -- | -- |
| MySQL		| 300ms avg	| 350ms avg	|
| Cassandara| 0.12ms avg| 15ms avg	|

----

### The Mystery of X - The Cap Theorem

- 3 very imp properties for any Distributed system , particularly storage system
	- **Consistency** all nodes see same data at any time
	- **Availability** the system operations all the time
	- **Partition-Tolerance** the system continues to work inspite of network partitions
- ***the CAP theorem says you cannot guarantee all 3 but only 2 out of these.***
- 500ms increase in latency for operations at Amazon.com and Google.com can cause 20% drop in revenue
- At Amazon each milisecond of latency implies a $6M yearly loss
- Service Level Agreements mainly deal with the latencies faced by the customers

#### CAP Theorem Fallout
- today's cloud needs partition tolerance, so a system needs to choose between the consistency and availability
- *Cassandra:* Eventual (weak) consistency, availability, partition-tolerance
- *Traditional RDBMSs:* Strong consistency over availability under a partition

#### CAP TradeOff
- Starting point for NoSQL Revolution
- A distributed storage system can achieve at most two of C, A and P
- C or A when P is required
- RDBMSs generally are stored on a single system so dont require partition tolerance
- HBase, Hyper Table etc. dont require availability
- 


#### Eventual Consistency
- if all the writes stop (to a key), then all its values(replicas) will converge eventually to the latest write
- if writes continue, the system always tries to keep converging.
	- *i.e. it keeps trying to catch up with the updated values and can't figure out the latest values by the clients*
- in case of many back to back writes, it may still return a stale value to the query sender
- works well when the no. of writes is low

> Eventual consistency says:

> - [ ] All values for a given key will eventually be different.
> - [ ] All values for a given key will be the same within a time bound.
> - [x] All values for a given key will eventually be the same after writes have stopped.

#### RDBMSs vs Key value stores
- RDBMSs provid **ACID**
	- Atomicity
	- Consistency
	- Isolation
	- Durability
- Key-Value provides 
	- **BASE**
		- Basically 
		- Available
		- Soft-state (*e.g. memtable*)
		- Eventual
	- Prefers availability over consistency


#### Mystery of X
- Cassandara has the consistency levels(X) for each operations read/write
- **ANY** any server (may not be replica)
	- fastest: coordinator caches write and replies quickly to client
- **ALL** all replicas
	- ensures strong consistency, but slowest
- **ONE** at least one replica
	- Faster than ALL , but cannot tolerate failure
- **Quorum** across all replicas in all DCs
	- global consistency but still fast
- **Local_qurom** quorum in coordinator's DC
	- faster: only waits for quorum in first DC client contacts
- **Each_Quorum** quorum in every DC
	- lets each DC do its own quorum: supports hierarchical replies


#### QUORUM
- refers to >50% (ie. majority)
- any 2 quoroms intersects
	- client 1 does a write 
- important as it brings strong consistency guarantee same as ALL
- several key-value/NoSQL stores(e.g. Riak and Cassandara) use quorums
- Reads
	- clients specifies value of R (<= N, *total no. of replicas of that key*)
	- R = read consistency level
	- coordinator waits for R replicas to respond before sending result to client
	- in bgd, coordinator checks for consistency of remaining (N-R) replicas, and initiates read repair if needed
- WRITES
	- client specifies W (<= N)
	- W = write consistency levels
	- clients writes new valurs to W replicas and returns 2 flavors
		- Coordinator blocks until quorum is reached
		- Asynchronous: Just write and return




- 2 necessary conditions for the strong consistency
	- `W+R > N`
	-  `W > N/2`
- select values based on applications

| W | R | when needed|
|--|--|--|
| 1 	| 1		| very few writes and reads|
| N 	| 1 	| great for read-heavy workloads|
|N/2 + 1|N/2 + 1| great for write heavy workloads|
| 1 	| n 	| great for write heavy workloads <br> with mostly one client writing per key|



----

### The Consistency Spectrum

<table>
<caption> The Consistency Spectrum </caption>
<tr>
	<td rowspan=3> Eventual Consistency </td>
	<td> <<<<< Faster Reads and writes --- </td>
	<td rowspan=3> Strong Consistency </td>	
</tr>
<tr>
    <td>  </td> 
</tr>
<tr>
	<td> ---- More Consistency >>>>> </td>
</tr>
</table>

- Cassandra offers eventual consistency
- Amazon's Dynamo or LinkedIn's Voldemort Systems uses last writers policy (LWW)
- DYNAMO DB system inspired cassandara and LinkedIn's system
-  Now new consistency models are being researched on

#### Newer Consistency Models

![images04_01_06.png](/images/images04_01_06.png)

- **Per-Key sequential:** Per Key, all operations have a global order
- **CRDTs (Commutative Replicated Data Types):** Data structures for which communtated writes give same results [INRIA, France]
	- commutating writes means, if you reverse the orders of the writes the writes would be the same
	- e.g. value == int, and only op allowed is +1
	- Effectively, servers dont need to worry about consistency
-  **Red blue** rewrite client txns to separate ops into red ops vs blue ops [MPI-SWS Germany]
	- there are some order which are needed to be ordered.
	- Blue ops can be executed (commutated) in any order across DCs
	- Red ops need to be executed in the same order
- **Causal Consistency:** 


#### Strong consistency models
- **Linearizability:**
	- each ops by a client is visible instantaneously (real time) to all other clients
	- i.e. only one copy of key-value pair and is stored in a server
	- difficult to support in distributed systems
- **Sequential Consistency:**
	- result of any execution is th same as if the ops of all the processors ere executed in some sequential order, and the ops of each ind. processor appears in this seq. in the order specified by the program
	- not as strong as linearizability
- TXM ACID properties are supported by newer sequential systems
	- Hyperdex [cornell]
	- Spanner [Google]
	- TXn chains [MSR]



----

### HBase

- Google's BigTable was the first "Blob-based" storage
- Yahoo open sourced it -> HBase
- API fxn
	- Get/Put row
	- Scan(Row range, filter - range queries)
	- MultiPut (multi key value pairs)
- Unlike Cassandra, HBase prefers consistency (over availability)


#### HBase Arch

![images04_01_07.png](/images/images04_01_07.png)

- client: sends queries for read/write
- zookeeper: runs consensus
- HRegionServers: 
	- associated with an HLog
	- contains multi HRegions
		- Multiple Stores
			- contains MEMstores
			- Multiple StoreFiles
				- a HFile
- HFile:  file stores in underlying HDFS file, same hadoop system
- HMaster: coordinates with zookeeper, and other parts

#### HBase Table Hierarchy

- HBase Table
	- split into **regions**: replicated across servers
		- ColumnFamily = subset of columns with similar query
		- One **store** per combination of ColumnFamily  + region
			- **Memstore** for each Store: in-memory updates to Store, flushed to disk when full
				- **StoreFiles** for each store for each region: where the data lives
					- **HFile**
- HFile 
	- SSTable frm Google's Bigtable


> Which of the following HBase components is stored in memory?
> - [ ] HFile
> - [ ]HRegionServer
> - [x] MemStore
> - [ ]Store

#### HFile

![images04_01_08.png](/images/images04_01_08.png)

- Data 
	- Magic no: to id uniquely
	- varities of **key value pair**, entire entries are known as HBase keys
		- key length
		- value length
		- row length 
		- row: for entire app it is social security no. 
		- col family length
		- col family: Demographic information
		- col qualifier: Ethicity
		- timestamp
		- key type
		- value

#### Strong Consistency: HBase Write-Ahead Log

![images04_01_09.png](/images/images04_01_09.png)

- Writer head log/ HLog is used
- different keys are asigned to diff regions
- regions first writes the entry to HLog, then to the Memstore
- if memstore is full, it is flushed to the Store file called HFile


#### LOG Replay

- on failure or upon bootup the log is replayed
- use timestamps to find out where the db is wrt the logs
- on replays edits are made to memstore


#### Cross-Datacenter Replication
- single MASTER dc
- other SLAVE dc replicate the same tables
- MASTER cluster synchronously sends HLogs over to slave DCs
- Coordination among clusters is via ZOOKEEPER
- ZOOKEEPER can be used like a file system to store control info


1. `/hbase/replication/state`
2. `/hbase/replication/peers/<peer cluster no>`
3. `/hbase/replication/rs/<hlog>`


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA3NzU1NzAwMiwtMTg5NjQ4MjE0NSw2OD
cyNjAwMzEsMTIxNTY1MzcxNCw5MjQ2MTcwMjYsLTE0Mjk5MTYw
MzUsNDAxMjQ0NTIsLTg3MTQwNTg3NSw2MDM5NTIzMTEsMTgwMj
gzMDI0Nyw5MDcxNjI0NTIsLTE0NDAxNzU4MDgsMTAxNDQ2MjUx
MCwxNDY1NDcyNzg5LDU3NzAxNDg5LC0xNjAzODQ5MjU4LDQxOD
E5MjQ4LC05OTcxNzE1NzYsLTg0NDUwNTQ5NiwzNTQzMDA0MDBd
fQ==
-->