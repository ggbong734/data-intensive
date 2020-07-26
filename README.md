_Reading notes | Completed in September 2020_

# Notes on Designing Data Intensive Applications
## [Book](https://dataintensive.net/) written by [Dr. Martin Kleppmann](http://martin.kleppmann.com/) 

Summary: Broad overview of the architecture of data systems and how to choose database for specific applications.

## Part 1 Fundamental ideas
1. Chapter 1 - Terminology
2. Chapter 2 - Data models and Query languages
3. Chapter 3 - Storage and Retrieval
4. Chapter 4 - Data Encoding

## Part 2 Distributed storage
5. Chapter 5 - Replication
6. Chapter 6 - Partitioning/Sharding
7. Chapter 7 - Transactions
8. Chapter 8 - Problems with Distributed systems
9. Chapter 9 - Consistency in Distributed system

## Part 3 Derived data
10. Chapter 10 - Batch processing
11. Chapter 11 - Stream processing
12. Chapter 12 - Building reliable, scalable, maintainable applications

### Chapter 1 - Terminology

**Reliability**
>  system continues to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software faults, and even human error).

**Scalability**
> As system grows in data volume, traffic volume, or complexity, there should be reasonable ways of dealing with that growth.

**Maintainability**
> Different people can work on the system (maintain current behavior or adapt system to new use cases) productively.

Fault
> One component of the system deviating from spec.

Failure
> System as a whole stops providing the required service to the user.

> It is impossible to reduce the probability of a fault to zero. Best to design fault-tolerance mechanisms to prevent faults from causing failures.
> > It can make sense to deliberately induce faults to ensure fault-tolerance machinery is continually tested. 

#### Hardware faults

Hardware faults are common but occur randomly and independently. The mean time to failure of a hard disk is 10-50 years. On a storage cluster with 10,000 disks, on average one disk dies per day. One approach is to **add redundancy** (i.e. add backup machines). Another newer approach is **software fault-tolerance techniques**.

#### Software faults

Software errors are systematic, they tend to affect more nodes as nodes are correlated.No quick solution, test thoroughly and continuously, isolate processes, measure, monitor, and analyze system behavior in production!

#### Human errors

Humans are known to be unreliable and create more faults than hardware. 
- Decouple places where people make mistakes from places where they can cause failures. Provide fully featured non-production sandbox environments.
- Test thoroughly: unit test, system integration, manual tests.
- Allow quick recovery from human errors. Example, roll back configuration.
- **Set up detailed and clear monitoring with metrics and error rates.**

#### Load parameters
- to describe load on the system
- maybe requests per second to a server, ratio of reads to writes in a database, number of active users, hit rate on a cache.
- Twitter example, when **rate of writes is much lower than reads**, it's preferable to do more work at write time and less at read time. Store the tweet at follower's cache during write, rather than join tables when reading. 
    - Look at **distribution of followers per user** to determine to whether to shift work during write or read. 
    - If a user has many followers, best not to burden during write as it can take a while before the tweet gets sent out.

- **Throughput** - number of records we can process per second. 
- **Service's response time** - time between a client sending a request and receiving a response.
- **Latency** - duration that a request is waiting to be handled. 
- When checking response time, instead of just looking at **mean**, study the tail latencies/outliers at 95, 99, 99.9 th percentile over a rolling window of say 30 minutes. These are the slowest responses that could break a user's experience.


-When generating load artificially to test scalibility, need to send requests independently of response time and don't wait for the system to complete the previous request. 
-scaling up (vertical) moving to a more powerful machine 
- scaling out (horizontal) distributing load across multiple smaller machines. Also called *shared-nothing* architecture.
-**elastic** automatically add computing resources when they detect a load increase. Good for unpredictable loads.

#### Maintainability

It's well known that majority of software cost is not initial development but ongoing maintenance. 
- *Operability* - easy for operations team to keep system running smoothly. Have good visibility into the system's health.
- *Simplicity* - easy for new engineers to understand. Use good *abstractions* using well-defined reusable components.
- *Evolvability* - easy for engineers to make changes

