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















