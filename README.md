_Reading notes | Completed in August 2020_

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

Three different types of systems:
- **Services (online systems)**: A service waits for a request or instruction from a client to arrive. When one is received, the service tries to handle it as quickly as possible and sends a response back.
- **Batch processing systems (offline systems)**: A batch processing system takes a large amount of input data, runs a job to process it, and produces some output data.
- **Stream processing systems (near-real-time systems)**: Stream processing systems (near-real-time systems) Stream processing is somewhere between online and offline/ batch processing (so it is sometimes called near-real-time or nearline processing). Like a batch processing system, a stream processor consumes inputs and produces outputs (rather than responding to requests). However, a stream job operates on events shortly after they happen, whereas a batch job operates on a fixed set of input data.

#### Batch Processing with Unix Tools

Unix commands are powerful for batch processing. For example to fin the five most popular pages on your website:
```
cat /var/log/nginx/access.log |  # Read log file
  awk '{print $7}' |             # Split each line into fields by whitespace, output only the seventh field from each line
  sort             |             # sort the list of URLs alphabetically
  uniq -c          |             # filters out repeated lines
  sort -r -n       |             # sort by the number at the start of each line, number of times URL was requested
  head -n 5        |             # outputs just the first five lines
```

The `sort` utility in GNU Coreutils (Linux) automatically handles larger-than-memory datasets by spilling to disk, and automatically parallelizes sorting across multiple CPU cores. This means that the simple chain of Unix commands we saw earlier easily scales to large datasets, without running out of memory. The bottleneck is likely to be the rate at which the input file can be read from disk.

In Unix, the output of every program can be expected to become the input to another, as yet unknown, program. Not many pieces of software interoperate and compose as well as Unix tools do. Even databases with the same data model often don’t make it easy to get data out of one and into the other.

Another characteristic feature of Unix tools is their use of standard input (stdin) and standard output (stdout). Pipes let you attach the stdout of one process to the stdin of another process. Separating the input/ output wiring from the program logic makes it easier to compose small tools into bigger systems.

Some nice features of Unix tools:

- The input files to Unix commands are normally treated as immutable. This means you can run the commands as often as you want, trying various command-line options, without damaging the input files.
- You can end the pipeline at any point, pipe the output into less, and look at it to see if it has the expected form. This ability to inspect is great for debugging. 
- You can write the output of one pipeline stage to a file and use that file as input to the next stage. This allows you to restart the later stage without rerunning the entire pipeline.

However, there are limits to what you can do with stdin and stdout. Programs that need multiple inputs or outputs are possible but tricky. You can’t pipe a program’s output into a network connection. The biggest limitation of Unix tools is that they run only on a single machine.

#### Map Reduce and Distributed Filesystems

MapReduce is a bit like Unix tools, but distributed across potentially thousands of machines.

A single MapReduce job is comparable to a single Unix process: it takes one or more inputs and produces one or more outputs.

In Hadoop's implementation of MapReduce, jobs read and write files on Hadoop Distributed File System (HDFS). 

HDFS is based on the shared-nothing principle architecture where nodes don't share memory, disks, or CPU. 

HDFS consists of a daemon process running on each machine, exposing a network service that allows other nodes to access files stored on that machine.

To create a MapReduce job, you need to implement two callback functions, the mapper and reducer, which behave as follows:
- **Mapper**: The mapper is called once for every input record, and its job is to extract the key and value from the input record. For each input, it may generate any number of key-value pairs (including none).
- **Reducer**: The MapReduce framework takes the key-value pairs produced by the mappers, collects all the values belonging to the same key, and calls the reducer with an iterator over that collection of values. The reducer can produce output records.

In MapReduce, if you need a second sorting stage, you can implement it by writing a second MapReduce job and using the output of the first job as input to the second job. Viewed like this, the role of the mapper is to prepare the data by putting it into a form that is suitable for sorting, and the role of the reducer is to process the data that has been sorted.

The main difference from pipelines of Unix commands is that MapReduce can parallelize a computation across many machines, without you having to write code to explicitly handle the parallelism.

The MapReduce scheduler runs each mapper on one of the machines that stores a replica of the input file. In most cases, the application code that should run in the map task is not yet present on the machine that is assigned the task of running it, so the MapReduce framework first copies the code (JAR file for Java program) to the machines.

It then starts the map task and begins reading the input file, passing one record at a time to the mapper callback. The output of the mapper consists of key-value pairs. 

The reduce side of the computation is also partitioned. While the number of map tasks is determined by the number of input file blocks, the number of reduce tasks is configured by the job author (it can be different from the number of map tasks). To ensure that all key-value pairs with the same key end up at the same reducer, the framework uses a hash of the key to determine which reduce task should receive a particular key-value pair.

The key-value pairs must be sorted, but the dataset is likely too large to be sorted with a conventional sorting algorithm on a single machine. Instead, the sorting is performed in stages.

