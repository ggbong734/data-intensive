_Reading notes | Completed in September 2020_

# Notes on Designing Data Intensive Applications
## [Book](https://dataintensive.net/) written by [Dr. Martin Kleppmann](http://martin.kleppmann.com/) 

Summary: Broad overview of the architecture of data systems and how to choose database for specific applications.

## Part 1 Fundamental ideas
1. Chapter 1 - Terminology
2. Chapter 2 - Data models and Query languages
3. Chapter 3 - Storage engines and Databases
4. Chapter 4 - Data Encoding or Serialization

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




