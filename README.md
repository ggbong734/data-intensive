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

This chapter covers ways of turning data structures into bytes on network or bytes on disk. 

Code changes on server-side applications is performed through a rolling upgrade (staged rollout) where the new  version is deployed to a few nodes at a time. For client-side applications, the user has to install the update. 

**Backward compatibility**
> Newer code can read old data

**Forward compatibility**
> Older code can read new data

Data is represented in memoty for efficient access by CPU (using pointers). When data is sent to another system, it is encoded as a self-contained sequence of bytes (e.g. JSON documen)

**Encoding/serialization**
> The translation of data from in-memory representation to a byte sequence (e.g. JSON file).

**Parsing/deserialization**
> Process of converting a byte sequence data into an in-memory representation.

Problems with encoding and decoding data:
- Encoding is tied to a particular language. Reading data from another language is difficult.
- Languages have built-in support for encoding. Java has `java.io.Serializable`, Python has `pickle`.
- Data versioning and efficiency (CPU time) are often an afterthought. 
- Security issue during decoding as arbitrary classes need to be instantiated.

It's a bad idea to use a programming language built-in decoding. JSON and CSV are alternatives.

Problems with common data encoding formats:
- Hard to distinguish between string and numbers in XML and CSV. JSON doesn't distinguish integers and floats.
- JSON and XML don't support Unicode character strings. 
- Not many people create JSON and XML schemas
- CSV does not have any schema. It's very vague. May see issues if value contains comma or newline character.

**Binary encoding**
- For internal data transfer within an organization, binary formats of JSON may be suitable.
- More compact and faster to parse compared to JSON, CSV.
- Examples include MessagePack, BSON.

**Thrift and Protocol Buffers** are binary encoding libraries based on the same principle. 
- a tag number is assigned to each field instead of using the full field name in the encoding.
- When a field name is changed, we keep the field tag (id) and just changed the schema definition.
- To maintain forward compatibility, new field can be ignored by old code. 
- For backward compatibility, every field added after initial deployment of the schema must be optional or have a default value.
- Be careful if data type is changed say from int32 to int64, the old code may truncate new data as it doesn't fit the int 32. 


#### Avro

Another binary encoding format as a subproduct of Hadoop.
- Uses a schema to specify the structure of the data being encoded. Two schema languages provided.
- no tag numbers for each field, does not include datatype in the encoding
- more compact encoding compared to Thrift or ProtocolBuffers
- parsing the binary data involves going through each field in order they appear in the schema. Schema tells us the datatype.
- requires exact same schema used for writing the data when reading

Writer's schema (produced when writing file) and reader's schema (produced when decoding) don't have to be the same in Avro. they just have to be compatible. Avro will compare the two schemas side by side and resolve the difference. E.g. different order will be matched up by field name, new data will be ignored.


With Avro, forward compatibility means new schema = writer while older version of the schema = reader. Conversely, backward compatibility means new version of the schema as reader while old version is the writer. **User may only add or remove a field with a default value.**