Whenever a mapper finishes reading its input file and writing its sorted output files, the MapReduce scheduler notifies the reducers that they can start fetching the output files from that mapper. The reducers connect to each of the mappers and download the files of sorted key-value pairs for their partition. The process of partitioning by reducer, sorting, and copying data partitions from mappers to reducers is known as the shuffle.

The reduce task takes the files from the mappers and merges them together, preserving the sort order. If different mappers produced records with the same key, they will be adjacent in the merged reducer input.

The reducer is called with a key and an iterator that sequentially scans over all records with the same key (which may in some cases not all fit in memory). The reducer can use arbitrary logic to process these records, and can generate any number of output records. These output records are written to a file on the distributed filesystem.

#### MapReduce Workflows

it is very common for MapReduce jobs to be chained together into workflows, such that the output of one job becomes the input to the next job.

This chaining is done implicitly by directory name: the first job must be configured to write its output to a designated directory in HDFS, and the second job must be configured to use that same directory name for reading its input.

Chained MapReduce jobs are therefore less like pipelines of Unix commands and more like a sequence of commands where each command’s output is written to a temporary file, and the next command reads from the temporary file. One job in a workflow can only start when the prior jobs — that is, the jobs that produce its input directories — have completed successfully.

To handle these dependencies between job executions, various workflow schedulers for Hadoop have been developed, including Oozie, Azkaban, Luigi, Airflow, and Pinball.

MapReduce has no concept of indexes — at least not in the usual sense. When a MapReduce job is given a set of files as input, it reads the entire content of all of those files; a database would call this operation a full table scan. If you only want to read a small number of records, a full table scan is outrageously expensive compared to an index lookup.


#### Joining and Grouping in Reduce side 
However, in analytic queries it is common to want to calculate aggregates over a large number of records. In this case, scanning the entire set of input files might be quite a reasonable thing to do, especially if it can be parallelized to multiple machines.


In order to achieve good throughput in a batch process, the computation must be (as much as possible) local to one machine. Making random-access requests over the network for every record you want to process is too slow.

One approach to joining tables will be to create a copy of the other table into the same HDFS, then MapReduce could be used to process the records in the same place efficiently.

In **sort-merge join**, the mapper(s) output is sorted by key, and the reducers then merge together the sorted lists of records from both sides of the join. In a sort-merge join, the mappers and the sorting process make sure that all the necessary data to perform the join operation for a particular user ID is brought together in the same place: a single call to the reducer.

Since MapReduce handles all network communication, it also shields the application code from having to worry about partial failures, such as the crash of another node: MapReduce transparently retries failed tasks without affecting the application logic.

Besides joins, another common use of the “bringing related data to the same place” pattern is grouping records by some key (as in the GROUP BY clause in SQL). A simple way to group by is to set up the mappers so that the key-value pairs they produce use the desired grouping key. The partitioning and sorting process then brings together all the records with the same key in the same reducer.

Another common use for grouping is collating all the activity events for a particular user session, in order to find out the sequence of actions that the user took — a process called sessionization.

The pattern of “bringing all records with the same key to the same place” breaks down if there is a very large amount of data related to a single key (skew or hot spots). One approach is to scan the records for hot keys and spread the work of handling the hot key over several reducers, at the cost of having to replicate the other join input to multiple reducers.

When grouping records by a hot key and aggregating them, you can perform the grouping in two stages. The first MapReduce stage sends records to a random reducer, so that each reducer performs the grouping on a subset of records for the hot key and outputs a more compact aggregated value per key. The second MapReduce job then combines the values from all of the first-stage reducers into a single value per key.

#### Map side joins

In reduce-side joins, The mappers take the role of preparing the input data: extracting the key and value from each input record, assigning the key-value pairs to a reducer partition, and sorting by key.

However, the downside of reduce side join is that all that sorting, copying to reducers, and merging of reducer inputs can be quite expensive.

If you can make certain assumptions about your input data, it is possible to make joins faster by using a so-called map-side join. This approach uses a cut-down MapReduce job in which there are no reducers and no sorting. Instead, each mapper simply reads one input file block from the distributed filesystem and writes one output file to the filesystem — that is all.

- The simplest way of performing a map-side join applies in the case where a large dataset is joined with a small dataset. The small dataset needs to be small enough that it can be loaded entirely into memory in each of the mappers. THis is called **broadcast hash join**. So the small input is effectively “broadcast” to all partitions of the large input.
- **partition hash joins**: If the inputs to the map-side join are partitioned in the same way, then the hash join approach can be applied to each partition independently. This approach only works if both of the join’s inputs have the same number of partitions, with records assigned to partitions based on the same key and the same hash function.
- Another variant of a map-side join applies if the input datasets are not only partitioned in the same way, but also sorted based on the same key. So called **map-side merge joins**. If a map-side merge join is possible, it probably means that prior MapReduce jobs brought the input datasets into this partitioned and sorted form in the first place.

It is important to know the number of partitions and the keys by which the data is partitioned and sorted.

#### Batch workflow outputs

