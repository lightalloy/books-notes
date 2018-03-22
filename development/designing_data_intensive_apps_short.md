Causality - The dependency between events that arises when one thing “happens
before” another thing in a system.  
Deterministic - a function that always produces the same output if you give it
the same input (no random numbers, time of the day, etc)  
Hash - A function that turns an input into a random-looking number. Same input => same number. If there's the same number for different inputs, it's a collision.  
Idempotent - an operation that can be safely retried; if it is executed more
than once, it has the same effect as if it was only executed once.  
Linearizable - Behaving as if there was only a single copy of data in the system, which is updated by atomic operations.  
Materialize - to perform a computation eagerly and write out its result, as opposed to calculating it on demand when requested.
OLAP - Online analytic processing. Access pattern characterized by aggregating
(e.g., count, sum, average) over a large number of records.  
OLTP - Online transaction processing. Access pattern characterized by fast queries that read or write a small number of records, usually indexed by key.  
Partitioning/Sharding - Splitting up a large dataset or computation that is too big for a single machine into smaller parts and spreading them across several machines.  
Quorum -- The minimum number of nodes that need to vote on an operation before it can be considered successful.  
Serializable -- A guarantee that if several transactions execute concurrently, they behave the same as if they had executed one at a time, in some serial order.  
Skew --
1) Imbalanced load across partitions, such that some partitions have lots of
requests or data, and others have much less.  
2) A timing anomaly that causes events to appear in an unexpected,
nonsequential order. 
System of record - source of truth.  
Total order -- A way of comparing things (e.g., timestamps) that allows you to always say which one of two things is greater and which one is lesser.  
Two-phase commit (2PC) -- An algorithm to ensure that several database nodes either all commit or all abort a transaction.  
Two-phase locking (2PL) -- An algorithm for achieving serializable isolation that works by a transaction acquiring a lock on all data it reads or writes, and holding the lock until the end of the transaction.  

Summaries:
### Chapter 1. Reliable, Scalable, and Maintainable Applications
Functional requirements: (what it should do, such as allowing data to be
stored, retrieved, searched, and processed in various ways).
Nonfunctional requirements (general properties like security, reliability,
compliance, scalability, compatibility, and maintainability).
Reliability -- making systems work correctly, even when faults occur.
Scalability -- having strategies for keeping performance good, even when
load increases.
Maintainability -- making life better for the engineering and operations teams who need to work with the system, good abstractions, reducing complexity and helping to adopt and change the system.

### Chapter 2. Data Models and Query Languages
Data started out being represented as one big tree (the hierarchical model), not good for representing many-to-many relationships.
The relational model was invented to solve that problem.
Later - nonrelational “NoSQL” datastores:
1) Document databases -- data comes in self-contained documents and relationships between one document and another are rare.
2) Graph databases -- designed for cases, where anything is potentially related to everything.

Document and graph databases typically don’t enforce a schema for the data they store, which can make it easier to adapt applications to changing requirements.
But the app still assumes some structure, so it’s just a question of whether the schema is explicit (enforced on write) or implicit (handled on read).

Other data models:
- specialized genome database software like GenBank (dealing with very long strings, representing DNA)
- custom solutions for projects like the Large Hadron Collider (LHC)
- Full-text search -- arguably a kind of data model, used alongside databases.

### Chapter 3. Storage and Retrieval
Storage engines fall into two broad categories:
1) OLTP (optimized for transaction processing) -- user-facing, huge volume of requests, touch a small number of records in each query. Use some kind of key, the storage engine uses an index to find the data for the requested key. Bottleneck - disk seek time.
2) OLAP (optimized for analytics) -- Data warehouses and similar analytic systems, used by business analysts.
Handle lower volume of queries, but each query requires many millions of records. Bottleneck -- disk bandwidth (not seek time). Column-oriented storage are popular.

OLTP:
1) The log-structured school (appending to files, never update) -- Bitcask,
SSTables, LSM-trees, LevelDB, Cassandra, HBase, Lucene.
2) Update-in-place -- B-trees, used in all major relational databases and also many nonrelational ones.

