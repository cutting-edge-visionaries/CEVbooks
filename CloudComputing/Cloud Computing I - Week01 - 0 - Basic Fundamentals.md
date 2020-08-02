Cloud Computing I - Week01 - 1 - Basic Fundamentals
---

### Orientation Towards Cloud Computing Concepts: Some Basic Computer Science Fundamentals

##### I. Basic DS

1.  Queue
    -   head, tail, dequeue
2.  Stack
    -   Push, Pop, Top

##### II. Process : A Program in Action

-   basically a program in action
-   a process  **P1**  is given as

```
graph LR
A[main()] -- calls --> B[int f1()] --> C[int f2()]

```

-   a snapshot of a process contain
    -   code, program counter, current status of stack/heap & registers and few other elements

##### III. Computer Architecture

CPU

Registers

Cache

Memory

Disk

-   Accessing registers > cache >> Memory >> disk
-   Prog gets compiled to low level machine instructions.
-   CPU loads inst. in batches into memory
-   on execution CPU loads data into memory

##### IV. Big O() Notation

-   describes the upper bound on behaviour on algorithms, as some vars are scaled to infinity
-   "Worst case" perfomance
-   **rules:**
	- 1. Different steps get added
		- *for e.g. O(a) and O(b) are within same function. Then, for the function* `BigO = O(a+b)`
	- 2. Drop the Constants
		- *e.g. O(N) and O(N) are within same function. Then, for the function* `BigO = O(N)` *and not* `O(2N)`
	- 3. Differene i/p => Different Vars
		- *e.g. in O(N) doesn't mean the length of the array. What you want to describe is instead* `O(a*b)` *in case* array a *and* array b *are used*
	- 4. Drop non-dominate terms
		- *e.g. a function has O(N) and O(N^2), so here N^2 dominates, so, N ones is dropped*

##### V. Basic Probability
- set, subset, event, 

##### Vi. DNS
- name resolutions, IP address, CDN(content distribution network)

##### VII. Graphs
- network graphs, 
<!--stackedit_data:
eyJwcm9wZXJ0aWVzIjoiZXh0ZW5zaW9uczpcbiAgcHJlc2V0Oi
AnJ1xuIiwiaGlzdG9yeSI6WzEwMTYyOTkxOTldfQ==
-->