The output of a batch process is often not a report, but some other kind of structure.

A batch process is a very effective way of building the indexes for full-text search system: the mappers partition the set of documents as needed, each reducer builds the index for its partition, and the index files are written to the distributed filesystem.


Search indexes are just one example of the possible outputs of a batch processing workflow. Another common use for batch processing is to build machine learning systems such as classifiers and recommendation systems. The output of these batch jobs is some kind of database that can be queried. 

One way is to build a brand-new database inside the batch job and write it as files to the job’s output directory in the distributed filesystem, just like the search indexes described above. Since Most of these key-value stores (database) are read-only (the files can only be written once by a batch job and are then immutable), the data structures are quite simple.

By treating inputs as immutable and avoiding side effects (such as writing to external databases), batch jobs not only achieve good performance but also become much easier to maintain.

the design principles that worked well for Unix also seem to be working well for Hadoop — but Unix and Hadoop also differ in some ways. On Hadoop, some of those low-value syntactic conversions are eliminated by using more structured file formats: Avro are often used, as they provide efficient schema-based encoding and allow evolution of their schemas over time.

#### Hadoop versus traditional distributed databases

Hadoop is somewhat like a distributed version of Unix, where HDFS is the filesystem and MapReduce is a quirky implementation of a Unix process (which happens to always run the sort utility between the map phase and the reduce phase).

Hadoop distributed filesystem (HDFS) supports data encoded in any format.

To put it bluntly, Hadoop opened up the possibility of indiscriminately dumping data into HDFS, and only later figuring out how to process it further. By contrast, massively parallel processing (MPP) databases typically require careful up-front modeling of the data and query patterns before importing the data into the database’s proprietary storage format.

In practice, it appears that simply making data available quickly — even if it is in a quirky, difficult-to-use, raw format — is often more valuable than trying to decide on the ideal data model up front.

Simply dumping data in its raw form allows for several such transformations. This approach has been dubbed the sushi principle: “raw data is better”.

Hadoop has often been used for implementing ETL processes: data from transaction processing systems is dumped into the distributed filesystem in some raw form, and then MapReduce jobs are written to clean up that data, transform it into a relational form, and import it into an MPP data warehouse for analytic purposes.

In MPP databases, they are optimized for querying using SQL. But not all kinds of processing can be sensibly expressed as SQL queries. Some applications require writing code not just queries. MapReduce gave engineers the ability to easily run their own code over large datasets. If you have HDFS and MapReduce, you can build a SQL query execution engine on top of it, and indeed this is what the Hive project did.

MapReduce”). Having two processing models, SQL and MapReduce, was not enough: even more different models were needed! And due to the openness of the Hadoop platform, it was feasible to implement a whole range of approaches, which would not have been possible within the confines of a monolithic MPP database.

In the Hadoop approach, there is no need to import the data into several different specialized systems for different kinds of processing: the system is flexible enough to support a diverse set of workloads within the same cluster. Not having to move data around makes it a lot easier to derive value from the data, and a lot easier to experiment with new processing models.

When comparing MapReduce to MPP databases, two more differences in design approach stand out: the handling of faults and the use of memory and disk. The MapReduce approach is more appropriate for larger jobs: jobs that process so much data and run for such a long time that they are likely to experience at least one task failure along the way.

MapReduce is designed to tolerate frequent unexpected task termination. In an environment where tasks are not so often terminated, the design decisions of MapReduce make less sense.

#### Beyond Map Reduce

Implementing a complex processing job using the raw MapReduce APIs is actually quite hard and laborious — for instance, any join algorithms used by the job would need to be implemented from scratch.

On the one hand, MapReduce is very robust: you can use it to process almost arbitrarily large quantities of data on an unreliable multi-tenant system with frequent task terminations, and it will still get the job done (albeit slowly). On the other hand, other tools are sometimes orders of magnitude faster for some kinds of processing.

**materialization**
> The process of computing the result of some operation eagerly and writing it out, rather than computing it on demand when requested.

In a workflow with many MapReduce jobs, the output of a task is stored in a file in the distributed filesystem which serves as intermediate state, so it can be be used as input to the next task. In contrast with Unix pipelines, pipes in Unix do not fully materialize the intermediate state, but instead stream one command’s output to the next command’s input incrementally, using only a small in-memory buffer.

MapReduce’s approach of fully materializing intermediate state has downsides compared to Unix pipes:
- A MapReduce job can only start when all tasks in the preceding jobs (that generate its inputs) have completed, whereas processes connected by a Unix pipe are started at the same time, with output being consumed as soon as it is produced.
- Mappers are often redundant: they just read back the same file that was just written by a reducer. In many cases, the mapper code could be part of the previous reducer.
- Storing intermediate state in a distributed filesystem means those files are replicated across several nodes, which is a waste.