### Chapter 2 - Data Models and Query Languages

Applications are built by layering one data model on top of another. 

Data models can be JSON, XML documents, tables in relational database, or graph model.

#### Relational versus Document data model

Driving forces behind adoption of NoSQL (not only SQL) databases
- need for greater scalability (for very large dataset or large write throughput)
- specialized query operations
- restrictiveness of relational schemas


Document-oriented databases: MongoDB. Storing data as JSON document has better locality that multi-table schema as there's no need to run multiple queries. All the relevant information is in one place. JSON document has a tree structure. 

**Normalization** 
> a concept to remove duplicates to ensure consistency in database but requires many-to-one relationships (one tabel refers to many others). Denormalization means to create duplicate of the data (e.g. fact dimension table, star schema)

**Hierarchical model**
> tree structure and each record has exactly one parent

**Network model**
> a record could have multiple parents. Pointers link between records. To access a record, we need to follow a path from the root along these links. 

**Relational model**
> by contrast to the other two, this data model lays out the data in the open, easily accessible by joining tables.

The main arguments in favor of the document data model are schema flexibility, better performance due to locality, and that for some applications it is closer to the data structures used by the application. The relational model counters by providing better support for joins, and many-to-one and many-to-many relationships.


If your application does use many-to-many relationships, the document model becomes less appealing. For highly interconnected data, the document model is awkward while the relational model is acceptable and graph models are the most natural. Relational databases now support JSON documents.


Document model usually do not enforce any schema on the data in the documents. It is enforced during read, **schema-on-read**, as opposed to **schema-on-write** for relational databases where schema is explicitly defined when writing new data. Schema-on-read is advantageous when there are many different types of objects with different structures.

Schema changes using the `ALTER TABLE` function in SQL only takes milliseconds and does not deserve its bad reputation. 

Running `UPDATE` on a large database can be slow, however, as every row needs to be rewritten.

In the docuent model, database needs to load an entire document, which can be wasteful if only a small portion is needed. On updates, the entire document is rewritten. Thus, it is recommended to keep document fairly small.

#### Declarative versus Imperative language

**declarative query language**
> like SQL, we specify pattern of the data we want, but not how to achieve the goal. Implementation details are hidden.
>  - Able to be parallelized easily.
>  - in databases, declarative language like SQL is much better than imperative query APIs.
>  - in web browser, declarative CSS is better for manipulating styles than imperatively in JavaScript.

**imperative query language**
> tells the computer to perform operations in a certain order
> very hard to parallelize because it specifies instructions that must be performed in a particular oder

#### MapReduce querying

MapReduce is a programming mode for processing large amounts of data across many machines. It is between declarative and imperative languge.

MapReduce is fairly low-level programming model. SQL can be implemented as a series of MapReduce operations but it is not the only way to perform distributed query.

As it is often difficult to program map-reduce operations, new declarative language has been added to document databases such as MongoDB to allow SQL like expressions to query data. 

#### Graph data model

A graph model is suitable for data with many-to-many relationships. Consists of *vertices/nodes* and *edges*.

**property graph model**
> Each vertex has a unique ID, a set of outgoing and incoming edges and collection of properties (key-value pairs)
   - each edge has a unique ID, tail vertex, head vertex, label to describe the relationship between the two vertices.
   - can be represented with a relational schema
   - example language to query graph model: *SQL*, *Cypher*


**triple-store model**
> All information stored in the form of three-part statements: subject, predicate, object. E.g. Jim, likes, bananas.
   - The subject is a vertex/node
   - The object is either a property of the node or another vertex. In that case, the predicated is the label of the edge.
   - Example languages to write Triplets is *Turtle*, *XML*, or *SPARQL*.

Graphs are good for evolvability/extendability as we can add features to our application to acoommodare changes in the application data structure.