OLAP: it becomes important to encode data very compactly, to minimize the amount of data that the query needs to read from disk.

### Chapter 4. Encoding and Evolution
The problem of rolling upgrade without downtime.
It is important that all data flowing around the system is encoded in a way that provides backward compatibility (new code can read old data) and
forward compatibility (old code can read new data).
Data encoding formats and their compatibility properties:
1) Programming language–specific encodings, often fail to provide forward and backward compatibility.
2) Textual formats like JSON, XML, and CSV, compatibility depends on how you use them. Somewhat vague about datatypes, you have to be careful.
3) Binary schema–driven formats like Thrift, Protocol Buffers, and Avro, allow compact, efficient encoding with clearly defined forward and backward compatibility semantics. Data needs to be decoded before it is human-readable.

Modes of dataflow:
- Databases (encoding/decoding by the db)
- RPC and REST APIs (encoding/decoding by a server and a client)
- Asynchronous message passing (using message brokers or actors, encoded by the
sender and decoded by the recipient)

### Chapter 5. Replication
Replication - keeping a copy of the same data on several machines.
Purposes:
- High availability - keeping the system working even if one machine (or several machines, or an entire datacenter) goes down
- Allowing an application to continue working when there is a network
interruption
- Latency - Placing data geographically close to users
- Scalability - Being able to handle a higher volume of reads than a single machine could handle, by performing reads on replicas.

Requires carefully thinking about concurrency, unavailable nodes and network interruptions.

Approaches:
- Single-leader replication
All writes are sent to a single node (the leader), which sends a stream of
data change events to the other replicas (followers). Reads are performed from any replica (but might be stale).
- Multi-leader replication
Clients send each write to one of several leader nodes, any of which can
accept writes. The leaders send streams of data change events to each other
and to any follower nodes.
- Leaderless replication
Clients send each write to several nodes, and read from several nodes in
parallel in order to detect and correct nodes with stale data.

Replication can be synchronous or asynchronous.
Сonsistency models for behaving under replication lag:
- Read-after-write consistency (Users should always see data that they submitted themselves.)
- Monotonic reads (After users have seen the data at one point in time, they shouldn’t later see the data from some earlier point in time.)
- Consistent prefix reads (Users should see the data in a state that makes causal sense: for example, seeing a question and its reply in the correct order.)

### Chapter 6. Partitioning
Necessary when the amount of data is too big for a single machine.
Goal -- spread the data and query load evenly across multiple machines, avoiding hot spots.
Approaches:
- Key range partitioning. Keys are sorted, and a partition owns all the
keys from some minimum up to some maximum.
Efficient range queries, but the risk of hot spots.
Partitions are typically rebalanced dynamically by splitting range into 2 ranges.
- Hash partitioning. A hash function is applied to each key, and a
partition owns a range of hashes. Fixed number of partitions in advance, moving entire partitions if the new node is added.
- Hybrid approaches (compound key - one part of the key to identify the partition and another part for the sort order).

A secondary index also needs to be partitioned:
- Document-partitioned indexes (local indexes). Only a single partition needs to be updated on write, but a read of the secondary index requires a scatter/gather across all partitions.
- Term-partitioned indexes (global indexes), the secondary indexes are
partitioned separately, using the indexed values. On write several partitions of the secondary index need to be updated; a read can be served from a single partition.