There are new execution engines for distributed batch computations such as Spark and Flink (dataflow engines. They handle an entire workflow as one job, rather than breaking it up into independent subjobs.

Unlike in MapReduce, the new dataflow engines can use user-defined functions (operators) which need not take the strict roles of alternating map and reduce.

The dataflow engine provides several different options for connecting one operator’s output to another’s input:
- repartition and sort records by key, like in the shuffle stage of MapReduce. enables sort-merge joins and grouping in the same way as in MapReduce.
- Another possibility is to take several inputs and to partition them in the same way, but skip the sorting. This saves effort on partitioned hash joins, where the partitioning of records is important but the order is irrelevant because building the hash table randomizes the order anyway.
- For broadcast hash joins, the same output from one operator can be sent to all partitions of the join operator.

The advantages of the flexibility of new dataflow engines compared to MapReduce are:
- Expensive work such as sorting need only be performed in places where it is actually required, rather than always happening by default between every map and reduce stage.
- There are no unnecessary map tasks, since the work done by a mapper can often be incorporated into the preceding reduce operator.
- The scheduler has an overview of what data is required where, so it can make locality optimizations.
- It's usually sufficient for intermediate state between operators to be kept in memory or written to local disk, which requires less I/O than writing to HDFS. 
- Operators can start executing as soon as their input is ready; there is no need to wait for the entire preceding stage to finish before the next one starts. 

#### Fault tolerance in dataflow engines

Spark, Flink, and Tez avoid writing intermediate state to HDFS, so they take a different approach to tolerating faults: if a machine fails and the intermediate state on that machine is lost, it is recomputed from other data that is still available.

the framework must keep track of how a given piece of data was computed — which input partitions it used, and which operators were applied to it. Spark uses the resilient distributed dataset (RDD) abstraction for tracking the ancestry of data.

In order to avoid cascading downstream operators faults, it is better to make operators deterministic (use fixed seed when generating random numbers ,no reliance on system clocks and no external data source). Such causes of nondeterminism need to be removed in order to reliably recover from faults.

When the job completes, its output needs to go somewhere durable so that users can find it and use it — most likely, it is written to the distributed filesystem again. Thus, when using a dataflow engine, materialized datasets on HDFS are still usually the inputs and the final outputs of a job. The improvement over MapReduce is that you save yourself writing all the intermediate state to the filesystem as well.

#### Graphs and Iterative Processing

One of the most famous graph analysis algorithms is PageRank, which tries to estimate the popularity of a web page based on what other web pages link to it. It is used as part of the formula that determines the order in which web search engines present their results.

Graph algorithm is thus often implemented in an iterative style:
- An external scheduler runs a batch process to calculate one step of the algorithm.
- When the batch process completes, the scheduler checks whether the iterative algorithm has finished
 If the algorithm has not yet finished, the scheduler goes back to step 1

This approach works, but implementing it with MapReduce is often very inefficient, because MapReduce does not account for the iterative nature of the algorithm: it will always read the entire input dataset and produce a completely new output dataset, even if only a small part of the graph has changed compared to the last iteration.

The bulk synchronous parallel (BSP) model of computation, also known as the Pregel model has become popular. The idea behind Pregel: one vertex can “send a message” to another vertex, and typically those messages are sent along the edges in a graph.

The difference from MapReduce is that in the Pregel model, a vertex remembers its state in memory from one iteration to the next, so the function only needs to process new incoming messages.

The fact that vertices can only communicate by message passing (not by querying each other directly) helps improve the performance of Pregel jobs. Even though the underlying network may drop, duplicate, or arbitrarily delay messages (see “Unreliable Networks”), Pregel implementations guarantee that messages are processed exactly once at their destination vertex in the following iteration.

Fault tolerance is achieved by periodically checkpointing the state of all vertices at the end of an iteration — i.e., writing their full state to durable storage. If a node fails and its in-memory state is lost, the simplest solution is to roll back the entire graph computation to the last checkpoint and restart the computation.

Graph algorithms often have a lot of cross-machine communication overhead, as vertices that communicate a lot may not be partitioned in the same machine and the intermediate state (messages sent between nodes) is often bigger than the original graph.

For this reason, if your graph can fit in memory on a single computer, it’s quite likely that a single-machine (maybe even single-threaded) algorithm will outperform a distributed batch process. Efficiently parallelizing graph algorithms is an area of ongoing research

#### High Level APIS and Languages

As the problem of physically operating batch processes at very large scale has been considered more or less solved, attention has turned to other areas: improving the programming model, improving the efficiency of processing, and broadening the set of problems that these technologies can solve.

Dataflow APIs (Spark, Flink) generally use relational-style building blocks to express a computation: joining datasets on the value of some field; grouping tuples by key; filtering by some condition; and aggregating tuples by counting, summing, or other functions.

Besides the obvious advantage of requiring less code, these high-level interfaces also allow interactive use, in which you write analysis code incrementally in a shell and run it frequently to observe what it is doing.

The benefit of specifying joins as relational operator (in dataflow APIs) instead of spelling out the code that performs the join in MapReduce, dataflow framework can analyze the properties of the join inputs and automatically decide which of the aforementioned join algorithms would be most suitable.

Hive, Spark, and Flink have cost-based query optimizers that can do this, and even change the order of joins so that the amount of intermediate state is minimized.

By incorporating declarative aspects in their high-level APIs, and having query optimizers that can take advantage of them during execution, batch processing frameworks begin to look more like MPP databases (and can achieve comparable performance). At the same time, by having the extensibility of being able to run arbitrary code and read/ write data in arbitrary formats, they retain their flexibility advantage. Specialization for different

#### Chapter 11: Stream Processing 

 In batch processing, the input is bounded — i.e., of a known and finite size, so a batch process knows when it has finished reading its input.

Sometimes the time interval between batch processes may be too slow, to reduce delay, the processing can be run more frequently, say every second or every event as it comes in. This is idea is called stream processing. 

#### Transmitting Event Streams

In a stream processing context, a record is more commonly known as an event: a small, self-contained, immutable object containing
the details of something that happened at some point in time. For example, an action that a user took, such as viewing a page or making a purchase.

in streaming terminology, an event is generated once by a producer (also known as a publisher or sender), and then potentially processed by multiple consumers (subscribers or recipients). In a streaming system, related events are usually grouped together into a topic or stream.

Each consumer can periodically poll the datastore to check for new events, but frequent polling becomes expensive especially at high frequency, it is better to get notified when new events appear. 

Relational databases have triggers, which react to a change (a row being inserted) but they are limited in what they do. Messaging systems are designed to deliver event notifications.

#### Messaging Systems

A common approach for notifying consumers about new events is to use a messaging system: a producer sends a message containing the event, which is then pushed to consumers.

A messaging system allows multiple producer nodes to send messages to the same topic and allows multiple consumer nodes to receive messages in a topic.

What if producers send messages faster than consumers can process them?
- drop new messages,
- buffer messages in a queue
- apply backpressure, prevent producer from sending more messages until recipient takes data out of buffer. 

What if nodes crash or temporarily go offline - are messages lost?
- for durability, may require replication or writing to dish, which has a cost

Whether message loss is acceptable depends on the system, a high frequency sensor can afford to lose a few points. 

#### Direct messaging producers to consumers

Some messaging systems use direct network communication between producers and consumers. 

hese direct messaging systems work well in the situations for which they are designed, they generally require the application code to be aware of the possibility of message loss.

The faults they can tolerate are quite limited: even if the protocols detect and retransmit packets that are lost in the network, they generally assume that producers and consumers are constantly online. 

If a consumer is offline, it may miss messages that were sent while it is unreachable. Some protocols allow the producer to retry failed message deliveries, but this approach may break down if the producer crashes, losing the buffer of messages that it was supposed to retry.

#### Message Brokers

An alternative is to send messages via a message broker (message queue), which is a database for handling message streams. 
It runs as a server, with producers and consumers connecting to it as clients. Producers write messages to the broker, and consumers receive them by reading them from the broker.

Some brokers write messages to disk to avoid losing when they crash. 

Message delivery is asynchronous, if a producer sends a message to the broker, it does not wait for the broker to send the message to the consumers. Delivery to consumer may happen immediately or after a while if there is a backlog. 

Differences between message brokers and databases:
- Databases usually keep data until it is explicitly deleted, whereas most message brokers automatically delete a message when it has been successfully delivered to its consumers.
- Since they quickly delete messages, most message brokers assume that their working set is fairly small — i.e., the queues are short. A large message queue may slow down throughput. 
- Databases often support secondary indexes and various ways of searching for data, while message brokers often support some way of subscribing to a subset of topics matching some pattern. Essentially a way for clients to select a portion of data. 
- Message brokers notify clients (Producers and consumers) if data changes

#### Multiple consumers

When multiple consumers read messages in the same topic, the following two strategies can be used:

**Load Balancing**
- each message is delivered to one (only one) of the consumers, so the consumers can share the work of processing the messages in the topic. Assignment of messages to consumers may be arbitrary. Useful when messages are expensive to process. 

**Fan-out**
- Each message is delivered to all of the consumers. All consumers receive the same broadcast of messages.

The two strategies can be combined. Two separate groups of consumers may each subscribe to a topic, such that each group receives all messages but within each group only one of the nodes receives each message. 

#### Acknowledgements and redelivery

Consumers may crash at any time, so it could happen that a broker delivers a message to a consumer but the consumer never processes it,

In order to ensure that the message is not lost, message brokers use acknowledgments: a client must explicitly tell the broker when it has finished processing a message so that the broker can remove it from the queue.

As consumers may crash and brokers reinitiate delivery, messages may not be delivered in the same order they were sent by producer, the combination of load balancing with redelivery inevitably leads to messages being reordered. To avoid this issue, you can use a separate queue per consumer (i.e., not use the load balancing feature).

Message reordering is not an issue if messages are independent of each other but there might be causal dependencies between messages.

#### Partitioned Logs

Unlike batch processing which does not damage the input data, with AMQP/ JMS-style messaging: receiving a message is destructive if the acknowledgment causes it to be deleted from the broker, so you cannot run the same consumer again and expect to get the same result.

When adding a new consumer to a messaging system, it typically only starts receiving messages sent after the time it was registered; any prior messages are already gone and cannot be recovered. 

Why not have both, combining the durable storage of databases and low-latency notification of messaging brokers. This idea is log-based message brokers. 

#### Using logs for message store

A log is simply an append-only sequence of records on disk.

The same structure can be used to implement a message broker: a producer sends a message by appending it to the end of the log, and a consumer receives messages by reading the log sequentially.

In order to scale to higher throughput than a single disk can offer, the log can be partitioned. 

Different partitions can then be hosted on different machines, making each partition a separate log that can be read and written independently from other partitions. A topic can then be defined as a group of partitions that all carry messages of the same type.

Within each partition, the broker assigns a monotonically increasing sequence number, or offset, to every message.

Because a partition is append-only, so the messages within a partition are totally ordered, But there is no ordering guarantee across different partitions.

Apache Kafka, Amazon Kinesis Streams, and Twitter's DistributedLog are log-based message brokers that work like this. Even though these message brokers write all messages to disk, they are able to achieve throughput of millions of messages per second by partitioning across multiple machines, and fault tolerance by replicating messages.

#### Logs compared to traditional messaging

To achieve load balancing across a group of consumers, instead of assigning individual messages to consumer clients, the broker can **assign entire partitions to nodes** in the consumer group.

Each client then consumes all the messages in the partitions it has been assigned. When a consumer has been assigned a log partition, it reads the messages in the partition sequentially, in a single-threaded manner.

Downsides to assigning entire log partitions to consumers:
- The number of nodes sharing the work of consuming a topic can be at most the number of log partitions in that topic
- If a single message is slow to process, it holds up the processing of subsequent messages in that partition

Thus, in situations where messages may be expensive to process and you want to parallelize processing on a message-by-message basis, and where message ordering is not so important, the JMS/ AMQP style of message broker is preferable. On the other hand, in situations with high message throughput, where each message is fast to process and where message ordering is important, the log-based approach works very well.

Partitioned logs typically preserve message ordering only within a single partition, thus all messages that need to be ordered consistently need to be routed to the same partition. 

#### Consumer offsets

The broker does not need to track acknowledgments for every single message — it only needs to periodically record the consumer offsets.

If a consumer node fails, another node in the consumer group is assigned the failed consumer’s partitions, and it starts consuming messages at the last recorded offset.

#### Disk space usage

Disk space is limited and if we keep appending to the log, space will run out. To reclaim disk space, the log is actually divided into segments, and from time to time old segments are deleted or moved to archive storage.

If a slow consumer cannot keep up with the rate of messages, and it falls so far behind that its consumer offset points to a deleted segment, it will miss some of the messages.

In practice, the log can typically keep a buffer of several days’ or even weeks’ worth of messages.

The log-based approach is essentially a form of buffering with a large but fixed-size buffer limited by disk space. 

If a consumer falls so far behind that the messages it requires are older than what is retained on disk, it will not be able to read those messages.

You can monitor how far a consumer is behind the head of the log, and raise an alert if it falls behind significantly.

As each consumer is independent, we can experiment with production log for development, testing, or debugging without affecting production services. 

In AMQP- and JMS-style message brokers, processing and acknowledging messages is a destructive operation, since it causes the messages to be deleted on the broker. In a log-based message broker, consuming messages is more like reading from a file: it  does not change the log. This makes log-based messaging more like batch processes, where derived data is separate from input data. 

#### Databases and Streams

As data appears in several different places, they need to be kept in sync with one another: if an item is updated in the database, it also needs to be updated in the cache, search indexes, and data warehouse.

One approach is to do batch processes to update the data. another approach is to do dual writes, where write to database is followed by updating search index, cache entries (or concurrently).

Dual writes can suffer from race condition. Another issue is if one of the writes fail. Then there is no consistency between the index/cache and the database. 

The situation would be better if there really was only one leader and if we could make the search index a follower of the database.

#### Change data capture

There has been growing interest in change data capture (CDC), which is the process of observing all data changes written to a database and extracting them in a form in which they can be replicated to other systems.

The search index and any other derived data systems are just consumers of the change stream. 

Change data capture is a mechanism for ensuring that all changes made to the system of record are also reflected in the derived data systems (copyt of the data in search index, cache, data warehouse). 

Change data capture makes one database the leader (the one from which the changes are captured), and turns the others into followers. **A log-based message broker is well suited** for transporting the change events from the source database to the derived systems, since it preserves the ordering of messages.

Change data capture is usually asynchronous, it does not wait for consumers to process changes before committing it. Thus it is also susceptible to issues of replication lag. 

#### Initial snapshot for change data capture. 

If we don't have the entire log history of the database, we need to start with a consistent snapshot. The snapshot of the database must correspond to a known position or offset in the change log, so that you know at which point to start applying changes after the snapshot has been processed.

#### Log compaction

Log compaction is a good idea to limit disk usage and allow keeping longer history of the log. 

If the CDC system is set up such that every change has a primary key, and every update for a key replaces the previous value for that key, then it’s sufficient to keep just the most recent write for a particular key.

The log is guaranteed to contain the most recent value for every key in the database.

#### Event sourcing

Similarly to change data capture, event sourcing involves storing all changes to the application state as a log of change events.

The main difference is that even sourcing applies idea at a different level of abstraction:
- In change data capture, tthe log of changes is extracted from the database at a low level (e.g., by parsing the replication log), which ensures that the order of writes extracted from the database matches the order in which they were actually written
- In event sourcing, the application logic is explicitly built on the basis of immutable events that are written to an event log. Events are designed to reflect things that happened at the application level, rather than low-level state changes.

Event sourcing is a powerful technique for data modeling: it is more meaningful to record the user’s actions as immutable events,
than the result of the action in a database. 

Event sourcing makes it easier to help with debugging by making it easier to understand after the fact why something happened.

#### Deriving current state from event logs

An event log by itself is not very useful, because users generally expect to see the current state of a system, the history of modifications.

Applications that use event sourcing need to take the log of events (representing the data written to the system) and transform it into application state that is suitable for showing to a user.

Unlike change data capture, in even logs, later events typically do not override prior events, and so you need the full history of events to reconstruct the final state. Because an event also contains the intent of the user action, so we may want to store that. 

#### Commands and events

When a request from a user first arrives, it is initially a command: at this point it may still fail, if validation passes the command is accepted, **it becomes an event, which is durable and immutable**.

At the point when the event is generated, it becomes a fact. Any validation of a command needs to happen synchronously, before it becomes an event — for example, by using a serializable transaction that atomically validates the command and publishes the event.

But validation can also happen asynchronously if the action is split into two events, first a tentative event, and then a separate confirmation event once the action is validated.

#### State, Streams, and Immutability

This principle of immutability makes event sourcing and change data capture so powerful.

The key idea is that mutable state and an append-only log of immutable events do not contradict each other: they are two sides of the same coin. The log of all changes, the changelog, represents the evolution of state over time.

You might say that the application state is what you get when you integrate an event stream over time, and a change stream is what you get when you differentiate the state by time. 

The contents of the database hold a caching of the latest record values in the logs. The truth is the log.

This idea of storing events is not new. In financial world, each transaction is recorded in an append-only ledger. 

With an append-only log of immutable events, it is much easier to diagnose what happened and recover from the problem.

Some additional information is also stored in events that are not captured in the final state of the database. 

The traditional approach to database and schema design is based on the fallacy that data must be written in the same form as it will be queried. If you can translate data from a write-optimized event log to read-optimized application state: it is entirely reasonable to denormalize data in the read-optimized views, as the translation process from event log to read views gives you a mechanism to be consistent. 

The biggest downside of event sourcing and change data capture is that consumers of event logs are asynchronous. 

There is a possibility that a user may make a write to the log, then read from a log-derived view and find that their write has not yet been reflected in the read view. One solution would be to perform the updates of the read view synchronously with appending the event to the log. This requires a transaction to combine the writes into an atomic unit.

Deriving the current state from an event log also simplifies some aspects of concurrency control. The user action then requires only a single write in one place - the event log. 

If the event log and the application state are partitioned in the same way, a single threaded log consumer needs no concurrency control for writes as it only processes a single event at a time. 

The downside of immutability comes when you want to delete a data say for compliance purposes. It’s not sufficient to just append another event to the log to indicate that the prior data should be considered deleted — you actually want to rewrite history and pretend that the data was never written in the first place.

Truly deleting data is surprisingly hard as copies may live in many places with backups. Deletion is more a matter of “making it harder to retrieve the data” than actually “making it impossible to retrieve the data.”

#### Processing Streams

Three possible way to process a stream:
- take the data in the events and write it to a database, cache, search index, or similar storage system, from where it can then be queried by other clients.
- push the events to users in some way, for example by sending email alerts or push notifications, or by streaming the events to a real-time dashboard.
- process one or more input streams to produce one or more output streams. Streams may go through a pipeline consisting of several such processing stages before ending up at an output.

A stream processor consumes input streams in a read-only fashion and writes its output to a different location in an append-only fashion.

Sorting does not make sense with an unbounded dataset.

#### Uses of Stream Processing

- Fraud detection system
- Trading system to examine price changes and execute trades
- Manufacturing systems in factory to identify a malfunction
- Military and Intelligence systems to identify activities of potential aggressor. 

**Complex event processing**
- CEP allows you to specify rules to search for certain patterns of events in a stream.
- CEP systems often use a high-level declarative query language like SQL, or a graphical user interface, to describe the patterns of events that should be detected.
- When a match is found, the engine emits a complex event (hence the name) with the details of the event pattern that was detected

**Stream analytics**
- geared toward aggregations and statistical metrics over a large number of events for instance measuring the rolling average of a value over time
- statistics are usually computed over fixed time intervals
- sometimes use probabilistic algorithms, such as Bloom filters for set membership, an other percentile estimation algorithms.

Streaming can also be used to continuously update a materialized view, for example, a view in a data warehouse which needs to be updated every now and then. Unlike streaming analytics, this requires the entire series of events in the timeline, not just in a small window. 

**Search on streams**
- searching for individual events based on complex criteria, such as full-text search queries. 
- queries are stored and documents are run past the queries. But there could be a lot of queries.
- To optimize the process, it is possible to index the queries as well as the documents, and thus narrow down the set of queries that may match.

#### Reasoning about time

The meaning of “the last five minutes” time window should be unambiguous and clear, but unfortunately the notion is surprisingly tricky.

Many stream processing frameworks use the local system clock on the processing machine (the processing time) to determine windowing. This approach has the advantage of being simple, however, it breaks down if there is any significant processing lag.

Instead, using the timestamps in the events allows the processing to be deterministic: running the same process again on the same input yields the same result.

A tricky problem when defining windows in terms of event time is that you can never be sure when you have received all of the events for a particular window, or whether there are some events still to come. 

It could happen that some events were buffered on another machine somewhere, delayed due to a network interruption. You need to be able to handle such straggler events that arrive after the window has already been declared complete. Two solutions
- Ignore the straggler events. Alert if a lot of stragglers
- Publish a correction, updated value for the window with the stragglers included. Need to retract previous value.

To adjust for incorrect device clocks, one approach is to log three timestamps: 
- The time at which the event occurred, according to the device clock 
- The time at which the event was sent to the server, according to the device clock 
- The time at which the event was received by the server, according to the server clock

Subtract two from three to get the offset between server clock and device clock, then we can know the time that the event occurred. 

#### Stream Joins

The fact that new events can appear anytime on a stream makes joins on streams more challenging than in batch jobs.

Three different types of joins: stream-stream joins, stream-table joins, and table-table joins:

**Stream-stream join (window join)**
- Both input streams consist of activity events, and the join operator searches for related events that occur within some window of time.

**Stream-table join (steam enrichment)**
- One input stream consists of activity events, while the other is a database changelog.
- to load a copy of the database into the stream processor so that it can be queried locally without a network round-trip
- the stream processor’s local copy of the database needs to be kept up to date. This issue can be solved by change data capture: the stream processor can subscribe to a changelog of the table database as well as the stream of activity events.

**Table-table join (materialized view maintenance)**
- Both input streams are database changelogs.
- Every change on one side is joined with the latest state of the other side.
- The result is a stream of changes to the materialized view of the join between the two tables.

The three types of joins  have a lot in common: they all require the stream processor to maintain some state (search and click events, user profiles, or follower list) based on one join input, and query that state on messages from the other join input.

The order of the events that maintain the state is important. In a partitioned log, the ordering of events within a single partition is preserved, but there is typically no ordering guarantee across different streams or partitions.

If the ordering of events across streams is undetermined, the join becomes nondeterministic which means you cannot rerun the same job on the same input and necessarily get the same result. 

This issue is known as a slowly changing dimension (SCD), and it is often addressed by using a unique identifier for a particular version of the joined record: but has the consequence that log compaction is not possible, since all versions of the records in the table need to be retained.

#### Fault Tolerance

If a task in a batch MapReduce job fails, it can simply be started again on another machine, and the output of the failed task is discarded.

In stream processing, waiting until a task is finished before making its output visible is not an option in stream processing, because a stream is infinite and so you can never finish processing it.

One solution is to break the stream into small blocks, and treat each block like a miniature batch process. This approach is called microbatching, and it is used in Spark Streaming. The batch size is typically around one second.

A variant approach, used in Apache Flink, is to periodically generate rolling checkpoints of state and write them to durable storage. 

We need to ensure that all outputs and side effects of processing an event take effect if and only if the processing is successful.Those things either all need to happen atomically, or none of them must happen. 

An idempotent operation is one that you can perform multiple times, and it has the same effect as if you performed it only once. Even if an operation is not naturally idempotent, it can often be made idempotent with a bit of extra metadata. When writing data to an external database, we can include the offset of the message that triggered the last write with the value, thus we can check the update has already been applied, and avoid performing the same update again.

Relying on idempotence implies several assumptions: restarting a failed task must replay the same messages in the same order (a log-based message broker does this), the processing must be deterministic, and no other node may concurrently update the same value.

Any stream process that requires state — for example, any windowed aggregations (such as counters, averages, and histograms) and any tables and indexes used for joins — must ensure that this state can be recovered after a failure. One way is to keep state local to the stream processor, and replicate it periodically. Then, when the stream processor is recovering from a failure, the new task can read the replicated state and resume processing without data loss. The other options is to keep a remote database. This dependes on network vs. disk latency.