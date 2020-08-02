Cloud Computing I - Week01 - 3 - MapReduce
---

### 1. MapReduce Paradigm

#### What is MapReduce?
- `(map square '(1 2 3 4))`
	- output: `(1 4 9 16)`
	- processes each record sequentially and independently
-  `(reduce + '(1 4 9 16))`
	- `(+ 16 (+ 9 (+ 4 1)))`
	- output: `30`
	- processes set of all records in batches

#### Map
- *process individual records to generate intermediate *key/value pairs**
- *e.g.* `Welcome Everyone Hello Everyone`

| **Key** | **Value** |
|---------|-----------|
| Welcome | 	1 	  |
| Everyone| 	1     |
| Hello   | 	1     |
| Everyone| 	1     |
- `Input <filename, file text>`
-  this can be *parallised* easily, ex. multiple Map tasks can be run together
- **still the final word count is not known,** here Reduce comes into use

#### Reduce
| **Key** | **Value** |
|---------|-----------|
| Welcome | 	1 	  |
| Everyone| 	2     |
| Hello   | 	1     |
- Reduce doesnt work in parallel but the keys are grouped
- keys are grouped
- Each key assigned to one Reduce
- Parallely process and all the reduce that will give the no of each keys
- hash is used for load balancing
- 


### 2. MapReduce Examples

#### Applications - 1 (Distributed Grep)

- IP : Large set of files
- OP: Lines that match pattern
	- map: *emits a line if it matches the supplied patter*
	- reduce: copies the intermediate data to OP
	 
#### Applications - 2  (Reverse weblink graph)
- you have a graph or a portion of it
- IP: Web Graphs: `tuples(a,b)`, where page a => page b
- OP: for each web page, list of pages that link to it
	- map: process web log and for each input `<source, target>`, it outputs `<target, source>`
	- reduce: emits `<target, list(source)>`
	- *i.e. the list all the pages targeting to link*

#### Applications - 3  (Count of URL Access freq)
- IP: log of accessed of URLs, e.g, from the proxy server
- OP: 


#### Applications - 4 (Sort)



### 3. MapReduce Scheduling


#### Programming Map reduce
- **Externally : for USER**
	- write a `map` program and a `reduce` program
	- Submit Job: wait for result
	- Nothing to be known about prallell/distributed programming
- **Internally:** for the Paradigm and Scheduler 
	- Parallelise map
	- Transfer data from Map to Reduce
	- Parallelise Reduce
	- Implement storage for Map inputs, map outputs, reduce inpur, and reduce output
	- *let us assume Reduce doesn't start before all the map tasks are finished*
	- 

#### Inside MapReduce
- Parallelize Map: (*easy*) 
	- each map task is different from another
	- all map output records with same key assigned to same reduce 
- transfer data from map to reduce:
	- all map outputs records with same key assigned to same reduce task
	- use partitionaing function *e.g* `hash(key)%number of reducers`
- Parallelize Reduce: (*easy*)
	- each reduce task is independent from other
- Implement storage from Map input, Map Output, Reduce Input, and Reduce Output
	- Map IP: from *distributed file system*
	- Map OP: to local disk (*at Map node*)
		- uses local file system
	- Reduce Input: from (*multiple*) remote disks
		- uses local file systems
	- Reduce OP: to distributed file system
- *Local file systems:* Linux FS etc.
- *Distributed File Systems:* GFS, HDFS etc.
- 

#### Internal working of MapReduce
- I have 7 box stored in DFS

| | map tasks| Servers| Shuffle Traffic  | outputs are sent to Reduce tasks| Output fles into DFS |
|-- |----| --| -- | -- | -- |
|1|--M1--| A | are| A | I |
|2|--M2--| A | written | B | II | 
|3|--M3--| B | locally | B | II |
|4|--M4--| B | at | C | III |
|5|--M5--| B | these| A | I |
|6|--M6--| C | servers &| B| II |
|7|--M7--| C | fetched to remote| C | III |

*Resource Manager assigns maps and reduce to servers*

#### YARN scheduler (*Yet Another Resource Negotiator*)
- resource negotiator of HADOOP
- *treats each server as the collection of the containers*
	- `each container = some CPU + some Memory`
- has 3 components
	- **Global Resource Manager** (RM)
		- Scheduling
	- **1 Node Manager per-server** (NM)
		- daemon and server-specific function
		- monitors the failures of tasks
	- **1 Application Master per application**
		- container negotiation with RM and NMs
		- also communicates with NMs for failure

#### YARN: how a job gets a container

![images03_01](/images/images03_01.png)

*in this fig. 1) 2 servers 2) 2 jobs*



### 4. Map Reduce Fault Tolerance

<br>

#### Fault Tolerance

- **Server Failure**
	- NM sends the heartbeats to RM
		- *if server fails, RM lets all affected AMs know, which then have the responsibility of rescheduling the tasks*
	- NM keeps track of each task runnina at its server
		- *if fails while in progress, mark the task as idle and restart is*
	- AM heartbeats to RM
		- *on failure, RM restarts AM, which then syncs up with its running tasks*
- **RM Failure**
	- *use old checkpoints and bring up secondary RM*
- Heartbeats also used piggyback container requests

<br>

#### Slow Servers

- **Stragglers**
	- *slow machine, more error server, bad disk, network bandwidth etc. could be the reason*
	- Perform backup execution of straggler task
		- Task is considered done when first replica is complete. Called *Speculative Execution*

####  Locality
- GFS/HDFS stores 3 replica of each of chunks (*e.g. maybe on different racks*)
- MapReduce attempts to shchedule a map
	- *a machine that contains a replica of corresponding ip data*, failing that
	- *on the same racks as a machine containing the ip*, failing that
	- *anywhere*