### Chapter 7. Transactions
Transactions are an abstraction layer that allows an application to pretend that certain concurrency problems and certain kinds of hardware and software
faults don’t exist.
Transaction help to not end up with inconsistent data, e.g. keep denormalized data in sync.
Isolation levels: read commited, snapshot isolation, serializable.
Dirty reads - one client reads another client’s writes before they have been committed (read commited prevents this)
Dirty writes - one client overwrites data that another client has written, but not yet committed (Almost all transaction implementations prevent this)
Read skew (nonrepeatable reads) - A client sees different parts of the database at different points in time (prevented with snapshot isolation, usually implemented with multi-version concurrency control (MVCC).)
Lost updates - Two clients concurrently perform a read-modify-write cycle.
One overwrites the other ’s write without incorporating its changes, so data is lost. Some implementations of snapshot isolation prevent this, other require manual lock (SELECT FOR UPDATE).
Write skew - A transaction reads smth, makes a desicion based on that data, makes write, but when the write is commited the prev.read data is no longer true. Only serializable isolation prevents this anomaly.
Phantom reads - A transaction reads objects that match some search condition. Another client makes a write that affects the results of that search.
Snapshot isolation prevents straightforward phantom reads, phantoms in the context of write skew require special treatment.
Only serializable isolation protects against all of these issues.
Approaches for serializability:
- Literally executing transactions in a serial order
- Two-phase locking (bad performance)
- Serializable snapshot isolation (SSI) - uses an optimistic approach, allowing transactions to proceed without blocking. When a transaction wants to commit, it is checked, and it is aborted if the execution was not serializable.

### Chapter 8. The Trouble with Distributed Systems
- Packets sent over the network may be lost or arbitrarily delayed. The response may be lost to, so you have no idea whether the message got through.
- A node’s clock may be significantly out of sync with other nodes
- A process may pause for a substantial amount of time at any point in its
execution (e.g. garbage collector) and be considered dead by other nodes.

In distributed systems, we try to build tolerance of partial failures into software, so that the system still functions while its parts are broken.

Detecting faults is hard: timeouts can’t distinguish between network and node failures, and variable network delay sometimes causes a node to be falsely suspected of crashing.

Major decisions cannot be safely made by a single node, so we require
protocols that enlist help from other nodes and try to get a quorum to agree.

It is possible to give hard real-time response guarantees and bounded delays in networks, but doing so is very expensive and results in lower utilization of hardware resources. Most non-safety-critical systems choose cheap and unreliable over expensive and reliable.

### Chapter 9. Consistency and Consensus
Linearizability - a consistency model, its goal is to make replicated data appear as though there were only a single copy, and to make all operations act on it atomically.
Easy to understand but slow.
Casuality -- imposes an ordering on events in a system (based on cause and effect). It's a weaker consistency model: some things can be concurrent, so the version history is like a timeline with branching and merging. Much less sensitive to network problems.
Achieving consensus means deciding something in such a way that all nodes agree on what was decided, and such that the decision is irrevocable.
A wide range of problems are actually reducible to consensus and are equivalent to each other.
- Linearizable compare-and-set registers
- Atomic transaction commit - wether to commit or abort
- Total order broadcast - decide the order
- Locks and leases - which client has successfully acquired a lock
- Membership/coordination service - which nodes are alive
- Uniqueness constraint - which one to allow and which should fail with a constraint violation

Single node or single leader -- the leader makes all the decisions, but if the leader fails, the whole system is unable to do any progress.
Ways to handle this situation:
- Wait for the leader to recover
- Manually fail over by getting humans to choose a new leader node and
reconfigure the system to use it
- Use an algorithm to automatically choose a new leader

Although a single-leader database can provide linearizability without executing
a consensus algorithm on every write, it still requires consensus to maintain its leadership and for leadership changes.

Tools like ZooKeeper play an important role in providing an “outsourced”
consensus, failure detection, and membership service that applications can use.

Not every system necessarily requires consensus: for example, leaderless and multi-leader replication systems typically do not use global consensus.

### Chapter 10. Batch Processinфаа
The design philosophy of Unix tools such as awk, grep, and sort affected MapReduce dataflow engines.
Principles:  
- inputs are immutable  
- outputs are intended to become the input to another (as yet
unknown) program  
- complex problems are solved by composing small tools that “do one thing well.”  
Dataflow engines add their own pipe-like data transport mechanisms to avoid materializing intermediate state to the distributed filesystem. The initial input and final output of a job is still usually HDFS.  
The two main problems that distributed batch processing frameworks need to
solve are:
- Partitioning
Mappers are partitioned according to input file blocks. The purpose is to bring all the related data—e.g., all the records with the same key—together in the same place.
Post-MapReduce dataflow engines try to avoid sorting unless it is required,
but they otherwise take a broadly similar approach to partitioning.
- Fault tolerance
Dataflow engines perform less materialization of intermediate state and keep more in memory, which means that they need to recompute more data if a node fails. Deterministic operators reduce the amount of data that needs to be recomputed.