Graph data in a relational structure can be queried using SQL but not as easy, may need to traverse a variable number of edges before the node we are looking for is found. thus the number of joins is not fixed in advance. In SQL, this can be done using *recursive common table expressions* (`WITH RECURSIVE` syntax) 

### Chapter 3 - Storage and Retrieval

There is a big difference between storage engines that are optimized for transactional workloads and those that are optimized for analytics.

**log-structured storage engines**
- *log* is an append only data file
- Concurrency and crash recovery are much simpler if log files are append-only or immutable.

**page-oriented storage engines**
- such as B-trees.  

**index**
- additional data structure to speed up read queries
- but every index slows down writes as the index needs to be updated with new data

**hash map**
- one implementation of index to store key value pair. the location of the value is stored in the map. 
- Limitations:
    - hash map must fit in memory. difficult to implement on disk, can't work for very large number of keys. 
    - range queries are not efficient. Cannot easily scan over all keys between k000 - k 1000 without looking up every key.

**compaction**
- throwing away duplicate keys in the log, keeping only the most recent update for each key.

Appending and segment merging are sequential write operations, which are generally much faster than random writes, especially on magnetic spinning-disk hard drives. 

**Sorted String Table (SSTable) and LSM-trees**
- another implementation of data index also called Log Structured Merge-Tree (LSMTree)
- SStables have several advantages over hash indexes:
   - merging segments is simple and efficient.
   - because keys are sorted, no need to keep index for all keys. E.g. to find `hard` , search between `hand` and `hook`. I.e. a sparse index can work.
   - Able to compress records into groups before writing to disk, saving disk space and reducing I/O bandwidth. Each entry in the index points at the start of the compressed block.

SStable can be created using an in-memory balanced tree data structure (e.g. red-black or AVL trees) which allows us to insert keys in any order and read them back in sorted order. This is also called a *memtable*.  If the memtable gets too big, it can be written out to disk. Merging and compaction operation can be performed in background to combine segment files and discard deleted values. 


 - To avoid data losses when metable is not yet written to disk, a separate log on disk to which every write is immediately appended. The log only purpose is to restore the memtable after a crash. 

 - LSM-tree algorithm can be slow when looking up keys that do not exist in the database. We have to check the memtable, then segments all the way back to the oldest (from disk). A Bloom filter can be used to optimize such access. A Bloom filter is a data structure for approximating the contents of a set to save unnecessary disk reads for nonexistent keys.

 - basic idea of LSM-Trees is to keep a cascade of SSTables that are merged in the background.  They do not interefere with incoming queries in this regard.

**B-Trees**

The standard index implementation in almost all relational databases (and non-relational databases).

Like SSTables, B-Trees keep key-value pairs sorted by key, which allows efficient key-value lookups and range queries. 

But B-Trees are different Unlike SStables/log-structured indexes which break the database into variable-size segmeents (a few MB) and always write a segment sequentially. In contrast, B-trees break the database down into fixes-size pages (4 KB in size) and read or write one page at a time. 

Each page can be identified using an address or location, which allows one page to refer to another. The pointer is on disk instead of in memory. One page is designated as the root of the B-tree. Start at the root to look up a key in the index. The page contains several keys and references to child pages. 

**branching factor**
> The number of references to child pages in one page of the B-tree. Typically several hundred.


Adding a key requires getting to the right page and if not enough space, splitting the page into two half-full pages, while maintaining balance in the B-tree. Depth of B-Tree with n keys has a depth of log n. Most databases can fit into a B-tree that is 3 or 4 levels deep. A 4 level tree of 4 KB pages with a branching factor of 500 can store up to 256 TB.

- B-tree implement an additional write-ahead log (append-only) to restore the B-tree to a consistent state in case of a crash while splitting pages.

B-tree optimization is achieved in some cases by not doing an in-place overwrite for new writes, instead a new page is written at a different location. Also, keys can be abbreviated on the interior of the tree, not at leaf nodes. Another trick is to lay out the tree so that the leaf pages appear in sequential order on disk for easy range queries. This is difficult and easier to do with LSM-trees as large segments are rewritten during merging. Each page can also have a pointer to its sibling pages on the left and right. 