Changing a field name is backward compatible (new code can read old data) but not forward compatible (old code can't read new field). Adding a branch to a union type is the same. 

**Schema versions**
- for a large file, the writer's schema can be included at the beginning of the file.
- for a database, the version number of the record can be included and when extracting data, the schema with that version number is referenced.
- for sending data over a network, the two processes need to negotiate the schema version.
- be sure to create a database of schema versions to keep track of schema compatibility.

**Dynamically generated schemas*
- Avro is better than Thrift or ProtocolBuffers for converting data into binary format when schema changes occur frequently.
- In Avro, a new schema can just be generated.
- In Thrift or ProtocolBuffer, a database administrator has to manually assign new field tags to changed fields

#### Dataflow through Databases

If a new field is added to a schema using newer code, and an older code reads the record, and updates it, and writes, it back. the ideal behavior is for the old code to keep the new field intact.

Unknown fields may be lost if a database value is decoded into model objects and later reencoded back into the database. 

When adding a new column to a database, may want to use `null` default value, such that the database fills in `null` in the new fields when reading old data.

When creating a backup copy of the database for archival, the dump is typically encoded using the latest schema. Avro object container files are a good format. Parquet column-oriented format is another good approach.

#### Dataflow through Services: REST and RPC

The server creates an API over the network and clients can connect to servers to make requests to the API. API exposed by the server is known as a service. 

An application where a server acts as a client to request data and act as a client to another server is called service-oriented architecture. 

Services are similar to databases that they allow clients to submit and query data. But the API that the service expose is specific that it only allows a set of predetermined inputs and outputs. Thus services and API can impose restrictions on what clients can and cannot do.

It is good to have each service be run by a separate team such that they can be update independently. Old and new versions of servers and clients can be expected to run at the same time so the data encoding used by servers and clients must be compatible across versions.


When the protocol for talking to a service uses HTTP, it is called a web service. But not only limited to the web. For e.g. mobile device app requesting to a service via HTTP over public internet. One service making requests to another service owned by the same organization. One service making requests to a service owned by a different organization. 


There are two approaches to **web services**.

**REST**
- not a protocol but a design philosophy
- emphasizes simple data formats
- uses URL for identifying resources
- uses HTTP features for cache control, authentication, content type negotiation. 
- getting more popular than SOAP.
- an API designed according to REST principles is called RESTful.

**SOAP**
- XML-based protocol for making network API requests
- aims to be independent from HTTP and avoids using most HTTP features. 
- APIS is described using Web Services Description Language (WDSL)
- SOAP messages are complex to construct and users have to rely on IDES and tool support. 
- integrating SOAP with languages not supported by SOAP vendors is difficult.

**Remote procedure call (RPC)**
- RPC has many issues and is fundamentally flawed.
- a network request is unpredictable, network problems are common.
- network request can result in a timeout, which makes it hard to diagnose what went wrong.
- a network request is slower than a function call, and its latency is highly variable
- client and service may be implemented in different programming language so RPC framework must translate.
- mainly ONLY used by services owned by the same organization.

RESTful APIs are the predominant style for public APIS because requests can be made simply with `curl` or web browser, no need for extra software, and is supported by mainstream programming languages and platform.

We can assume that servers will be updated first and then clients. Thus we only need backward compatibility on requests, and forward compatibility on responses. 

#### Message-Passing Dataflow

Asynchronous message-passing systems are between RPC and databases, a client's request is sent to another process just like RPC, but the message is not sent via direct network connection but goes to an intermediary message queue. 

Advantages of message-passing system:
- message queue act as buffer if recipient is not available
- avoids the sender needing to know the IP address and port number of recipient 
- allows one message to be sent to multiple recipients 
- decouples the sender from recipient (sender just publishes messages and doesn't care who consumes them)

Asynchronous means the sender doesn't wait for message to be delivered, it simply sends and forget about it)

**message broker/queue**
- Example is Apache Kafka
- used by a process sending a message to a queue/topic then the broker ensures delivery to one or more consumers/subscribers.
- a queue or topic provides only a one-way dataflow
- a consumer/subscriber can publish message to another topic (chaining)
- don't enforce a data model.

**actor model**
- programming model for concurrency in a single process. 
- encapsulates logic
- One actor represents one client/entity. Not shared with another actor.


## Part II: Distributed Data

In this part we ask what happens if multiple machines are involved in storage and retrieval of data. 

**Scalability**
> Ability to spread load across multiple machines when data volume/read load/write load grows too big for one machine

**Fault tolerance/high availability**
> If one machine goes down, multiple machines are present to give redundancy

**Latency**
> With users globally distributed, have servers at various locations to be close to users. 


**shared memory architecture**
> machines share CPU, RAMS, and disks. Also called scaling up (vertical scaling). 

**shared disk architecture**
> machines have independent CPUs and RAMS but store data on shared disks

**shared nothing architecture**
> Each node uses its own CPU, RAM, and disks. Also called scaling out (horizontal scaling).

Scaling out can distribute data across geographic regions, and reduce latency. Cloud deployments make this simpler. 

This book focuses on shared nothing architecture. Which has many complexities. 

### Chapter 5 - Replication

**Replication**
> Keeping a copy of the same data on several different nodes to provide redundancy.

**Partitioning**
> Splitting a big database into smaller subsets so that different partitions can be assigned to different nodes (a.k.a sharding). 

Three popular algorithm for replicating: single-leader, multi-leader, and leaderless application. 

The main tradeoffs to use synchronous or asynchronous (don't wait for process to complete), how to handle failed replicas. 

**leader-based replication (master-slave application)**
- One of the replica is designated as the leader/master/primary.
- Other replicas are followers/slaves/secondaries
- When writing, requests are sent to leaders who then send the replication log to followers.
- when a client wants to read data, it can query either leader or any of the followers.

**synchronous replication**
> the leader waits until the follower has confirmed it received the write before reporting success to the user
- follower is *guaranteed* to have the latest data like the leader
- but if the follower crashes or encounters faults, the write cannot be processed
- impractical to have many followers be synchronous

**asynchronous replication**
> leader sends the replication log to follower but doesn't wait for a response from follower.
- this ensures rapid writes but may also result in write failures even as the system reports a success
- One common used approach is semi-synchronous where 1+ node serves as synchronous follower.
- chain replication is another technique used to avoid losing data if the leader fails.

To copy data to a new follower, the follower copies from a snapshot of the leader database, then copies the data changes that appear in the leader's log since the snapshot was created to be in sync with the leader. 

#### Handling node outages

Maintaining replica consistency, durability, availability, latency are key problems in distributed systems.
To maintain high availability, our goal is to keep the whole system running even if nodes fail.

**Follower failure**
- The follower that crashes can refer to its log to check the last transaction that occurred before the fault.
- It then checks with the leader on the remaining transactions that occurred after the checkpoint

**Leader failure**
- One of the followers needs to be promoted to be the new leader
- Clients need to be configured to send writes to the new leader
- Other followers need to start consuming data changes from the new leader. 
- The process is called *failover*.
- If asynchronous replication, new leader may not be up to date with old leader. The unreplicated writes are discarded.
- Two nodes may both believe that they are the new leader.

#### Replication logs

How does leader-based replication work under the hood?

**statement based replication**
- leader logs every write request (statement) and sends the log to its follower
- problem may arise if nondeterministic function like `NOW()` and `RAND()` are used which produce different results in each replica.
- considered unsafe, not as popular right now

**write-ahead log (WAL) shipping**
- In the case of B-tree storage enginer, modifications are written to a write-ahead log so the index can be restored after a crash.
- the log is an append-only sequence of bytes containing all writes to the database.
- the disadvantage is that the log is written in very low level: which bytes were changed in disk blocks. Changes to storage format will require a change to the log. The leader and followers have to have the same database software version.
- followers use to this log to catch up with the leader. 

**logical log replication**
- different format for replication log (logical log) and for the storage engine. 
- the log has the granularity of a row. 
- for insert row, the log contains new values of all columns.
- for deleted row, the log contains the primary key of deleted rows 
- the logical log is decoupled from the storage engine internals, can be kept backward compatible, allowing leader and followers to run different versions of the software

#### Trigger based replication

**trigger** 
> lets you register custom application code that is automatically executed when a data change occurs in a database.
- has greater flexibility than database's built in replication
- but has greater overheads and is more prone to bugs and limitations than database's built in system.

#### Problems with Replication lag

Most web application require more reads than writes. To serve this, many followers can be created after the leader. For this type of read-scaling architecture, only asynchronous replication makes sense. But this creates an issue where the leader and follower may not have the same data due to **replication lag**.

**replication lag**
> difference between leader and follower in asynchronous replication due to writes not yet reflected in followers.
- lag may increase to several seconds or minutes in some cases.

Some techniques to resolve replication lag:
- when reading something the user may have modified, read it from the leader, otherwise read from follower.
- could also monitor replication lag on followers and prevent queries on follower that is more than x seconds behind the leader
- the system can be programmed to ensure replica serving any reads for that user reflects updates until that timestamp. 

When handling read writes requests from multiple devices, e.g. changing profile info on desktop and viewing on mobile phone, approches that uses timestamp of the last user's update become more difficult. The metadata will need to be centralized so that one device knows the updates that happened on the other device. Also, if replicas are in distributed across multiple regions, we may need to rout requests from all of user's devices to the same datacenter. 

**monotonic read**
 > guarantee that users who requests multiple reads in sequence, they don't see time moving backwards, especially for asynchronous followers.
- one way is to ensure each user always read from the same replica. 

**consistent prefix read**
> guarantees that if a sequence of writes occur in a certain order, anyone reading the writes will see them appear in the same order. 
- usually a problem in partitioned databases, different partitions operate independently, so no global ordering of writes. 

Always worth thinking about how the application may behave if the replication lag increases to several minutes or hours...

#### Multi-leader replication

We can have a leader in each datacenter, within each datacenter regular leader follower replication is used, between datacenters, each leader recplicates its changes to leaders in other datacenters. This provides improvement to performance (asynchronous replication is possible), tolerance of datacenter outages, tolerance to network problems.

One big downside is when the same data is being modified in two datacenters. Conflict resolution must be used. 

An example is the calendar app on phone. Multiple devices need to sync and each device acts as a datacenter. Real-time collaborative editing (google sheets) also utilize multi-leader approach but requires good conflict resolution.

#### Write conflicts

**Conflict**
> Two writes concurrently modify the same field in the same record, setting it to two different values. Another kind is when a booking app makes two reservation on the same room/seat at the same time. Conflict may arise if they are written on different leaders.

Simplest way is to avoid conflicts. This can be done by assigning each user/record into one datacenter for writing. Any changes or writes to the user will be routed to the same datacenter and use the leader in the datacenter for reading and writing. 

In multi-leader configuration, there's no defined ordering of writes, it's not clear what the final value should be. 
- One way is to give each write a unique ID (e.g. timestamp, incrementing number) and pick writes with highest value. But this approach implies data loss.
- Another way is to record all changes and let user resolve conflict. 

#### Multi-leader replication topologies

**replication topology**
> describes the communication paths along which writes are propagated from one node to another. 

Examples include circular topology, star topology, all to all topology.
- All-to-all: every leader sends writes to every other leader
- Circular: each node receives writes from on node and forwards those writes (plus its own writes) to on other node
- Star: One designated root node forwards writes to all other nodes. Similar to a tree

Fault tolerance of a more densely connected topology (like all-to-all) is better because it avoids a single point of failure. 

Another type of conflict is when a leader overtakes another leader in terms of writes due to network issues in the slow leader.  When using multi-leader replication, it is worth being aware of these issues.

#### Leaderless replication

No designated leader in this scheme so writes are concurrently sent to multiple replicas without order. If some replicas fail to write say for a system update, the write is allowed to fail at the replica. However, when reading, read requests are sent to several nodes in parallel. Version numbers are used to determine which value is newer. 

Repair on *stale* replica can be done during reads (read repair). If system detects one replica has old value, it can replace the value with the new one from newer replica. There could also be a background process (anti-entropy process) which continuously checks for differences between replica. Note that without anti-entropy process, data that are rarely read may be missing as values are only repaired during reads.

#### Quorums for reading and writing

For leaderless replication, if there are n replicas, every write must be confirmed by w nodes and we must query at least r node for each read. Leaderless replication is designed to tolerate conflicting concurrent writes, network interruptions, and latency spikes. 

As long as w + r > n, we expect to get an up-to-date value when reading because at least one of the r nodes we're reading from must be up to date. 

Reads and writes that obey these r and w values are called quorum reads and writes.

If fewer than the required w or r nodes are available, writes or reads return an error. 

In some cases, using a smaller w and r may be desirable because even though we are more likely to read stale values, as there is a chance that our read didn't include node with the latest value, this configuration allows lower latency and higher availability.

**Sloppy quorum**
> In the case of node failures, writes are written to nodes that are among the designated n "home" nodes for a value. By analogy, if we lock ourselves out of our house, we seek refuge at a neighbor's place. Once the home node recovers, 
- Sloppy quorum is useful for increasing write availability. We can write as long as w nodes are available, but the this means even if w + r > n, we cannot be sure that the latest value for a key is read. As the latest value may be outside the n "home" nodes.

#### Monitoring stateness

It's important to monitor whether database is returning up-to-date results. We need to be aware of the health of the application.
For leader-based replication, the database typically keeps track of the replication lag (by subtracting leader's position from follower's position, we can measure replication lag).

In systems with leaderless replication, there is no fixed in which writes are applied, which makes monitoring more difficult. Moreover, if the database only uses read repair (no anti-entropy), there is no limit to how old a value might be. 

#### Dealing with concurrent writes

**Last write wins**
> declare that each replica need only to store the most "recent value" and allow "older" values to be overwritten and discarded. Requires an unambiguous wasy to determine which write is more recent. If losing data is not acceptable, Last write winds is a poor choice for conflict resolution. 

When we have two operations A and B, there are three possibilities: either A happened before B, or B happened before A, or A and B are concurrent. What we need is an algorithm to tell us whether two operations are concurrent or not. 

**For defining concurrency, exact time doesn’t matter: we simply call two operations concurrent if they are both unaware of each other, regardless of the physical time at which they occurred.**

Algorithm for dealing with concurrent writes using version number (for a single replica) is provided in the text on page 266.

### Chapter 6 Partitioning

Partitions are defined such that each piece of data belongs to exactly one partition. The main reaason for partitioning data is scalability. Partitions can be placed on different nodes across many disks. 

Usually combined with replication such that each record (belonging to exactly one partition) is stored on several different nodes for fault tolerance. 

Our goal with partitioning is to spread the data and the query load evenly across nodes.
If the partitioning is unfair, so that some partitions have more data or queries than others, we call it skewed. The presence of skew makes partitioning much less effective. In an extreme case, all the load could end up on one partition,

**hot spot**
> A partition with disproportionately high load.

#### Partitioning by Key range

One way of partitioning is to assign a continuous range of keys. The ranges of keys are not necessarily evenly spaced, because your data may not be evenly distributed.

However, the downside of key range partitioning is that certain access patterns can lead to hot spots. If the key is a timestamp, then the partitions correspond to ranges of time — e.g., one partition per day. Unfortunately, because we write data from the sensors to the database as the measurements happen, all the writes end up going to the same partition. To avoid this problem in the sensor database, you need to use something other than the timestamp as the first element of the key. For example, you could prefix each timestamp with the sensor name so that the partitioning is first by sensor name and then by time.

#### Partitioning by Hash of Key

Once you have a suitable hash function for keys, you can assign each partition a range of hashes (rather than a range of keys), and every key whose hash falls within a partition’s range will be stored in that partition.This technique is good at distributing keys fairly among the partitions. The partition boundaries can be evenly spaced, or they can be chosen pseudorandomly.

Unfortunately however, by using the hash of the key for partitioning we lose a nice property of key-range partitioning: the ability to do efficient range queries. Keys that were once adjacent are now scattered across all the partitions, so their sort order is lost. Any range query has to be sent to all partitions.

For keys that are very hot (e.g. a Twitter account of a megastar), there will be a lot of read and write requests. One way to address this is to add a random number to the beginning and end of the key, essentially making 100 variations of this key. Now we can split the writes to the key evenly distributed across 100 different keys. But any read will have to do more work as it has to read all 100 keys from different partitions and combine the data.  

#### Partitioning and Secondary Indexes

The situation becomes more complicated if secondary indexes are involved (see also “Other Indexing Structures”). A secondary index usually doesn’t identify a record uniquely but rather is a way of searching for occurrences of a particular value.

**partitioning secondary indexes by document**
> In this indexing approach, each partition is completely separate: each partition maintains its own secondary indexes, covering only the documents in that partition.
- However, reading from a document-partitioned index requires care. If you want to search for e.g. all red cars, you need to send the query to all partitions. Thus it can make read queries on secondary indexes expensive. 

**partitioning secondary indexes by term**
> Rather than each partition having its own secondary index (a local index), we can construct a global index that covers data in all partitions. A global index must also be partitioned, but it can be partitioned differently from the primary key index.

The advantage of a global (term-partitioned) index over a document-partitioned index is that it can make reads more efficient: rather than doing scatter/ gather over all partitions, a client only needs to make a request to the partition containing the term that it wants. However, the downside of a global index is that writes are slower because a write to a single document may now affect multiple partitions of the index. (every term in the document might be on a different partition)

#### Rebalancing Partitions

The process of moving load from one node in the cluster to another is called rebalancing.
- After rebalancing, the load (data storage, read and write requests) should be shared fairly between the nodes in the cluster
- While rebalancing is happening, the database should continue accepting reads and writes.
- No more data than necessary should be moved between nodes, to make rebalancing fast and to minimize the network and disk I/ O load.

**fixed number of partitions**
- create many more partitions than there are nodes, and assign several partitions to each node.
- Only entire partitions are moved between nodes. The number of partitions does not change, nor does the assignment of keys to partitions. The only thing that changes is the assignment of partitions to nodes.
- the number of partitions configured at the outset is the maximum number of nodes you can have, so you need to choose it high enough to accommodate future growth. To not have to split partitions. 
- if partitions are very large, rebalancing and recovery from node failures become expensive. But if partitions are too small, they incur too much overhead.

**dynamic partitioning**
- When a partition grows to exceed a configured size (on HBase, the default is 10 GB), it is split into two partitions so that approximately half of the data ends up on each side of the split.
- if lots of data is deleted and a partition shrinks below some threshold, it can be merged with an adjacent partition.
- that the number of partitions adapts to the total data volume. If there is only a small amount of data, a small number of partitions is sufficient. 

**Partitioning proportionally to nodes**
- A third option, used by Cassandra and Ketama, is to make the number of partitions proportional to the number of nodes — in other words, to have a fixed number of partitions per node. 

It's better to have a human involved in the loop during rebalancing to avoid overloading nodes and creating a cascading failure. 

#### Request Routing

When a client makes a request, how does it know which node to connect to? which IP address and port number to connect to?

Approaches:
- Allow clients to contact any node. 
- sends requests from clients to a routing tier first which determines the node that should handle the request. 
- requires clients to be aware of partitioning and assignment of partitions to nodes. A client can connect directly to the appropriate node, without an intermediary. 

How does the component making the routing decision (which may be one of the nodes, or the routing tier, or the client) learn about changes in the assignment of partitions to nodes?
- Using Zookeeper to track of partition and node metadata.
- parallel query execution is also used for massively parallel processing databases. 

### Chapter 7 Transactions

A transaction is a way for an application to group several reads and writes into a logical unit.Transactions are an abstraction layer that allows an application to pretend that certain concurrency problems and certain kinds of hardware and software faults don’t exist. A large class of errors is reduced down to a simple transaction abort, and the application just needs to try again.

This way the system doesn't need to worry about partial failure. It will either succeed (commit) or fail (abort or rollback).

The safety guarantees provided by transactions are often described by the well-known acronym ACID, which stands for **Atomicity, Consistency, Isolation, and Durability**. 

One database’s implementation of ACID does not equal another’s implementation. For example, as we shall see, there is a lot of ambiguity around the meaning of isolation. The devil is in the details.

#### Atomicity

Atomic refers to something that cannot be broken down into smaller parts.

Rather, ACID atomicity describes what happens if a client wants to make several writes, but a fault occurs after some of the writes have been processed. If the writes are grouped together into an atomic transaction, the transaction cannot be completed due to a fault then it is aborted and any changes so far is discarded.

The ability to abort a transaction on error and have all writes from that transaction discarded is the defining feature of ACID atomicity. Perhaps abortability would have been a better term than atomicity.

#### Consistency

In the context of ACID, consistency refers to an application-specific notion of the database being in a “good state.”

The idea of ACID consistency is that you have certain statements about your data (invariants) that must always be true — for example, in an accounting system, credits and debits across all accounts must always be balanced.

This idea of consistency depends on the application’s notion of invariants, and it’s the application’s responsibility to define its transactions correctly so that they preserve consistency. This is not something that the database can guarantee. the letter C doesn’t really belong in ACID.

#### Isolation

Isolation in the sense of ACID means that concurrently executing transactions are isolated from each other: they cannot step on each other’s toes.

The classic database textbooks formalize isolation as serializability, which means that each transaction can pretend that it is the only transaction running on the entire database. The database ensures that when the transactions have committed, the result is the same as if they had run serially (one after another), even though in reality they may have run concurrently.

#### Durability

Durability is the promise that once a transaction has committed successfully, any data it has written will not be forgotten, even if there is a hardware fault or the database crashes.

It usually also involves a write-ahead log or similar (see “Making B-trees reliable”), which allows recovery in the event that the data structures on disk are corrupted. In a replicated database, durability may mean that the data has been successfully copied to some number of nodes.

#### Multi-object operations

Multi-object transactions require some way of determining which read and write operations belong to the same transaction. In relational databases, that is typically done based on the client’s TCP connection to the database server: on any particular connection, everything between a BEGIN TRANSACTION and a COMMIT statement is considered to be part of the same transaction.

Atomicity can be implemented using a log for crash recovery (see “Making B-trees reliable”), and isolation can be implemented using a lock on each object (allowing only one thread to access an object at any one time so no "dirty reads" or reading uncommitted writes).

#### Handling errors

ACID databases are based on this philosophy: if the database is in danger of violating its guarantee of atomicity, isolation, or durability, it would rather abandon the transaction entirely than allow it to remain half-finished.

Retrying an aborted transaction is not perfect:
- If the transaction actually succeeded, but the network failed while the server tried to acknowledge the successful commit to the client (so the client thinks it failed), then retrying the transaction causes it to be performed twice.
- If the error is due to overload, retrying the transaction will make the problem worse, not better.
- It is only worth retrying after transient errors (for example due to deadlock, isolation violation, temporary network interruptions, and failover).
- if email is scheduled when transaction is completed, multiple emails may be delivered.

#### Weak Isolation levels

**Serializable** isolation means that the database guarantees that transactions have the same effect as if they ran serially (i.e., one at a time, without any concurrency).

In practice, isolation is unfortunately not that simple. Serializable isolation has a performance cost, and many databases don’t want to pay that price [8]. It’s therefore common for systems to use weaker levels of isolation, which protect against some concurrency issues, but not all.

**Read Committed**
> The most basic level of transaction isolation is read committed. It makes two guarantees: When reading from the database, you will only see data that has been committed (no dirty reads). When writing to the database, you will only overwrite data that has been committed (no dirty writes).
- default isolation level in many databases
- does not prevent race conditions (concurrency issues) where two clients concurrently increment a counter but the counter only registered an increase of one instead of two.  

Databases **prevent dirty writes** by using row-level locks: when a transaction wants to modify a particular object (row or document), it must first acquire a lock on that object. It must then hold the lock until the transaction is committed or aborted. Only one transaction can hold the lock for any given object.

One way to prevent dirty reads is to lock the object when reading (prevent writes) then release the object once reading is done. But this is not good as even read-only jobs harms response time of other transactions. 

Databases **prevent dirty reads** by remembering the old committed value and the new value for every object that is being written. While the write is not committed, any read on that object is given the old value. 

#### Snapshot Isolation and Repeatable Read

Read committed isolation level may still be susceptible to *nonrepeatable reads* or *read skew*. This is a temporary inconsistency when values change when reads are done during and after the write transactions. However, situations like taking backups and performing analytic queries do not tolerate nonrepeatable reads. 

Databases **prevent nonrepeatable reads/read skew** using *snapshot isolation*.
> The idea is that each transaction reads from a consistent snapshot of the database - even if data is subsequently changed by another transaction, each transaction only see the old data from that particular point in time.

Implementations of snapshot isolation typically use write locks to prevent dirty writes.

However, reads do not require any locks. From a performance point of view, a key principle of snapshot isolation is readers never block writers, and writers never block readers.7-4. The database must potentially keep several different committed versions of an object, because various in-progress transactions may need to see the state of the database at different points in time. This technique is known as *multi-version concurrency control* (MVCC).

A typical approach is that read committed uses a separate snapshot for each query, while snapshot isolation uses the same snapshot for an entire transaction.

**visibility rules for observing a consistent snapshot**
- At the start of each transaction, the database makes a list of all the other transactions that are in progress (not yet committed or aborted) at that time. Any writes that those transactions have made are ignored, even if the transactions subsequently commit.
- Any writes made by aborted transactions are ignored.
- Any writes made by transactions with a later transaction ID (i.e., which started after the current transaction started) are ignored.

An object is only visible if:
- At the time when the reader’s transaction started, the transaction that created the object had already committed.
- The object is not marked for deletion, or if it is, the transaction that requested deletion had not yet committed at the time when the reader’s transaction started.

Snapshot isolation is also called **serializable** or **repeatable read** by different databases. Unfortunately the standard definition of repeable read is often ambiguous, inconsistent when applied by databases.

#### Preventing lost updates (write-write conflict)

Concurrent writes may result in one of the writes being overwritten by the second one as each transaction does a read-modify-write. The transactions read the same value but modify them to different values and do not consider the other one. 

Solutions:
- Atomic writes: avoid read-modify-write cycles by representing writes in atomic operations. Implemented by taking an exclusive lock on the object when it is read. 
- Explicit locking: lock the object until the first read-modify-write cycle is completed before the next one can occur. But easy to forget to add a lock in the code, thus introducing a race condition.
- Automatic deletion of lost updates: Allow transactions to occur in parallel and when a lost update is detected, the transaction (read-modify-write) cycle is retried. Less error prone than using explicit locking. 
- Compare and set: only allow an update to happen only if the value has not changed since we last read it. If there is a mismatch, the read-modify-write cycle must be retried. May not work if the database reads from a snapshot while another write is taking place. 
- In replicated databases, locks and compare-and-set may not be sufficient as there could be multiple up-to-date copy of a the data. Need to apply linearizability. A common approach in replicated databases is to allow concurrent writes to create several conflicting versions of a value (siblings) and merge them using special data structure. 

#### Write skew

Write skew as a generalization of the lost update problem. Write skew can occur if two transactions read the same objects, and then update some of those objects (different transactions may update different objects).

Automatically preventing write skew requires true serializable isolation.

The second-best option in this case is probably to explicitly lock the rows that the transaction depends on. Using the `FOR UPDATE` clause in SQL to lock the row that might be read by another transaction.

#### Serializability

Serializable isolation is usually regarded as the strongest isolation level. It guarantees that even though transactions may execute in parallel, the end result is the same as if they had executed one at a time, serially, without any concurrency. In other words, database prevents all possible race condition. 

Serializable isolation is done using one of these three methods.

**Actual Serial Execution**
- to execute only one transaction at a time, in serial order, on a single thread (one computer CPU).
- only recently do people think that a single-thread can execute at good performance because RAM because cheap enough to put entire dataset in memory.
- throughput limited to that of a single CPU core. 

**Stored procedure**
- encapsulates a series of transactions
- with single threaded processing, multi-statement transactions are not efficient, instead the entire transaction dode can be submitted as a stored procedure. This saves on network cost. less back and forth between database and application. 
- stored procedures can also be used during replication. Instead of copying a write at a time, stored procedure can be executed on each replica. This requires that stored procedures are deterministic (i.e. same result when run on different nodes).

For applications with high write throughput, the single-threaded transaction processor can become a serious bottleneck. One way is to partition the dataset so that each transaction only reads and writes data within a single partition such that each partition can have a dedicated CPU core. For any transaction that needs to access multiple partitions, the database must coordinate the transaction across all the partitions that it touches, and because of this is vastly slower than single partition transactions.

**Two-Phase Locking (2PL)**
- use shared lock when reading an object (share the lock with other transactions that want to read the object)
- use exclusive lock when writing an object (only allow one transaction to write)
- reads block writes, and writes blocks reads (and other writes). Contrast with snapshot isolation where readers never block writers and writers never block readers. Thus 2PL is more strict than snapshot isolation because it has to prevent the race conditions (concurrency issues like lost updates, and write skew).
- downside is that response times of queries, transaction throughput are much worse under 2PL than under weak isolationdue to overhead of acquiring and releasing all the locks. But also from reduced concurrency (only one writes at a time).
- databases running 2PL can have very slow latencies at high percentiles (unstable)

**predicate locks**
- It works similarly to the shared/ exclusive lock described earlier, but rather than belonging to a particular object (e.g., one row in a table), it belongs to all objects that match some search condition.
- prevent phantom writes - that is, one transaction changing the results of another transaction’s search query. Phantom writes have to be prevented for serializable isolation level.
- key idea here is that a predicate lock applies even to objects that do not yet exist in the database, but which might be added in the future (phantoms).

Predicate locks may be time-consuming if there are many locks by active transactions. Most databases implement **index-range locking**, a simplified approximation of predicate locking. 
- The idea is to simplify a predicate by making it match a greater set of objects. For example, if you have a predicate lock for bookings of room 123 between noon and 1 p.m., you can approximate it by locking bookings for room 123 at any time.
- The whole index is locked instead of the particular combination of room and time. 
- Index-range locks are not as precise as predicate locks (they lock a bigger range of objects) but they have much lower overheads. 

**Serializable Snapshot Isolation (SSI)**

This is a new algorithm to remedy the issues of serializable and snapshot isolation: Serializable don't perform well (two-phase locking) or don't scale well (serial execution) while weak isolation levels are prone to race conditions. When a transaction wants to commit, it is checked, and it is aborted if the execution was not serializable.

*Serializable snapshot isolation* is an optimistic concurrency control technique. Optimistic in this context means that instead of blocking if something potentially dangerous happens, transactions continue anyway, in the hope that everything will turn out all right. SSI allowing transactions to proceed without blocking like in 2PL.
- one of the way this algorithm works is by detecting stale MVCC (multi-version concurrency control) reads, recall that MVCC is the common strategy in snapshot isolation.
- another strategy is to detect writes that affect prior reads. The concurrent write which is committed later will be aborted. 

Compared to two-phase locking, the big advantage of serializable snapshot isolation is that one transaction doesn’t need to block waiting for locks held by another transaction. Like under snapshot isolation, writers don’t block readers, and vice versa. This design principle makes query latency much more predictable and less variable. In particular, read-only queries can run on a consistent snapshot without requiring any locks.

Compared to serial execution, serializable snapshot isolation is not limited to the throughput of a single CPU core: Even though data may be partitioned across multiple machines, transactions can read and write data in multiple partitions while ensuring serializable isolation.

SSI requires that read-write transactions be fairly short. However, SSI is probably less sensitive to slow transactions than two-phase locking or serial execution.

### Chapter 8 The Trouble with Distributed Systems

This chapter looks at an overview of things that may go wrong in a distributed systems such as network issues, clock and timing issues.

Single computers have a fully functional or entirely broken status making them easy to diagnose unlike distributed systems which may have partial failures. The challenge is that partial failures are non-deterministic: sometimes it works and sometimes it  unpredictably fails.

The fault handling must be part of the software design, and you (as operator of the software) need to know what behavior to expect from the software in the case of a fault. In distributed systems, suspicion, pessimism, and paranoia pay off.

The internet and most internal networks in datacenters (often Ethernet) are asynchronous packet networks. In this kind of network, one node can send a message (a packet) to another node, but the network gives no guarantees as to when it will arrive, or whether it will arrive at all.

The sender can’t even tell whether the packet was delivered: the only option is for the recipient to send a response message, which may in turn be lost or delayed. If you send a request to another node and don’t receive a response, it is impossible to tell why. The usual way of handling this issue is a timeout: after some time you give up waiting and assume that the response is not going to arrive.

Most systems we work with have neither of those guarantees: asynchronous networks have unbounded delays (that is, they try to deliver packets as quickly as possible, but there is no upper limit on the time it may take for a packet to arrive), and most server implementations cannot guarantee that they can handle requests within some maximum time.

You can only choose timeouts experimentally: measure the distribution of network round-trip times over an extended period, and over many machines, to determine the expected variability of delays.

Rather than using configured constant timeouts, systems can continually measure response times and their variability (jitter), and automatically adjust timeouts according to the observed response time distribution.

#### Unreliable clocks

Communication is not instantaneous: it takes time for a message to travel across the network from one machine to another.

Moreover, each machine on the network has its own clock, which is an actual hardware device: usually a quartz crystal oscillator. These devices are not perfectly accurate, so each machine has its own notion of time, which may be slightly faster or slower than on other machines.

It is possible to synchronize clocks to some degree: the most commonly used mechanism is the **Network Time Protocol (NTP)**, which allows the computer clock to be adjusted according to the time reported by a group of servers.

**time-of-day clocks**
- A time-of-day clock does what you intuitively expect of a clock: it returns the current date and time according to some calendar
- For example, the number of seconds (or milliseconds) since the epoch: midnight UTC on January 1, 1970, according to the Gregorian calendar, not counting leap seconds.
- If the local clock is too far ahead of the Network Time Protocol (NTP) server, it may be forcibly reset and appear to jump back to a previous point in time. These jumps, as well as similar jumps caused by leap seconds, make time-of-day clocks unsuitable for measuring elapsed time.

**monotonic clock**
- suitable for measuring a duration such as timeout or a response time.
- You can check the value of the monotonic clock at one point in time, do something, and then check the clock again at a later time. The difference between the two values tells you how much time elapsed between the two checks. However, the absolute value of the clock is meaningless.
- NTP allows the clock rate to be speeded up or slowed down by up to 0.05%, but NTP cannot cause the monotonic clock to jump forward or backward. The resolution of monotonic clocks is usually quite good: on most systems they can measure in microseconds.

The quartz clock in a computer is not very accurate: it drifts (runs faster or slower than it should). Clock drift varies depending on the temperature of the machine. Google assumes a clock drift of 200 ppm (parts per million) for its servers, which is equivalent to 17 seconds drift for a clock that is resynchronized once a day.

Further, NTP synchronization is only as good as the network delay, so there is a limit to its accuracy when we are in a congested network with variable packet delays. Delays can range from 35 ms at best to 1 second.

Although clocks work well most of the time, robust software eeds to be prepared to deal with incorrect clocks. The problem is that clock issues are usually unnoticed. 

When relying on timestamps to determine which writes occur first, problems can occur if the timestamp of each node is different. 
disappear: a node with a lagging clock is unable to overwrite values previously written by a node with a fast clock until the clock skew between the nodes has elapsed. This scenario can cause arbitrary amounts of data to be silently dropped without any error being reported to the application.

While it is tempting to keep the most “recent” value and discarding others, it’s important to be aware that the definition of “recent” depends on a local time-of-day clock, which may well be incorrect.

**Logical clocks**, which are based on incrementing counters rather than an oscillating quartz crystal, are a safer alternative for ordering events (see “Detecting Concurrent Writes”). Logical clocks do not measure the time of day or the number of seconds elapsed, only the relative ordering of events (whether one event happened before or after another). In contrast, time-of-day and monotonic clocks, which measure actual elapsed time, are also known as physical clocks.

It doesn’t make sense to think of a clock reading as a point in time — it is more like a range of times, within a confidence interval: for example, a system may be 95% confident that the time now is between 10.3 and 10.5 seconds past the minute, but it doesn’t know any more precisely than that.

If you’re getting the time from a server, the uncertainty is based on the expected quartz drift since your last sync with the server, plus the NTP server’s uncertainty, plus the network round-trip time to the server.

If you have two confidence intervals, each consisting of an earliest and latest possible timestamp (A = [Aearliest, Alatest] and B = [Bearliest, Blatest]), and those two intervals do not overlap (i.e., Aearliest < Alatest < Bearliest < Blatest), then B definitely happened after A — there can be no doubt. Only if the intervals overlap are we unsure in which order A and B happened.

#### Process pauses

Many programming language runtimes (such as the Java Virtual Machine) have a garbage collector (GC) that occasionally needs to stop all running threads. These “stop-the-world” GC pauses have sometimes been known to last for several minutes.

A virtual machine can be suspended (pausing the execution of all processes and saving the contents of memory to disk). This pause can occur at any time in a process’s execution and can last for an arbitrary length of time. This feature is sometimes used for live migration of virtual machines from one host to another without a reboot.

A distributed system has no shared memory — only messages sent over an unreliable network. A node in a distributed system must assume that its execution can be paused for a significant length of time at any point, even in the middle of a function. During the pause, the rest of the world keeps moving and may even declare the paused node dead because it’s not responding.
.

**Real-time** means that a system is carefully designed and tested to meet specified timing guarantees in all circumstances.

An enormous amount of testing and measurement must be done to ensure that guarantees are being met.

For these reasons, developing real-time systems is very expensive, and they are most commonly used in safety-critical embedded devices. Most server-side data processing systems, real-time guarantees are simply not economical or appropriate. Consequently, these systems must suffer the pauses and clock instability that come from operating in a non-real-time environment.

#### Knowledge, Truth, and Lies

With regard to timing assumptions, three system models are in common use:

- Synchronous model The synchronous model assumes bounded network delay, bounded process pauses, and bounded clock error. In other words: network delay, pauses, and clock drift will never exceed some fixed upper bound.
- Partially synchronous model Partial synchrony means that a system behaves like a synchronous system most of the time, but it sometimes exceeds the bounds for network delay, process pauses, and clock drift. This is a realistic model of many systems: most of the time, networks and processes are quite well behaved.
- Asynchronous model In this model, an algorithm is not allowed to make any timing assumptions — in fact, it does not even have a clock (so it cannot use timeouts). Very restrictive.

Most common type of node failure assumption:
- Crash-recovery faults: We assume that nodes may crash at any moment, and perhaps start responding again after some unknown time. In the crash-recovery model, nodes are assumed to have stable storage (i.e., nonvolatile disk storage) that is preserved across crashes, while the in-memory state is assumed to be lost.

Theoretical analysis can uncover problems in an algorithm that might remain hidden for a long time in a real system. They are incredibly helpful for distilling down the complexity of real systems to a manageable set of faults that we can reason about, so that we can understand the problem and try to solve it systematically.

Theoretical analysis and empirical testing are equally important.

### Chapter 9 Consistency and Consensus

Similar to transactions in chapter 7 where we assume by using a transaction, the application can pretend that there are no crashes (atomicity), that nobody else is concurrently accessing the database (isolation), and that storage devices are perfectly reliable (durability).

One of the most important abstractions for distributed systems is consensus: that is, getting all of the nodes to agree on something. 

First need to explore the range of guarantees and abstractions that can be provided in a distributed system.


#### Consistency Guarantees

If you look at two database nodes at the same moment in time, you’re likely to see different data on the two nodes, because write requests arrive on different nodes at different times.

Most replicated databases provide at least eventual consistency, which means that if you stop writing to the database and wait for some unspecified length of time, then eventually all read requests will return the same value.

A better name for eventual consistency may be convergence, as we expect all replicas to eventually converge to the same value.
But this is a weak guarantee as it doesn’t say anything about when the replicas will converge.

we will explore stronger consistency models that data systems may choose to provide. They don’t come for free: systems with stronger guarantees may have worse performance or be less fault-tolerant than systems with weaker guarantees.

Some similarity between distributed consistency models and the hierarchy of transaction isolation levels we discussed previously.
Transaction isolation is primarily about avoiding race conditions due to concurrently executing transactions, whereas distributed consistency is mostly about coordinating the state of replicas in the face of delays and faults.

We will start by looking at one of the strongest consistency models in common use, linearizability.

#### Linearizability

In an eventually consistent database, if you ask two different replicas the same question at the same time, you may get two different answers. That’s confusing. Wouldn’t it be a lot simpler if the database could give the illusion that there is only one replica (i.e., only one copy of the data)? Then every client would have the same view of the data, this is the idea behind linearizability.

The basic idea is to make a system appear as if there were only one copy of the data, and all operations on it are atomic. With this guarantee, even though there may be multiple replicas in reality, the application does not need to worry about them.

In a linearizable system, as soon as one client successfully completes a write, all clients reading from the database must be able to see the value just written. In other words, linearizability is a recency guarantee.

Distinction between Serializability and Linearizability:
- Serializability is an isolation property of transactions, where every transaction may read and write multiple objects (rows, documents, records). It guarantees that transactions behave the same as if they had executed in some serial order (each transaction running to completion before the next transaction starts).
- Linearizability is a recency guarantee on reads and writes of a register (an individual object). It doesn’t group operations together into transactions, so it does not prevent problems such as write skew

A database may provide both serializability and linearizability, and this combination is known as strict serializability.

Serializable snapshot isolation (see “Serializable Snapshot Isolation (SSI)”) is not linearizable: by design, it makes reads from a consistent snapshot, it does not include writes that are more recent than the snapshot, and thus reads from the snapshot are not linearizable.

If you want to ensure that two people don’t concurrently book the same seat on a flight or in a theater, this requires there to be a single up-to-date value (the account balance, the stock level, the seat occupancy) that all nodes agree on (i.e. linearizability).

Problem arises because there are two different communication channels between the web server and the resizer: the file storage and the message queue. Without the recency guarantee of linearizability, race conditions between these two channels are possible. If the file storage service is linearizable, then this system should work fine.

In a system with single-leader replication , the leader has the primary copy of the data that is used for writes, and the followers maintain backup copies of the data on other nodes. If you make reads from the leader, or from synchronously updated followers, they have the potential to be linearizable.

Systems with multi-leader replication are generally not linearizable, because they concurrently process writes on multiple nodes and asynchronously replicate them to other nodes.

In system with leaderless replication using strict quorum reads and writes (w+r > n), it is not quite linearizable. it is safest to assume that a leaderless system with Dynamo-style replication does not provide linearizability.

“Last write wins” conflict resolution methods based on time-of-day clocks (e.g., in Cassandra) are almost certainly nonlinearizable, because clock timestamps cannot be guaranteed to be consistent with actual event ordering due to clock skew.

#### Cost of Linearizability

For a single leader setup in multiple datacenters , if there is a network interruption between the two datacenters, causes the application to become unavailable in the datacenters that cannot contact the leader, clients that can only reach a follower datacenter will experience an outage until the network link is repaired.

The trade-off is as follows:
- If your application requires linearizability, and some replicas are disconnected from the other replicas due to a network problem, then some replicas cannot process requests while they are disconnected: they must either wait until the network problem is fixed, or return an error (either way, they become unavailable).
- If your application does not require linearizability, then it can be written in a way that each replica can process requests independently, even if it is disconnected from other replicas (e.g., multi-leader). In this case, the application can remain available in the face of a network problem, but its behavior is not linearizable.

Thus, applications that don’t require linearizability can be more tolerant of network problems.

Many distributed databases that choose not to provide linearizable guarantees: they do so primarily to increase performance, not so much for fault tolerance. Linearizability is slow — and this is true all the time, not only during a network fault.

One way of describing the trade-off is with the CAP theorem, either Consistent (linearizable) or Available (no down nodes) when Partitioned (or have network faults). A more reliable network needs to make this choice less often, but at some point the choice is inevitable.

#### Ordering Guarantees

Causality imposes an ordering on events: cause comes before effect; a message is sent before that message is received; the question comes before the answer.

If a system obeys the ordering imposed by causality, we say that it is causally consistent. For example, snapshot isolation provides causal consistency: when you read from the database, and you see some piece of data, then you must also be able to see any data that causally precedes it (assuming it has not been deleted in the meantime).

According to this definition, there are no concurrent operations in a linearizable datastore: there must be a single timeline along which all operations are totally ordered. There might be several requests waiting to be handled, but the datastore ensures that every request is handled atomically at a single point in time, acting on a single copy of the data, along a single timeline, without any concurrency.

#### Causal consistency

Linearizability is not the only way of preserving causality — there are other ways too. A system can be causally consistent without incurring the performance hit of making it linearizable

In many cases, systems that appear to require linearizability in fact only really require causal consistency, which can be implemented more efficiently.

In order to maintain causality, you need to know which operation happened before which other operation. In order to determine the causal ordering, the database needs to know which version of the data was read by the application.

Actually keeping track of all causal dependencies can become impracticable. Explicitly tracking all the data that has been read would mean a large overhead. 

It can instead come from a logical clock, which is an algorithm to generate a sequence of numbers to identify operations, typically using counters. Such sequence numbers or timestamps are compact (only a few bytes in size), and they provide a total order: that is, every operation has a unique sequence number, and you can always compare two sequence numbers to determine which is greater. Concurrent operations may be ordered arbitrarily.

If there is not a single leader (perhaps because you are using a multi-leader or leaderless database, or because the database is partitioned), it is less clear how to generate sequence numbers for operations. Other methods include:
- Each node can generate its own independent set of sequence numbers. For example, if you have two nodes, one node can generate only odd numbers and the other only even numbers. This would ensure that two different nodes can never generate the same sequence number.
- Attach a timestamp from a time-of-day clock (physical clock) to each operation. This is used in last write wins conflict resolution method.
- Preallocate blocks of sequence numbers. For example, node A might claim the block of sequence numbers from 1 to 1,000, and node B might claim the block from 1,001 to 2,000.

However, they all have a problem: the sequence numbers they generate are not consistent with causality. If one node generates even numbers and the other generates odd numbers, the counter for even numbers may lag behind the counter for odd numbers, you cannot accurately tell which one causally happened first. Similarly timestamps from physical clocks are subject to clock skew which makes them inconsistent with causality.

**Lamport timestamps**
- Each node has a unique identifier, and each node keeps a counter of the number of operations it has processed. The Lamport timestamp is then simply a pair of (counter, node ID). Each timestamp is made unique. 
- The counter keeps incrementing up. Every node and every client keeps track of the maximum counter value it has seen so far.
- if you have two timestamps, the one with a greater counter value is the greater timestamp; if the counter values are the same, the one with the greater node ID is the greater timestamp.

However, Lamport timestamps are not quite sufficient to solve many common problems in distributed systems.

In order to implement something like a uniqueness constraint for usernames, it’s not sufficient to have a total ordering of operations — you also need to know when that order is finalized.

#### Total order broadcast

Total order broadcast is usually described as a protocol for exchanging messages between nodes. Informally, it requires that two safety properties always be satisfied: 
- Reliable delivery No messages are lost: if a message is delivered to one node, it is delivered to all nodes. 
- Totally ordered delivery Messages are delivered to every node in the same order.

Another way of looking at total order broadcast is that it is a way of creating a log, delivering a message is like appending to the log.

To build a linearizable compare-and-set operation from total order broadcast:
You can sequence writes through the log by appending a message, reading the log, and performing the actual write when the message is delivered back to you. If the first message is from another user, then abort the operation.

Sequential consistency sometimes also known as timeline consistency is a slightly weaker guarantee than linearizability.

It can be proved that a linearizable compare-and-set (or increment-and-get) register and total order broadcast are both equivalent to consensus. 

#### Distributed Transactions and Consensus

Consensus: the goal is simply to get several nodes to agree on something.

There are a number of situations in which it is important for nodes to agree. For example:

Leader election 
- In a database with single-leader replication, all nodes need to agree on which node is the leader. The leadership position might become contested if some nodes can’t communicate with others due to a network fault. In this case, consensus is important to avoid a bad failover, resulting in a split brain situation.

Atomic commit 
- In a database that supports transactions spanning several nodes or partitions, we have the problem that a transaction may fail on some nodes but succeed on others.If we want to maintain transaction atomicity (in the sense of ACID; see “Atomicity”), we have to get all nodes to agree on the outcome of the transaction: either they all abort/ roll back (if anything goes wrong) or they all commit (if nothing goes wrong).


#### Atomic commit and two-phase commit

To fulfill transaction atomicity, the outcome of a transaction is either a successful commit, in which case all of the transaction’s writes are made durable, or an abort, in which case all of the transaction’s writes are rolled back. Atomicity prevents failed transactions from littering the database with half-finished results and half-updated state.

Thus, on a single node, transaction commitment crucially depends on the order in which data is durably written to disk: first the data, then the commit record [72]. The key deciding moment for whether the transaction commits or aborts is the moment at which the disk finishes writing the commit record: before that moment, it is still possible to abort (due to a crash), but after that moment, the transaction is committed (even if the database crashes).

If multiple nodes are involved in a transaction, if some nodes commit the transaction but others abort it, the nodes become inconsistent with each other. For this reason, a node must only commit once it is certain that all other nodes in the transaction are also going to commit.

A transaction commit must be irrevocable — you are not allowed to change your mind and retroactively abort a transaction after it has been committed.

**Two phase commit**
- Two-phase commit is an algorithm for achieving atomic transaction commit across multiple nodes — i.e., to ensure that either all nodes commit or all nodes abort. tplit into two phases.
- A 2PC transaction begins with the application reading and writing data on multiple database nodes, as normal.
- When the application is ready to commit, the coordinator begins phase 1: it sends a prepare request to each of the nodes, asking them whether they are able to commit.
- If any of the participants replies “no,” the coordinator sends an abort request to all nodes in phase 2.
- If all participants reply “yes,” indicating they are ready to commit, then the coordinator sends out a commit request in phase 2, and the commit actually takes place.
- 2PC uses a new component that does not normally appear in single-node transactions: a coordinator (also known as transaction manager).

The protocol contains two crucial “points of no return”: when a participant votes “yes,” it promises that it will definitely be able to commit later (although the coordinator may still choose to abort); and once the coordinator decides, that decision is irrevocable. Those promises ensure the atomicity of 2PC.

The coordinator must write its commit or abort decision to a transaction log on disk before sending commit or abort requests to participants: when the coordinator recovers, it determines the status of all in-doubt transactions by reading its transaction log. Any transactions that don’t have a commit record in the coordinator’s log are aborted. Thus, the commit point of 2PC comes down to a regular single-node atomic commit on the coordinator.

Much of the performance cost inherent in two-phase commit is due to the additional disk forcing (fsync) that is required for crash recovery [88], and the additional network round-trips.

**Exactly once messaging**
- For example, say a side effect of processing a message is to send an email, and the email server does not support two-phase commit: it could happen that the email is sent two or more times if message processing fails and is retried.

#### XA transactions

X/Open XA (Extended Architecture) is a standard for implementing two-phase commit across heterogeneous databases (different systems). It is not a network protocol but an API for interfacing with transaction coordinator. 

If the appplication network driver supports XA, that means it calls the XA API to find out whether an operation should be part of a distributed transaction — and if so, it sends the necessary information to the database server.

The transaction coordinator implements the XA API. It keeps track of the participants in a transaction, collects partipants’ responses after asking them to prepare, and uses a log on the local disk to keep track of the commit/ abort decision for each transaction.

If the application process crashes, the coordinator goes with it. Since the coordinator’s log is on the application server’s local disk, that server must be restarted, and the coordinator library must read the log to recover the commit/ abort outcome of each transaction. Only then can the coordinator use the database driver’s XA callbacks to ask participants to commit or abort.

Database transactions usually take a row-level exclusive lock on any rows they modify, to prevent dirty writes. In addition, if you want serializable isolation, a database using two-phase locking would also have to take a shared lock on any rows read by the transaction. Therefore, when using two-phase commit, a transaction must hold onto the locks throughout the time it is in doubt. If the coordinator has crashed and takes 20 minutes to start up again, those locks will be held for 20 minutes.

While those locks are held, no other transaction can modify those rows. Depending on the database, other transactions may even be blocked from reading those rows. This can cause large parts of your application to become unavailable until the in-doubt transaction is resolved.

In practice, orphaned in-doubt transactions do occur [89, 90] — that is, transactions for which the coordinator cannot decide the outcome for whatever reason. These transactions cannot be resolved automatically, so they sit forever in the database, holding locks and blocking other transactions. The only way out is for an administrator to manually decide whether to commit or roll back the transactions.


XA implementations have an emergency escape hatch called heuristic decisions: allowing a participant to decide to abort or commit an in-doubt transaction without a definitive decision from the coordinator. Heuristic here is a euphemism for probably breaking atomicity, since the heuristic decision violates the system of promises in two-phase commit.

XA does not work with SSI (Serializable Snapshot Isolation), since that would require a protocol for identifying conflicts across different systems. For database-internal distributed transactions (not XA), the limitations are not so great — for example, a distributed version of SSI is possible.

#### Fault tolerant consensus

The consensus problem is normally formalized as follows: one or more nodes may propose values, and the consensus algorithm decides on one of those values.

Properties a consensus algorithm must satisfy:
- Uniform agreement: No two nodes decide differently. 
- Integrity: No node decides twice. 
- Validity:  If a node decides value v, then v was proposed by some node. 
- Termination: Every node that does not crash eventually decides some value.

The termination property formalizes the idea of fault tolerance. It essentially says that a consensus algorithm cannot simply sit around and do nothing forever. Even if some nodes fail, the other nodes must still reach a decision.

There is a limit to the number of failures that an algorithm can tolerate: in fact, it can be proved that any consensus algorithm requires at least a majority of nodes to be functioning correctly in order to assure termination.

There is a limit to the number of failures that an algorithm can tolerate: in fact, it can be proved that any consensus algorithm requires at least a majority of nodes to be functioning correctly in order to assure termination.

To implement consensus algorithm, protocols define an epoch number (called the ballot number in Paxos, view number in Viewstamped Replication, and term number in Raft) and guarantee that within each epoch, the leader is unique. Epoch numbers are totally ordered and monotonically increasing.

If there is a conflict between two different leaders in two different epochs (perhaps because the previous leader actually wasn’t dead after all), then the leader with the higher epoch number prevails.

Before a leader is allowed to decide anything, it must first check that there isn’t some other leader with a higher epoch number which might take a conflicting decision.

We have two rounds of voting: once to choose a leader, and a second time to vote on a leader’s proposal. if the vote on a proposal does not reveal any higher-numbered epoch, the current leader can conclude that no leader election with a higher epoch number has happened, and therefore be sure that it still holds the leadership.

The biggest differences between consensus algorithm with 2PC is that in 2PC the coordinator is not elected, and that fault-tolerant consensus algorithms only require votes from a majority of nodes, whereas 2PC requires a “yes” vote from every participant.

The benefits come at a cost. The process by which nodes vote on proposals before they are decided is a kind of synchronous replication. As discussed in “Synchronous Versus Asynchronous Replication”, databases are often configured to use asynchronous replication. Consensus systems always require a strict majority to operate. This means you need a minimum of three nodes in order to tolerate one failure (the remaining two out of three form a majority), or a minimum of five nodes to tolerate two failures. 

Consensus systems generally rely on timeouts to detect failed nodes. Sometimes, consensus algorithms are particularly sensitive to network problems.

Some inmportant features of coordination services:
- linearizable atomic operations (requires consensus protocol)
- Total ordering of operations (requires fencing token that is monotonically increasing)
- Failure detection (client and server exchange heartbeats to check if other node is alive)
- Change notifications (each client can read locks and values created by another client)

Some consensus systems support read-only caching replicas. These replicas asynchronously receive the log of all decisions of the consensus algorithm, but do not actively participate in voting. They are therefore able to serve read requests that do not need to be linearizable.

## Part 3 Derived Data

Two types of data storage:

- **Systems of record**: A system of record, also known as source of truth, holds the authoritative version of your data. When new data comes in, e.g., as user input, it is first written here. Each fact is represented exactly once (the representation is typically normalized).

- **Derived data systems**: Data in a derived system is the result of taking some existing data from another system and transforming or processing it in some way.

Derived data is redundant, in the sense that it duplicates existing information. However, it is often essential for getting good performance on read queries. It is commonly denormalized. You can derive several different datasets from a single source, enabling you to look at the data from different “points of view.”

#### Chapter 10 Batch Processing


Kleppmann, Martin. Designing Data-Intensive Applications (p. 534). O'Reilly Media. Kindle Edition. 

Kleppmann, Martin. Designing Data-Intensive Applications (p. 534). O'Reilly Media. Kindle Edition. 

Kleppmann, Martin. Designing Data-Intensive Applications (p. 534). O'Reilly Media. Kindle Edition. 

Kleppmann, Martin. Designing Data-Intensive Applications (p. 534). O'Reilly Media. Kindle Edition. 
