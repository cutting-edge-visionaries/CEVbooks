
Cloud Computing I - Week01 - 1 - Introduction to Clouds
--

    
#### 1. Why Clouds?
    
-   is expected to grow only exponentially
-   _e.g._  AWS : Amazon Web Services
    -   EC2 (Elastic Compute Cloud) : for computing services
    -   S3 (Simple Storage Service) : for storing data so that it could be accessed from anywhere
    -   EBS (Elastic Block Storage) : used by EC2 while running
-   2 categories:
    -   private cloud
    -   public cloud

#### 2. What is a Cloud?

-   it may be clusters, supercomputer, datastore, as per use.
-   _“Cloud = Lots of storage + Compute Cycles”_
-   a single-site cloud (“Datacenter”")
    -   - compute nodes (grouped into racks)
    -   switches, connecting the racks
    -   a network topology
    -   Storage nodes connected to the network
    -   front ends
    -   Software Services
-   a geographical distributed cloud consists of
    -   multiple such (single) sites
    -   each site can have different structures
-   **the topology**

<table align="center">
<tbody><tr>
	<td colspan="7"> **CORE SWITCH** </td>
</tr>
<tr>
	<td colspan="3"> Top of the Rack Switch </td>
	<td> ... </td>
	<td colspan="3"> Top of the Rack Switch</td>
</tr>
<tr>
	<td> servers</td>	<td> servers</td>	<td>servers </td>
	<td> ... </td>
	<td> servers </td> <td> servers </td> <td> servers </td>
</tr>
</tbody></table>


#### 3. Intro to Clouds: History

-   PCs led to creation of clusters
-   Data processing insdustries started around 1960s
-   Trends
    -   now CPU grows horizontally
    -   now, CERN’s Large Hadron Collider producing many PB/year

#### 4. What is new in Today’s cloud?
- 4 major chars:
    -   massive scale
    -   on demand accesss
    -   data Intensive nature
        -   _mapreduce/hadoop, NoSQL, Cassandra etc._
    -   New Cloud Programming Paradigm
-   Power comes from
    -   on site (solar panels)
    -   off site (dams)
-   WUE = Annual Water Usage / IT Equipment Energy
-   PUE = Total Facility Power / IT Equipment Energy (_google has the lowest_)

#### 5. new aspects of cloud

-   Hardware as a service (HaaS)
    
    -   you get h/w machines
-   Software as a service (SaaS)
    
-   Infra as a Services (IaaS)
    
    -   get access to flexible computing and storage infrasctrcture (_e.g. virtualisation_)
    -   AWS EC2 and S3
-   Platform as a Service (PaaS)
    
    -   flexible computing + storage + software platform
    -   get access to Software services
-   _**New Cloud Computing Paradigms**_
    
    -   google : mapreduce, sawzall
    -   amazon : elastic mapreduce service
    -   Yahoo : hadoop + pig
    -   FB : hadoop + hive

#### 6. Economics of the Cloud

-   outsource or own?
<!--stackedit_data:
eyJoaXN0b3J5IjpbNDY5OTE4MDY3XX0=
-->