#### Comparison between B-Trees and LSM-Trees

As a rule of thumb, LSM Trees are faster for writes, whereas B-trees are usually faster for reads.  Reads are slower on LSM-trees because they have to check several different data-structures and SSTables at different stages of compaction. 

- A B-tree index must write data multiple times to the log and the page. LSM-Trees also have to write multiple times but they do sequential writes on disk which is fast. 

- LSM-Trees can be compressed better, since they periodically rewrite SSTables to remove fragmentation. Thus they have lower storage overheads compared to B-Tree index.

- However, compaction process can impact performance of LSM-Tree index. Sometimes requests may need to wait for the disk to finish a costly compaction process resulting in high response time at high percentiles. B-Trees are more predictable.

- LSM-Trees compaction also takes up large amount of disk bandwidth. If this is not enough unmerged segments may grow until disk space runs out, affecting reads and writes. 

- In B-Trees, key exists in one place in the index, whereas LSM-Tree might have multiple copies of the same key in different segments. Thus B-Trees are attractive in databases that want to offer transactional semantics. 

#### Storing values within the index

The value for a key in an index can be the actual row or a reference to the row stored elsewhere. The place where rows are stored can be a heap file. The heap file approach is common as it avoids duplicating data. Each index references a location in the heap file. 

Storing rows in index is called *clustered index* and are done when there is too much performance penalty for reads at a heap file location. 

A middle road is to store only some columns in the index, this approach is called covering index.

B-trees or LSM-trees are not able to address multi-index search efficiently. For example looking for restaurants within a given range of latitudes and longitudes are not possible to be done simultaneously. We need another data structure such as R-Tree. 


#### Transaction Analytics

**Transaction**
> Low latency reads and writes

**Online transaction processing (OLTP) vs. Online analytic processing (OLAP) **
- Read - small number per query vs. large number of records
- Write - low-latency writes vs. bulk import (ETL)
- Used by - End user vs. internal analyst
- Represents - latest data vs. history of events
- Stored in - regular database vs. data warehouse
- Internals of the two are different and often two separate products are needed

 **Data warehouse**
 - A separate database that analysts can query without affecting OLTP operations.
 - Contains read-only copy of data in the various OLTP systems in the company.
 - Data is extracted from OLTP databases (continuous or data dumps) vis ETL.
 - requires a different indexing algorithms

#### Stars and Snowflakes: Schemas for Analytics

Not many data models are used in analytics. One of them is star schema. Another one is snowflake schema.

**Star Schema**
- dimensional modeling. Uses a fact table and a set of dimension tables
- facts are usually individual event. other columns in the data tables are foreign key referencs to dimension tables
- dimensions represent the who, what, where, when, how, why of the event
- Fact tables are often much much larger than dimension tables.

**Snowflake schema**
- A variation of star schema where dimensions are further broken down into subdimensions
- snowflake schemas are more normalized than star schemas (less repeats) but star schemas are easier for analysts to work with.

#### Column-oriented Storage

Analytical queries often only require access to a few columns at a time.

Pulling just a few columns from row-oriented storage required loading of all the rows which can take a long time.

**Column-oriented storage**
- stores values from each column together instead of storing values from one row together.
- an example is the Parquet data format.
- easily compressed, e.g. using bitmap encoding as number of distinct values in a row are often small
- bitmap encoding represents each distinct value as a sequence of bits (ones and zeroes).
- sort keys are used for 1,2,3 columns to speed up search.

**Materialized view**
- defined like a standard (virtual) view: table-like object whose contents are the results of a query.
- actual copy of the query results written to disk, unlike virtual view which is a shortcut for writing queries.
- but materialized view needs to be updated if the underlying data changes, which can be expensive.
- a specialized case is data cube or OLAP cube. An aggregate table grouped by different dimensions.
- Advantage: fast queries
- Disadvantage: less flexibility than querying raw data

### Chapter 4 - Encoding and Evolution