Join algorithms:
- Sort-merge joins
- Broadcast hash joins
- Partitioned hash joins

Distributed batch processing engines have a deliberately restricted
programming model: callback functions (such as mappers and reducers) are
assumed to be stateless and to have no externally visible side effects besides
their designated output.

The framework can guarantee that the final output of a job is the same as if no faults had occurred, even though in reality various tasks perhaps had to be retried.
Batch processing job reads some input, produces some output data, without modifying the input, the output is derived from the input.
The data is bounded, it has a known, fixed size.

### Chapter 11. Stream Processing
Stream processing is very much like the batch processing, but run on the unbounded data. From this perspective, message brokers and event logs serve as the streaming equivalent of a filesystem.

2 types of message brokers:
1) AMQP/JMS-style message broker
The broker assigns individual messages to consumers, and consumers acknowledge individual messages when they have been successfully processed. Messages are deleted from the broker once they have been acknowledged.
This approach is appropriate as an asynchronous form of RPC -- task queue, where the order is not important and there's no need to go back.
2) Log-based message broker
The broker assigns all messages in a partition to the same consumer node,
and always delivers messages in the same order. Parallelism is achieved through partitioning. The broker retains messages on disk.

The log-based approach has similarities to the replication logs found in databases and log-structured storage engines.
It can also be useful to think of the writes to a database as a stream: we can capture the changelog.
Representing databases as streams opens up powerful opportunities for integrating systems. You can keep derived data systems such as search indexes, caches, and analytics systems continually up to date by consuming the log of
changes and applying them to the derived system.

Purposes of stream processing: searching for event patterns (complex event processing), computing windowed aggregations (stream analytics), keeping derived data systems up to date (materialized views).

Types of joins in stream processing:
- Stream-stream joins - 2 activity events streams (the join operator searches for related events that occur within some window of time.)
- Stream-table joins - One input stream consists of activity events, while the other is a database changelog. For each activity event, the join operator queries the database and outputs an enriched activity event.
- Table-table joins - Both input streams are database changelogs. Every change on one side is joined with the latest state of the other side.

A finer-grained recovery mechanism can be used, based on microbatching, checkpointing, transactions, or idempotent writes -- for achieving fault tolerance and exactly-once semantics in a stream processor.

### Chapter 12. The Future of Data Systems
Certain systems are designated as systems of record, and other data is derived from them through transformations.
In this way we can maintain indexes, materialized views, machine learning models, statistical summaries, and more.
By making these derivations and transformations asynchronous and loosely coupled, a problem in one area is prevented from spreading to unrelated parts of the system, increasing the robustness and fault-tolerance of the system as a whole.
If you want to change one of the processing steps, for example to change the structure of an index or cache, you can just rerun the new transformation code on the whole input dataset in order to rederive the output. If something goes wrong, you can fix the code and reprocess the data in order to recover.
Derived state can be updated by observing changes in the underlying data.
The derived state itself can further be observed by downstream consumers.
We can even build user interfaces that dynamically update to reflect data changes and continue to work offline.

Strong integrity guarantees can be implemented scalably with asynchronous event processing, by using end-to-end operation identifiers to make operations idempotent and by checking constraints asynchronously.
Clients can either wait until the check has passed, or go ahead without waiting but risk having to apologize about a constraint violation. This approach is much more scalable and robust than the traditional approach of using distributed transactions, and fits with how many business processes work in practice.
By structuring applications around dataflow and checking constraints asynchronously, we can avoid most coordination and create systems that maintain integrity but still perform well, even in geographically distributed scenarios and in the presence of faults.
























































































