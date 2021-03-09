---
layout: post
title:  "Design Data-Intensive Applications"
categories: cloud
tags: big-data design
excerpt: "Reading notes for the book talking about what aspects to think from during data-intensive application design"
---

* content
{:toc}

## C0. Trends

We are seeing some trends in the area of software applications:

* Big companies created new tools
* Short development cycles and flexible data model
* From open source to commercial
* Increasing parallelism
* Cloud infrastructure
* High availability requirement
* **Data intensive**: high quantity, high complexity, high changing speed

## C1. Reliable, Scalable, Maintainable Applications

Applications are to:

* Store data so that they, or another application, can find it again later (databases)
* Remember the result of an expensive operation, to speed up reads (caches)
* Allow users to search data by keyword or filter it in various ways (search indexes)
* Send a message to another process, to be handled asynchronously (stream processing)
* Periodically crunch a large amount of accumulated data (batch processing)

> Thinking: We will explore what different tools have in common, what distinguishes them, and how they achieve their characteristics.

We need to take these into considerations: access patterns, cross-category tools, data system of multiple tools

> Thinking: How do you ensure that the data remains correct and complete, even when things go wrong internally? How do you provide consistently good performance to clients, even when parts of your system are degraded? How do you scale to handle an increase in load? What does a good API for the service look like?

Factors that may influence the design of a data system:

* including the skills and experience of the people involved
* legacy system dependencies
* the timescale for delivery
* your organization’s tolerance of different kinds of risk
* regulatory constraints
* etc

### 1.1 Reliability: making systems work correctly, even when faults occur

The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software faults, and even human error).

A fault is not the same as a failure, best to design fault-tolerance mechanisms that prevent faults from causing failures

* Hardware faults: random and independent from each other
 -> add redundancy, using software fault-tolerance techniques in preference or in addition to hardware redundancy to tolerate the loss of entire machines and also bringing operational advantages.
* Software systematic faults: software is making some kind of assumption about its environment, faults are triggered by an unusual set of circumstances
 -> carefully thinking about assumptions and interactions in the system; thorough testing; process isolation; allowing processes to crash and restart; measuring, monitoring, and analyzing system behavior in production.
* Human errors: humans are known to be unreliable
 -> well-designed API but not too strict, sandbox environment, thoroughly test, roll-back mechanism, monitoring

### 1.2 Scalability: having strategies for keeping performance good, even when load increases

As the system grows (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth.

Here are some aspects:

* Load parameters:
 For example, in Twitter, it is the distribution of followers per user.
 An architecture that is appropriate for one level of load is unlikely to cope with 10 times that load. So, rethink your architecture on every order of magnitude load increase.

* Change in resource and performance: throughput, latency
 Check the response time distribution, high percentiles, service level objectives (SLOs) and service level agreements (SLAs). Sometimes, slower responses can be more important, and client-side latency is more important.

Key concepts:
* scaling up (vertical scaling, moving to a more powerful machine)
* scaling out (horizontal scaling, distributing the load across multiple smaller machines)
* elastic (automatically add computing resources when they detect a load increase)
* scaled manually (a human analyzes the capacity and decides to add more machines to the system)

There is no one-fit-all solution: the problem may be the volume of reads, the volume of writes, the volume of data to store, the complexity of the data, the response time requirements, the access patterns, or (usually) some mixture of all of these plus many more issues.

### 1.3 Maintainability: making life better for the engineering and operations teams

Over time, many different people will work on the system (engineering and operations, both maintaining current behavior and adapting the system to new use cases), and they should all be able to work on it productively.

* Operability: making routine tasks easy -> good monitoring, standard tools, avoiding dependency on individual machines, good runbook, good default behavior and freedom to override, self-healing and manual control, exhibiting predictable behavior

* Simplicity: there are various possible symptoms of complexity: explosion of the state space, tight coupling of modules, tangled dependencies, inconsistent naming and terminology, hacks aimed at solving performance problems, special-casing to work around issues elsewhere, and many more. -> One of the best tools we have for removing accidental complexity is abstraction.

* Evolvability: Agile working patterns, test-driven development (TDD) and refactoring

### Summary

There are functional requirements (what it should do, such as allowing data to be stored, retrieved, searched, and processed in various ways), and some nonfunctional requirements (general properties like security, reliability, compliance, scalability, compatibility, and maintainability)

## C2. Data Models and Query Languages

Data model: profound effect on how the software is written and how we think about the problem that we are solving -> important to choose one that is appropriate to the application

### Relational model and document model

The best-known data model today is probably that of **SQL**, based on the relational model proposed by Edgar Codd in 1970: data is organized into relations (called tables in SQL), where each relation is an unordered collection of tuples (rows in SQL).

The use cases appear mundane from today’s perspective: typically transaction processing (entering sales or banking transactions, airline reservations, stock-keeping in warehouses) and batch processing (customer invoicing, payroll, reporting). Relational databases turned out to generalize very well, beyond their original scope of business data processing, to a broad variety of use cases. Much of what you see on the web today is still powered by relational databases.

**NoSQL** now have driving forces: greater scalability including very large datasets or very high write throughput, free and open source software, specialized query operations, more dynamic and expressive data model.

If data is stored in relational tables, an awkward translation layer is required between the objects in the application code and the database model of tables, rows, and columns.
Then we have Object-relational mapping (ORM) frameworks like ActiveRecord and Hibernate.
But that is not enough for one-to-many relationship and we need to

* separate tables with foreign key references
* database-supported nested structure
* document in a text column

Removing such duplication is the key idea behind **normalization** in databases: normalizing this data requires many-to-one relationships.

Representing many-to-one and many-to-many relationships, relational and document databases are not fundamentally different: in both cases, the related item is referenced by a unique identifier, which is called a foreign key in the relational model and a document reference in the document model. In such cases, using a document model can lead to significantly more complex application code and worse performance.

**Document database**: A more accurate term is schema-on-read (the structure of the data is implicit, and only interpreted when the data is read), in contrast with schema-on-write (the traditional approach of relational databases, where the schema is explicit and the database ensures all written data conforms to it)

Schemaless documents can be a much more natural data model. But in cases where all records are expected to have the same structure, schemas are a useful mechanism for documenting and enforcing that structure.

The locality advantage only applies if you need large parts of the document at the same time.

Most relational database systems (other than MySQL) have supported XML since the mid-2000s. This includes functions to make local modifications to XML documents and the ability to index and query inside XML documents, which allows applications to use data models very similar to what they would do when using a document database.

A hybrid of the relational and document models is a good route for databases to take in the future.

### Query Languages for Data

SQL is a declarative query language (followed the structure of the relational algebra fairly closely), whereas IMS and CODASYL queried the database using imperative code.

1. A declarative query language is attractive because it is typically more concise and easier to work with than an imperative API. But more importantly, it also hides implementation details of the database engine, which makes it possible for the database system to introduce performance improvements without requiring any changes to queries.
2. The SQL example doesn’t guarantee any particular ordering, and so it doesn’t mind if the order changes.
3. Declarative languages often lend themselves to parallel execution.

MapReduce is a programming model for processing large amounts of data in bulk across many machines, popularized by Google.

Being able to use JavaScript code in the middle of a query is a great feature for advanced queries.

### Graph-Like Data Models

As the connections within your data become more complex, it becomes more natural to start modeling your data as a graph.

* **property graph** model (implemented by Neo4j, Titan, and InfiniteGraph): You can think of a graph store as consisting of two relational tables, one for vertices and one for edges. Cypher is a declarative query language for property graphs, created for the Neo4j graph database.

* **triple-store** model (implemented by Datomic, AllegroGraph, and others): In a triple-store, all information is stored in the form of very simple three-part statements: (subject, predicate, object). Data can be written as triples in a format called Turtle, a subset of Notation3 (N3).

## C3. Storage and Retrieval

You do need to select a storage engine that is appropriate for your application, from the many that are available. In order to tune a storage engine to perform well on your kind of workload, you need to have a rough idea of what the storage engine is doing under the hood.
In particular, there is a big difference between storage engines that are optimized for transactional workloads and those that are optimized for analytics.

### log

Logs are incredibly useful (log is used in the more general sense: an append-only sequence of records).
The general idea behind indexes is to keep some additional metadata on the side, which acts as a signpost and helps you to locate the data you want. An index is an additional structure that is derived from the primary data. Maintaining additional structures incurs overhead, especially on writes. This is an important trade-off in storage systems: well-chosen indexes speed up read queries, but every index slows down writes.

* compaction: We can then perform compaction on these segments. Compaction means throwing away duplicate keys in the log, and keeping only the most recent update for each key. We can also merge several segments together at the same time as performing the compaction.
* binary file format with string length encoded
* deletion record (tombstone)
* store the index snapshot in disk
* checksum
* single write, concurrent reads

#### hash table index

Pros:

* fast read
* easier concurrency and crash recovery
* no fragmented data files

Cons:

* limitation of hash map index
* hard to do range queries

#### Optimization to hash index

The sequence of key-value pairs is sorted by key - Sorted String Table, or SSTable for short.

* merging segments is simple and efficient (mergesort).
 The most common options are size-tiered and leveled compaction for the order and timing of how SSTables are compacted and merged.  In size-tiered compaction, newer and smaller SSTables are successively merged into older and larger SSTables. In leveled compaction, the key range is split up into smaller SSTables and older data is moved into separate “levels,” which allows the compaction to proceed more incrementally and use less disk space.
 This will cause write amplification
* sparse in-memory index to files and then linear search in the file, storage engines often use additional Bloom filters in addition
* possible to group those records into a block and compress it before writing it to disk
* maintaining a sorted structure in memory (memtable) and write it out to disk as an SSTable file when the size is big

Log-Structured Merge-Tree (or LSM-Tree), Storage engines that are based on this principle of merging and compacting sorted files are often called LSM storage engines. Lucene, an indexing engine for full-text search used by Elasticsearch and Solr, uses a similar method for storing its term dictionary.

In order to avoid that database crashes, we can keep a separate log on disk to which every write is immediately appended.

LSM-trees are typically faster for writes, and have these behaviors:

* sometimes have lower write amplification;
* they sequentially write compact SSTable files rather than having to overwrite several pages in the tree (magnetic hard drives);
* can be compressed better;
* but the bigger the database gets, the more disk bandwidth is required for compaction. If write throughput is high and compaction is not configured carefully, it can happen that compaction cannot keep up with the rate of incoming writes;

#### B-tree

B-trees is the most widely used indexing structure. It breaks the database down into fixed-size blocks or pages, traditionally 4 KB in size (sometimes bigger), and read or write one page at a time. This design corresponds more closely to the underlying hardware, as disks are also arranged in fixed-size blocks.

This algorithm ensures that the tree remains balanced: a B-tree with n keys always has a depth of O(log n). Most databases can fit into a B-tree that is three or four levels deep, so you don’t need to follow many page references to find the page you are looking for. (A four-level tree of 4 KB pages with a branching factor of 500 can store up to 256 TB.)

The basic underlying write operation of a B-tree is to overwrite a page on disk with new data.

* In order to make the database resilient to crashes, it is common for B-tree implementations to include an additional data structure on disk: a write-ahead log (WAL, also known as a redo log) or copy-on-write scheme.
* Concurrency control is typically done by protecting the tree’s data structures with latches (lightweight locks).
* Abbreviating the key to pack more keys into a page to allow the tree to have a higher branching factor, and thus fewer levels.

B-trees are thought to be faster for reads and B-trees can be more predictable (no background tasks).

* each key exists in exactly one place in the index
* offer strong transactional semantics

B-tree storage engines leave some disk space unused due to fragmentation

#### others

The heap file approach is common because it avoids duplicating data when multiple secondary indexes are present: each index just references a location in the heap file, and the actual data is kept in one place.

A compromise between a clustered index (storing all row data within the index) and a nonclustered index (storing only references to the data within the index) is known as a covering index or index with included columns, which stores some of a table’s columns within the index.

More commonly, specialized spatial indexes such as R-trees are used.

To cope with typos in documents or queries, Lucene is able to search text for words within a certain edit distance.
In Lucene, the in-memory index is a finite state automaton over the characters in the keys, similar to a trie. This automaton can be transformed into a Levenshtein automaton, which supports efficient search for words within a given edit distance.

#### In-memory database

As RAM becomes cheaper, the cost-per-gigabyte argument is eroded. Many datasets are simply not that big, so it’s quite feasible to keep them entirely in memory, potentially distributed across several machines. This has led to the development of in-memory databases. They can be faster because they can avoid the overheads of encoding in-memory data structures in a form that can be written to disk.

The disk is merely used as an append-only log for durability, and reads are served entirely from memory. Writing to disk also has operational advantages: files on disk can easily be backed up, inspected, and analyzed by external utilities.

An in-memory database architecture could be extended to support datasets larger than the available memory, without bringing back the overheads of a disk-centric architecture。 The so-called anti-caching approach works by evicting the least recently used data from memory to disk when there is not enough memory, and loading it back into memory when it is accessed again in the future. This is similar to what operating systems do with virtual memory and swap files

#### Access pattern

An application typically looks up a small number of records by some key, using an index. Records are inserted or updated based on the user’s input. Because these applications are interactive, the access pattern became known as online transaction processing (OLTP). -> SQL
> OLTP systems are typically user-facing, which means that they may see a huge volume of requests. In order to handle the load, applications usually only touch a small number of records in each query. The application requests records using some kind of key, and the storage engine uses an index to find the data for the requested key. Disk seek time is often the bottleneck here.

Usually an analytic query needs to scan over a huge number of records, only reading a few columns per record, and calculates aggregate statistics (such as count, sum, or average) rather than returning the raw data to the user. It has been called online analytic processing (OLAP). 
##### data warehouse

Data is extracted from OLTP databases (using either a periodic data dump or a continuous stream of updates), transformed into an analysis-friendly schema, cleaned up, and then loaded into the data warehouse. This process of getting data into the warehouse is known as Extract–Transform–Load (ETL)

Data warehouse vendors such as Teradata, Vertica, SAP HANA, and ParAccel typically sell their systems under expensive commercial licenses. Amazon RedShift is a hosted version of ParAccel. More recently, a plethora of open source SQL-on-Hadoop projects have emerged; they are young but aiming to compete with commercial data warehouse systems. These include Apache Hive, Spark SQL, Cloudera Impala, Facebook Presto, Apache Tajo, and Apache Drill. Some of them are based on ideas from Google’s Dremel.

In a typical data warehouse, tables are often very wide: fact tables often have over 100 columns, sometimes several hundred; a typical data warehouse query only accesses 4 or 5 of them at one time.
Optimization: column-oriented storage: a query only needs to read and parse those columns that are used in that query:
* column-oriented storage often lends itself very well to compression. An update-in-place approach is not possible with compressed columns.
* one technique that is particularly effective in data warehouses is bitmap encoding: Bitmap indexes such as these are very well suited for the kinds of queries that are common in a data warehouse: where conditions with bitwise operations applied.
* besides reducing the volume of data that needs to be loaded from disk, column-oriented storage layouts are also good for making efficient use of CPU cycles: vectorized processing, making use of single-instruction-multi-data (SIMD) instructions in modern CPUs

Materialized views are not often used in OLTP databases. In read-heavy data warehouses they can make more sense. The advantage of a materialized data cube is that certain queries become very fast because they have effectively been precomputed. The disadvantage is that a data cube doesn’t have the same flexibility as querying the raw data.

Data warehouses and similar analytic systems are less well known, because they are primarily used by business analysts, not by end users. They handle a much lower volume of queries than OLTP systems, but each query is typically very demanding, requiring many millions of records to be scanned in a short time. Disk bandwidth (not seek time) is often the bottleneck here, and column-oriented storage is an increasingly popular solution for this kind of workload.

## C4. Encoding and Evolution

### Encoding data

Old and new versions of the code, and old and new data formats, may potentially all coexist in the system at the same time. In order for the system to continue running smoothly, we need to maintain compatibility in both directions:
Backward compatibility: Newer code can read data that was written by older code.
Forward compatibility: Older code can read data that was written by newer code.

The translation from the in-memory representation to a byte sequence is called encoding (also known as serialization or marshalling), and the reverse is called decoding (parsing, deserialization, unmarshalling).
Many programming languages come with built-in support for encoding in-memory objects into byte sequences. Many third-party libraries also exist.
> Cons: cannot read the data in another language; the decoding process needs to be able to instantiate arbitrary classes (source of security); versioning; efficiency

JSON, XML, and CSV are textual formats, and thus somewhat human-readable.
> Cons: ambiguity around the encoding of numbers; large numbers; support for binary strings (e.g. Base64); no strong schema; verbose and use a lot of space -> a profusion of binary encodings for JSON have been adopted in various niches, but none of them are as widely adopted as the textual versions of JSON and XML.

Binary encodings based on schemas have a number of nice properties:
> more compact; the schema is a valuable form of documentation; keeping a database of schemas allows you to check forward and backward compatibility of schema changes before anything is deployed; for statically typed programming languages, the ability to generate code from the schema enables type checking at compile time.

Apache Thrift and Protocol Buffers (protobuf) are binary encoding libraries that are based on the same principle. Protocol Buffers was originally developed at Google, Thrift was originally developed at Facebook, and both were made open source in 2007–08. Both Thrift and Protocol Buffers require a schema for any data that is encoded.

> Each field is identified by its tag number and annotated with a datatype. If a field value is not set, it is simply omitted from the encoded record. If old code (which doesn’t know about the new tag numbers you added) tries to read data written by new code, including a new field with a tag number it doesn’t recognize, it can simply ignore that field. As long as each field has a unique tag number, new code can always read old data, because the tag numbers still have the same meaning.

Apache Avro is another binary encoding format that is interestingly different from Protocol Buffers and Thrift. It was started in 2009 as a subproject of Hadoop, as a result of Thrift not being a good fit for Hadoop’s use cases.
> The key idea with Avro is that the writer’s schema and the reader’s schema don’t have to be the same—they only need to be compatible.
> Avro is friendlier to dynamically generated schemas. Avro provides optional code generation for statically typed programming languages, but it can be used just as well without any code generation. This property is especially useful in conjunction with dynamically typed data processing languages

### Data flows between processes

Databases: The encoding formats discussed previously support such preservation of unknown fields, but sometimes you need to take care at an application level. Data outlives code.

Archival storage: As the data dump is written in one go and is thereafter immutable, formats like Avro object container files are a good fit. This is also a good opportunity to encode the data in an analytics-friendly column-oriented format such as Parquet

Service calls: services expose an application-specific API that only allows inputs and outputs that are predetermined by the business logic (application code) of the service. The backward and forward compatibility properties of an RPC scheme are inherited from whatever encoding it uses.

* REST is not a protocol, but rather a design philosophy that builds upon the principles of HTTP. It emphasizes simple data formats, using URLs for identifying resources and using HTTP features for cache control, authentication, and content type negotiation. REST has been gaining popularity, at least in the context of cross-organizational service integration, and is often associated with microservices. An API designed according to the principles of REST is called RESTful. A definition format such as OpenAPI, also known as Swagger, can be used to describe RESTful APIs and produce documentation.
	> RESTful APIs most commonly use JSON (without a formally specified schema) for responses, and JSON or URI-encoded/form-encoded request parameters for requests.
	>
	> API versioning: For RESTful APIs, common approaches are to use a version number in the URL or in the HTTP Accept header. For services that use API keys to identify a particular client, another option is to store a client’s requested API version on the server and to allow this version selection to be updated through a separate administrative interface.

* SOAP is an XML-based protocol for making network API requests. it aims to be independent from HTTP and avoids using most HTTP features. Instead, it comes with a sprawling and complex multitude of related standards (the web service framework, known as WS-*) that add various features.

* Remote Procedure Call (RPC) model tries to make a request to a remote network service look the same as calling a function or method in your programming language, within the same process (this abstraction is called location transparency). A network request is unpredictable. All the parameters need to be encoded into a sequence of bytes that can be sent over the network, and it is problematic with larger objects. The client and the service may be implemented in different programming languages, so the RPC framework must translate datatypes from one language into another. -> Furtures(promises), streams, service discovery. The main focus of RPC frameworks is on requests between services owned by the same organization, typically within the same datacenter.
	> Thrift, gRPC (Protocol Buffers), and Avro RPC can be evolved according to the compatibility rules of the respective encoding format.

Asynchronous message passing: somewhere between RPC and databases. They are similar to RPC in that a client’s request (usually called a message) is delivered to another process with low latency. They are similar to databases in that the message is not sent via a direct network connection, but goes via an intermediary called a message broker (also called a message queue or message-oriented middleware), which stores the message temporarily. Message brokers typically don’t enforce any particular data model—a message is just a sequence of bytes with some metadata, so you can use any encoding format.

The actor model is a programming model for concurrency in a single process. Rather than dealing directly with threads (and the associated problems of race conditions, locking, and deadlock), logic is encapsulated in actors. Each actor typically represents one client or entity, it may have some local state (which is not shared with any other actor), and it communicates with other actors by sending and receiving asynchronous messages. Message delivery is not guaranteed: in certain error scenarios, messages will be lost. Since each actor processes only one message at a time, it doesn’t need to worry about threads, and each actor can be scheduled independently by the framework. In distributed actor frameworks, this programming model is used to scale an application across multiple nodes. The same message-passing mechanism is used, no matter whether the sender and recipient are on the same node or different nodes. If they are on different nodes, the message is transparently encoded into a byte sequence, sent over the network, and decoded on the other side.
> <https://www.jianshu.com/p/449850aa8e82>

## C5. Replication

Replication for:

* To keep data geographically close to your users (and thus reduce access latency)
* To allow the system to continue working even if some of its parts have failed (and thus increase availability)
* To scale out the number of machines that can serve read queries (and thus increase read throughput)

### Leaders and Followers

Three popular algorithms for replicating changes between nodes: single-leader, multi-leader, and leaderless replication.

1. The most common solution for this is called leader-based replication (also known as active/passive or master–slave replication)

	An important detail of a replicated system is whether the replication happens synchronously, asynchronously, semi-synchronous (always one sync and others async) or chain replication.

	The advantage of synchronous replication is that the follower is guaranteed to have an up-to-date copy of the data that is consistent with the leader.

	The disadvantage of synchronous replication is that if the synchronous follower doesn’t respond, the write cannot be processed.

	A fully asynchronous configuration has the advantage that the leader can continue processing writes, even if all of its followers have fallen behind. This is widely used, especially if there are many followers or if they are geographically distributed.

	* New Followers
		1. snapshot; 2. copy data; 3. log replay; 4. caught up

	* Node Outages
		Follower: log replay
		Leader failover: 1. one of the followers needs to be promoted to be the new leader (by election process or controller node: bestcandidate with most up-to-date data); 2. clients need to be reconfigured to send their writes to the new leader, and the other followers need to start consuming data changes from the new leader.
			Problems:
				Inconsistency: The new leader may have received conflicting writes if it lagged behind the old leader.
				Split brain conflicts: Two leaders.
				Unnecessary failovers if timeout too short.

	* Replication Log
		* STATEMENT-BASED REPLICATION: compact but cannot handle nondeterministic cases
		* WRITE-AHEAD LOG (WAL) SHIPPING: it builds a copy of the exact same data structures as found on the leader, but coupled to the storage engine
		* LOGICAL (ROW-BASED) LOG REPLICATION: very compatible between versions and easier for external applications
		* TRIGGER-BASED REPLICATION: application-layer replication via triggers and stored procedures

	* Replication Lag
		* eventual consistency: temoporary inconsistency as  not all writes to the leader have been reflected in the follower
		* read-after-write consistency in a system with leader-based replication: logic of when to read from leader, it could be based on user, modification time, logical timestamp, geographical proximity
		* cross-device read-after-write consistency: centralized metadata, route requests to the same datacenter
		* monotonic reads: one user reads from the same replica
		* consistent prefix reads: make sure that any writes that are causally related to each other are written to the same partition
		* transactions

2. A multi-leader configuration (also known as master–master or active/active replication). In this setup, each leader simultaneously acts as a follower to the other leaders. Replication still happens in the same way: each node that processes a write must forward that data change to all the other nodes.

	* multi-datacenter operation: Within each datacenter, regular leader–follower replication is used; between datacenters, each datacenter’s leader replicates its changes to the leaders in other datacenters.
	* Pros: the inter-datacenter network delay is hidden from users; each datacenter can continue operating independently of the others; a multi-leader configuration with asynchronous replication can usually tolerate network problems better
	* Cons: the same data may be concurrently modified in two different datacenters, and those write conflicts must be resolved
	* Offline operations / Real-time collaborative editing: a local database that acts as a leader (it accepts write requests), and there is an asynchronous multi-leader replication process (sync) between the replicas
	* Write conflicts: if the application can ensure that all writes for a particular record go through the same leader, then conflicts cannot occur; resolve the conflict in a convergent way, which means that all replicas must arrive at the same final value when all changes have been replicated, using write ID or information-preserved data structure for later resolution by user.
		> As the most appropriate way of resolving a conflict may depend on the application, most multi-leader replication tools let you write conflict resolution logic using application code. That code may be executed on write or on read.
		>
		> Algorithms:
			* Conflict-free replicated datatypes
			* Mergeable persistent data structures
			* Operational transformation
	* replication topology: the communication paths along which writes are propagated from one node to another
		* all-to-all: densely connected topology provides better fault tolerance, but sometimes slower/later overtake faster/earlier -> version vectors
		* circular and star (tree): single point failure

3. In some leaderless implementations, the client directly sends its writes to several replicas, while in others, a coordinator node does this on behalf of the client.

	* When a client reads from the database, read requests are also sent to several nodes in parallel. The client may get different responses from different nodes. Version numbers are used to determine which value is newer.
		> n replicas, every write must be confirmed by w nodes to be considered successful, and we must query at least r nodes for each read -> quorum: w + r > n
		> 
		> appropriately configured quorums can tolerate the failure of individual nodes without the need for failover: high availability and low latency with only occasional stale reads
		suitable for multi-datacenter operation
	* Read repair
		> A background anti=entropy process that constantly looks for differences in the data between replicas and copies any missing data from one replica to another no in any particular order.
		> there is no fixed order in which writes are applied, which makes monitoring more difficult
	* sloppy quorum and hinted handoff: temporarily write to nodes other than the n nodes and write back to the n nodes when they are back online, it is particularly useful for increasing write availability.

	* Concurrent Writes: events may arrive in a different order at different nodes
		> last write wins (LWW): only safe when avoiding any concurrent updates to the same key
		use version numbers (per replica per key, then all replicas: version vector or vector clock) to keep concurrent values and merge them eventually

## C6. Partitioning

What we call a partition here is called a shard in MongoDB, Elasticsearch, and SolrCloud; it’s known as a region in HBase, a tablet in Bigtable, a vnode in Cassandra and Riak, and a vBucket in Couchbase.

The main reason for wanting to partition data is scalability and thus performance.

The choice of partitioning scheme is mostly independent of the choice of replication scheme.

If the partitioning is unfair, so that some partitions have more data or queries than others, we call it skewed. A partition with disproportionately high load is called a hot spot, and it could be cuased by certain access patterns.

One way of partitioning is to assign a continuous range of keys to each partition. The boundaries might be chosen manually by an administrator, or the database can choose them automatically.

> Many distributed datastores use a hash function to determine the partition for a given key.
>
> The partition boundaries can be evenly spaced, or they can be chosen pseudorandomly (in which case the technique is sometimes known as consistent hashing).

Keys that were once adjacent are now scattered across all the partitions, so their sort order is lost.
> Cassandra achieves a compromise between the two partitioning strategies. Only the first part of that key is hashed to determine the partition, but the other columns are used as a concatenated index for sorting the data in Cassandra’s SSTables.

Secondary Indexes:

* A local index (document-partitioned) -> scatter/gather: query all partitions, and combine all the results you get back
* A global index (term-partitioned) must also be partitioned, but it can be partitioned differently from the primary key index. Now a write to a single document may affect multiple partitions, and a distributed transaction is required.

The process of moving load from one node in the cluster to another is called rebalancing: balanced loads, minimum data moving (still expensive, better to have humen involved in the process), no downtime (rerouting requests)

* create many more (a fixed number of partitions) partitions than there are nodes, and you can even account for mismatched hardware in your cluster (more powerful nodes take greater shares of the load)
* create partitions dynamically: When a partition grows to exceed a configured size, it is split into two partitions; also a partition can shrink and be merged with an adjacent one. The number of partitions adapts to the total data volume.
* make the number of partitions proportional to the number of nodes to have a fixed number of partitions per node, when a new node joins the cluster, it randomly chooses a fixed number of existing partitions to split.

Service discovery

1. request handling or request routing
2. only partition-aware load balancer
3. require clients be aware of the partitioning to find the appropriate node

Achieving consensus in a distributed system -> ZooKeeper; gossip protocol

The massively parallel processing (MPP) query optimizer breaks complex warehouse queries into a number of execution stages and partitions

## C7. Transactions

Harsh reality of data systems:

* software or hardware failure
* application crash
* network interruptions
* concurrent writes
* partially updated data being read
* race conditions between clients

A transaction is a way for an application to group several reads and writes together into a logical unit. Conceptually, all the reads and writes in a transaction are executed as one operation: either the entire transaction succeeds (commit) or it fails (abort, rollback).
By using transactions, the application is free to ignore certain potential error scenarios and concurrency issues, because the database takes care of them instead (we call these safety guarantees).

The safety guarantees provided by transactions are often described by the well-known acronym ACID, which stands for Atomicity, Consistency, Isolation, and Durability. (Systems that do not meet the ACID criteria are sometimes called BASE, which stands for Basically Available, Soft state, and Eventual consistency.)

* atomicity: if a transaction was aborted, the application can be sure that it didn’t change anything, so it can safely be retried. Atomicity can be implemented using a log for crash recovery.
	> retrying: a transaction could be performed twice; it could overload the system, should  limit the number of retries, use exponential backoff; pointless for a permanent error; side effects outside of the database.
* consistency: certain statements about your data (invariants) that must always be true, this is a property of the application in the ACID sense
* isolation: concurrently executing transactions are isolated from each other, classically formalized as serializability. Isolation can be implemented using a lock on each object.
* durability: once a transaction has committed successfully, any data it has written will not be forgotten, even if there is a hardware fault or the database crashes. In a replicated database, durability may mean that the data has been successfully copied to some number of nodes. In practice, there is no one technique that can provide absolute guarantees. There are only various risk-reduction techniques.

Weak Isolation Levels

* read committed:
	* no dirty reads: only see data that has been committed when read -> remembers both the old committed value and the new value set by the transaction that currently holds the write lock
	* no dirty writes: only overwrite data that has been committed when write -> by using row-level locks
* snapshot isolation:
	each transaction reads from a consistent snapshot of the database, the transaction sees all the data that was committed in the database at the start of the transaction
	readers never block writers, and writers never block readers
	maintains several versions of an object side by side for each transaction (compaction and garbage collection later) - multi-version concurrency control (MVCC): the database makes a list of all the other transactions that are in progress (not yet committed or aborted) when each transaction starts, this can be used for condition change detection
* Corner cases:
	* preventing lost updates
		> sequentially atomic write operations: exclusive lock on the object for read and write (cursor stability); compare-and-set operation
		parallel operations with lost update detect and retry
	* prevent write skew, two operations concurrently check conditions and modify objects
		> phantoms (object to be added in the future), nothing to lock in some cases, can be solved by materializing conflicts or predicate locking (precise) or index-ranging locking (rough range, extreme case: entire table)

Serializable isolation
* single-threaded serial transaction processing, it is pessimistic to the extreme
		use stored procedure to improve performance if the whole
		use memory to improve performance
		cross-partition transactions have additional coordination overhead and are vastly slower than single-partition transactions
* interactive multi-statement transactions
		split a large transaction into multiple small ones, process multiple transactions concurrently
* two=phase locking, a pessimistic concurrency control mechanism: mutual exclusion style, writes block writes, reads and writes block each other
		quite unstable latencies, slow at high percentiles
* Serializable Snapshot Isolation (SSI), it is optimistic: on top of snapshot isolation, SSI adds an algorithm for detecting serialization conflicts among writes and determining which transactions to abort.
		performs badly if there is high contention (many transactions trying to access the same objects)
		contention can be reduced with commutative atomic operations

## C8. Distributed Systems

The difficulty is that partial failures are nondeterministic.

At one end of the scale is the field of high-performance computing (HPC). At the other extreme is cloud computing. Traditional enterprise datacenters lie somewhere between these extremes.

We must accept the possibility of partial failure and build fault-tolerance mechanisms into the software. In other words, we need to build a reliable system from unreliable components. The fault handling must be part of the software design, and you (as operator of the software) need to know what behavior to expect from the software in the case of a fault.

Shared-nothing is not the only way of building systems, but it has become the dominant approach for building internet services

The internet and most internal networks in datacenters (often Ethernet) are asynchronous packet networks. In this kind of network, one node can send a message (a packet) to another node, but the network gives no guarantees as to when it will arrive, or whether it will arrive at all.

Handling network faults doesn’t necessarily mean tolerating them: if your network is normally fairly reliable, a valid approach may be to simply show an error message to users while your network is experiencing problems.

### Timeout

Many systems need to automatically detect faulty nodes. In general you have to assume that you will get no response at all. You can retry a few times, wait for a timeout to elapse, and eventually declare the node dead if you don’t hear back within the timeout.

A long timeout means a long wait until a node is declared dead.
Prematurely declaring a node dead is problematic.

> Some latency-sensitive applications, such as videoconferencing and Voice over IP (VoIP), use UDP rather than TCP. It’s a trade-off between reliability and variability of delays: as UDP does not perform flow control and does not retransmit lost packets, it avoids some of the reasons for variable network delays (although it is still susceptible to switch queues and scheduling delays). UDP is a good choice in situations where delayed data is worthless.

A system with plenty of spare capacity can easily drain queues, whereas in a highly utilized system, long queues can build up very quickly.

* Choose timeouts experimentally.
* Systems can continually measure response times and their variability (jitter) and automatically adjust timeouts according to the observed response time distribution. This can be done with a Phi Accrual failure detector.

Ethernet and IP are packet-switched protocols, which suffer from queueing and thus unbounded delays in the network. These protocols do not have the concept of a circuit. They are optimized for bursty traffic.

### Clocks

In a distributed system, time is a tricky business, because communication is not instantaneous: it takes time for a message to travel across the network from one machine to another. Moreover, each machine on the network has its own clock, which is an actual hardware device: usually a quartz crystal oscillator. These devices are not perfectly accurate, so each machine has its own notion of time.

Modern computers have at least two different kinds of clocks: a time-of-day clock and a monotonic clock.

* A time-of-day clock does what you intuitively expect of a clock: it returns the current date and time according to some calendar (also known as wall-clock time). For example, clock_gettime(CLOCK_REALTIME) on Linuxv and System.currentTimeMillis() in Java. Time-of-day clocks are usually synchronized with NTP.
* A monotonic clock is suitable for measuring a duration (time interval), such as a timeout or a service’s response time: clock_gettime(CLOCK_MONOTONIC) on Linux and System.nanoTime() in Java. The name comes from the fact that they are guaranteed to always move forward (whereas a time-of-day clock may jump back in time). NTP may adjust the frequency at which the monotonic clock moves forward (this is known as slewing the clock) if it detects that the computer’s local quartz is moving faster or slower than the NTP server.

Hardware clocks and NTP can be fickle beasts:

* quartz clock drifts
* local clock reset
* NTP synchronization delay or misconfigured
* leap seconds
* virtualized hardware clock

Part of the problem is that incorrect clocks easily go unnoticed. Thus, if you use software that requires synchronized clocks, it is essential that you also carefully monitor the clock offsets between all the machines.

Most systems don’t expose this uncertainty. An interesting exception is Google’s TrueTime API in Spanner, which explicitly reports the confidence interval on the local clock. When you ask it for the current time, you get back two values: [earliest, latest], which are the earliest possible and the latest possible timestamp.
When a database is distributed across many machines, potentially in multiple datacenters, a global, monotonically increasing transaction ID (across all partitions) is difficult to generate, because it requires coordination.
In order to ensure that transaction timestamps reflect causality, Spanner deliberately waits for the length of the confidence interval before committing a read-write transaction. By doing so, it ensures that any transaction that may read the data is at a sufficiently later time, so their confidence intervals do not overlap.

A thread might be paused for a long time, causing gap between code lines:

* stop-the-world GC
* virtual machine suspension
* end-user device suspension
* context-switch
* slow disk I/O, swapping to disk (paging), network storage delay
* SIGSTOP - STGCONT

When writing multi-threaded code on a single machine, we have fairly good tools for making it thread-safe: mutexes, semaphores, atomic counters, lock-free data structures, blocking queues, and so on. Unfortunately, these tools don’t directly translate to distributed systems, because a distributed system has no shared memory—only messages sent over an unreliable network.

In hard real-time systems, there is a specified deadline by which the software must respond; if it doesn’t meet the deadline, that may cause a failure of the entire system. All of this requires a large amount of additional work and severely restricts the range of programming languages, libraries, and tools that can be used.

An emerging idea is to treat GC pauses like brief planned outages of a node, and to let other nodes handle requests from clients while one node is collecting its garbage. This trick hides GC pauses from clients and reduces the high percentiles of response time.
A variant of this idea is to use the garbage collector only for short-lived objects (which are fast to collect) and to restart processes periodically, before they accumulate enough long-lived objects to require a full GC of long-lived objects.

### Others

To ensure that a node that is under a false belief of being “the chosen one” cannot disrupt the rest of the system, a fairly simple technique that achieves this goal is called fencing. This mechanism requires the resource itself to take an active role in checking tokens by rejecting any writes with an older token than one that has already been processed.

A system is Byzantine fault-tolerant if it continues to operate correctly even if some of the nodes are malfunctioning and not obeying the protocol, or if malicious attackers are interfering with the network.

Algorithms need to be written in a way that does not depend too heavily on the details of the hardware and software configuration on which they are run. This in turn requires that we somehow formalize the kinds of faults that we expect to happen in a system. We do this by defining a system model, which is an abstraction that describes what things an algorithm may assume. To define what it means for an algorithm to be correct, we can describe its properties. To clarify the situation, it is worth distinguishing between two different kinds of properties: safety and liveness properties. Safety is often informally defined as nothing bad happens, and liveness as something good eventually happens. Abstract system models are incredibly helpful for distilling down the complexity of real systems to a manageable set of faults that we can reason about, so that we can understand the problem and try to solve it systematically. We can prove algorithms correct by showing that their properties always hold in some system model. Proving an algorithm correct does not mean its implementation on a real system will necessarily always behave correctly.

## C9. Consistency and Consensus

The best way of building fault-tolerant systems is to find some general-purpose abstractions with useful guarantees, implement them once, and then let applications rely on those guarantees.

Thinking: We need to understand the scope of what can and cannot be done: in some situations, it’s possible for the system to tolerate faults and continue working; in other situations, that is not possible. The limits of what is and isn’t possible have been explored in depth, both in theoretical proofs and in practical implementations.

Eventual consistency is a very weak guarantee—it doesn’t say anything about when the replicas will converge. When working with a database that provides only weak guarantees, you need to be constantly aware of its limitations and not accidentally assume too much.

### Linearizability

To make a system appear as if there were only one copy of the data, and all operations on it are atomic, is the idea behind linearizability (also known as atomic consistency, strong consistency, immediate consistency, or external consistency). In other words, linearizability is a recency guarantee on reads and writes of a register (an individual object).

> Coordination services like Apache ZooKeeper and etcd are often used to implement distributed locks and leader election. They use consensus algorithms to implement linearizable operations in a fault-tolerant way.
> A hard uniqueness constraint requires linearizability.
> Linearizability is a way of avoiding race conditions.

If your application requires linearizability, and some replicas are disconnected from the other replicas due to a network problem, then some replicas cannot process requests while they are disconnected: they must either wait until the network problem is fixed, or return an error (either way, they become unavailable). This insight is popularly known as the CAP theorem, named by Eric Brewer in 2000, although the trade-off has been known to designers of distributed databases since the 1970s. A better way of phrasing CAP would be either Consistent or Available when Partitioned.

Some distributed data systems have abandoned linearizability, which allows them to achieve better performance but can make them difficult to work with.

### Ordering

Causality imposes an ordering on events: cause comes before effect; a message is sent before that message is received; the question comes before the answer. If a system obeys the ordering imposed by causality, we say that it is causally consistent.

A total order allows any two elements to be compared. Concurrent operations are incomparable and in a partial order.

Linearizability implies causality: any system that is linearizable will preserve causality correctly.
Causal consistency is the strongest possible consistency model that does not slow down due to network delays, and remains available in the face of network failures

We can create sequence numbers in a total order that is consistent with causality: if operation A causally happened before B, then A occurs before B in the total order (A has a lower sequence number than B). Concurrent operations may be ordered arbitrarily.

A Lamport timestamp bears no relationship to a physical time-of-day clock, but it provides total ordering: if you have two timestamps, the one with a greater counter value is the greater timestamp; if the counter values are the same, the one with the greater node ID is the greater timestamp. The key idea about Lamport timestamps, which makes them consistent with causality, is the following: every node and every client keeps track of the maximum counter value it has seen so far, and includes that maximum on every request. When a node receives a request or response with a maximum counter value greater than its own counter value, it immediately increases its own counter to that maximum.
> To conclude: in order to implement something like a uniqueness constraint for usernames, it’s not sufficient to have a total ordering of operations—you also need to know when that order is finalized.

In the distributed systems literature, a problem is known as total order broadcast or atomic broadcast. Informally, it requires that two safety properties always be satisfied:

* Reliable delivery: No messages are lost: if a message is delivered to one node, it is delivered to all nodes eventually.
* Totally ordered delivery: Messages are delivered to every node in the same order, exactly once.

sequential consistency, or timeline consistency: the procedure ensures linearizable writes, it doesn’t guarantee linearizable reads

build total order broadcast from linearizable storage:  for every message you want to send through total order broadcast, you increment-and-get the linearizable integer, and then attach the value you got from the register as a sequence number to the message. You can then send the message to all nodes (resending any lost messages), and the recipients will deliver the messages consecutively by sequence number.

it can be proved that a linearizable compare-and-set (or increment-and-get) register and total order broadcast are both equivalent to consensus

### Consensus

FLP result is proved in the asynchronous system model (see “System Model and Reality”), a very restrictive model that assumes a deterministic algorithm that cannot use any clocks or timeouts. If the algorithm is allowed to use timeouts, or some other way of identifying suspected crashed nodes (even if the suspicion is sometimes wrong), then consensus becomes solvable.

Two-phase commit is an algorithm for achieving atomic transaction commit across multiple nodes—i.e., to ensure that either all nodes commit or all nodes abort.

2PC uses a new component that does not normally appear in single-node transactions: a coordinator (also known as transaction manager). The coordinator is often implemented as a library within the same application process that is requesting the transaction. Two-phase commit is called a blocking atomic commit protocol due to the fact that 2PC can become stuck waiting for the coordinator to recover.

3PC assumes a network with bounded delay and nodes with bounded response times; in most practical systems with unbounded network delay and process pauses (see Chapter 8), it cannot guarantee atomicity. In general, nonblocking atomic commit requires a perfect failure detector.

Some implementations of distributed transactions carry a heavy performance penalty.

In a heterogeneous transaction, the participants are two or more different technologies. X/Open XA (short for eXtended Architecture) is a standard for implementing two-phase commit across heterogeneous technologies. XA is not a network protocol—it is merely a C API for interfacing with a transaction coordinator. XA assumes that your application uses a network driver or client library to communicate with the participant databases or messaging services. The transaction coordinator implements the XA API.

* Many XA implementations have an emergency escape hatch called heuristic decisions: allowing a participant to unilaterally decide to abort or commit an in-doubt transaction without a definitive decision from the coordinator
* The transaction coordinator is itself a kind of database.

The consensus problem is normally formalized as follows: one or more nodes may propose values, and the consensus algorithm decides on one of those values.

* Uniform agreement: No two nodes decide differently.
* Integrity: No node decides twice.
* Validity If a node decides value v, then v was proposed by some node.
* Termination: Every node that does not crash eventually decides some value. The termination property formalizes the idea of fault tolerance. It essentially says that a consensus algorithm cannot simply sit around and do nothing forever—in other words, it must make progress. Even if some nodes fail, the other nodes must still reach a decision. In particular, 2PC does not meet the requirements for termination.

The best-known fault-tolerant consensus algorithms are Viewstamped Replication (VSR), Paxos, Raft(etcd), and Zab (Zookeeper).
Total order broadcast is equivalent to repeated rounds of consensus

All of the consensus protocols discussed so far internally use a leader in some form or another, but they don’t guarantee that the leader is unique. Instead, they can make a weaker guarantee: the protocols define an epoch number (called the ballot number in Paxos, view number in Viewstamped Replication, and term number in Raft) and guarantee that within each epoch, the leader is unique.

Before a leader is allowed to decide anything, it must first check that there isn’t some other leader with a higher epoch number which might take a conflicting decision. It must collect votes from a quorum of nodes.

Limitation of concensus:

* The process by which nodes vote on proposals before they are decided is a kind of synchronous replication.
* Consensus systems always require a strict majority to operate.
* Most consensus algorithms assume a fixed set of nodes that participate in voting, which means that you can’t just add or remove nodes in the cluster. Dynamic membership extensions to consensus algorithms allow the set of nodes in the cluster to change over time.
* Consensus systems generally rely on timeouts to detect failed nodes.

ZooKeeper and etcd are designed to hold small amounts of data that can fit entirely in memory (although they still write to disk for durability)—so you wouldn’t want to store all of your application’s data here. That small amount of data is replicated across all the nodes using a fault-tolerant total order broadcast algorithm. ZooKeeper is modeled after Google’s Chubby lock service.

* Total ordering of operations: When some resource is protected by a lock or lease, you need a fencing token to prevent clients from conflicting with each other in the case of a process pause. ZooKeeper provides this by totally ordering all operations and giving each operation a monotonically increasing transaction ID (zxid) and version number (cversion).
* Linearizable atomic operations: Using an atomic compare-and-set operation, you can implement a lock. A distributed lock is usually implemented as a lease, which has an expiry time so that it is eventually released in case the client fails.
* Failure detection: Clients maintain a long-lived session on ZooKeeper servers, and the client and server periodically exchange heartbeats to check that the other node is still alive.
* Change notifications: Not only can one client read locks and values that were created by another client, but it can also watch them for changes.
* Work allocation: ZooKeeper provides a way of “outsourcing” some of the work of coordinating nodes (consensus, operation ordering, and failure detection) to an external service.
* Service discovery: to find out which IP address you need to connect to in order to reach a particular service. (DNS is the traditional way.)
* Membership services: A membership service determines which nodes are currently active and live members of a cluster.

## C10. Data Processing

Applications thus commonly use a combination of several different datastores, indexes, caches, analytics systems, etc. and implement mechanisms for moving data from one store to another.

A system of record, also known as source of truth, holds the authoritative version of your data. Data in a derived system is the result of taking some existing data from another system and transforming or processing it in some way. A database is just a tool: how you use it is up to you.

* Services (online systems)
	A service waits for a request or instruction from a client to arrive. When one is received, the service tries to handle it as quickly as possible and sends a response back. Response time is usually the primary measure of performance of a service, and availability is often very important (if the client can’t reach the service, the user will probably get an error message).

* Batch processing systems (offline systems)
	A batch processing system takes a large amount of input data, runs a job to process it, and produces some output data. Jobs often take a while (from a few minutes to several days), so there normally isn’t a user waiting for the job to finish. Instead, batch jobs are often scheduled to run periodically (for example, once a day). The primary performance measure of a batch job is usually throughput (the time it takes to crunch through an input dataset of a certain size).

* Stream processing systems (near-real-time systems)
	Stream processing is somewhere between online and offline/batch processing (so it is sometimes called near-real-time or nearline processing). Like a batch processing system, a stream processor consumes inputs and produces outputs (rather than responding to requests). However, a stream job operates on events shortly after they happen, whereas a batch job operates on a fixed set of input data. This difference allows stream processing systems to have lower latency than the equivalent batch systems.

### Unix tools

Unix tools are very much worth learning about. Many data analyses can be done in a few minutes using some combination of awk, sed, grep, sort, uniq, and xargs, and they perform surprisingly well. The sort utility in GNU Coreutils (Linux) automatically handles larger-than-memory datasets by spilling to disk, and automatically parallelizes sorting across multiple CPU cores. Mergesort has sequential access patterns that perform well on disks.

Unix philosophy:
* Make each program do one thing well. To do a new job, build afresh rather than complicate old programs by adding new “features”.
* Expect the output of every program to become the input to another, as yet unknown, program. Don’t clutter output with extraneous information. Avoid stringently columnar or binary input formats. Don’t insist on interactive input.
* Design and build software, even operating systems, to be tried early, ideally within weeks. Don’t hesitate to throw away the clumsy parts and rebuild them.
* Use tools in preference to unskilled help to lighten a programming task, even if you have to detour to build the tools and expect to throw some of them out after you’ve finished using them.
This approach—automation, rapid prototyping, incremental iteration, being friendly to experimentation, and breaking down large projects into manageable chunks—sounds remarkably like the Agile and DevOps movements of today.

If you want to be able to connect any program’s output to any program’s input, that means that all programs must use the same input/output interface. In Unix, that interface is a file (or, more precisely, a file descriptor: an actual file on the filesystem, a communication channel to another process (Unix socket, stdin, stdout), a device driver (say /dev/audio or /dev/lp0), a socket representing a TCP connection, and so on).

Separating the input/output wiring from the program logic makes it easier to compose small tools into bigger systems.

Part of what makes Unix tools so successful is that they make it quite easy to see what is going on:
	The input files to Unix commands are normally treated as immutable. This means you can run the commands as often as you want, trying various command-line options, without damaging the input files.
	You can end the pipeline at any point, pipe the output into less, and look at it to see if it has the expected form. This ability to inspect is great for debugging.
	You can write the output of one pipeline stage to a file and use that file as input to the next stage. This allows you to restart the later stage without rerunning the entire pipeline.

### MapReduce

The main difference from pipelines of Unix commands is that MapReduce can parallelize a computation across many machines, without you having to write code to explicitly handle the parallelism. The mapper and reducer only operate on one record at a time; they don’t need to know where their input is coming from or their output is going to, so the framework can handle the complexities of moving data between machines.

In Hadoop MapReduce, the mapper and reducer are each a Java class that implements a particular interface. In MongoDB and CouchDB, mappers and reducers are JavaScript functions.

The Hadoop MapReduce framework does not have any particular support for workflows, so this chaining is done implicitly by directory name: the first job must be configured to write its output to a designated directory in HDFS, and the second job must be configured to use that same directory name for reading its input. From the MapReduce framework’s point of view, they are two independent jobs. To handle these dependencies between job executions, various workflow schedulers for Hadoop have been developed, including Oozie, Azkaban, Luigi, Airflow, and Pinball. Various higher-level tools for Hadoop, such as Pig, Hive, Cascading, Crunch, and FlumeJava, also set up workflows of multiple MapReduce stages that are automatically wired together appropriately.

Reducer-side joins:
	sort-merge join: mapper output is sorted by key, and the reducers then merge together the sorted lists of records from both sides of the join.
	records relating to the hot key need to be replicated to all reducers handling that key. This technique spreads the work of handling the hot key over several reducers, which allows it to be parallelized better, at the cost of having to replicate the other join input to multiple reducers.

Map-side joins:
	broadcast hash join: each mapper for a partition of the large input reads the entirety of the small input into a hash table
	Instead of loading the small join input into an in-memory hash table, an alternative is to store the small join input in a read-only index on the local disk
	Partitioned hash joins are known as bucketed map joins in Hive: If the partitioning is done correctly, you can be sure that all the records you might want to join are located in the same numbered partition, and so it is sufficient for each mapper to only read one partition from each of the input datasets. This approach only works if both of the join’s inputs have the same number of partitions, with records assigned to partitions based on the same key and the same hash function.
	If a map-side merge join is possible, it probably means that prior MapReduce jobs brought the input datasets into this partitioned and sorted form in the first place.

Google’s original use of MapReduce was to build indexes for its search engine, which was implemented as a workflow of 5 to 10 MapReduce jobs. Even today, Hadoop MapReduce remains a good way of building indexes for Lucene/Solr. The indexing process is very easy to reason about: documents in, indexes out.

Another common use for batch processing is to build machine learning systems such as classifiers and recommendation systems. The output of those batch jobs is often some kind of database.
	The most obvious choice might be to use the client library for your favorite database directly within a mapper or reducer, and to write from the batch job directly to the database server, one record at a time. -> slow, concurrent write overwhelm the database, partial complete
	A much better solution is to build a brand-new database inside the batch job and write it as files to the job’s output directory in the distributed filesystem. Those data files are then immutable once written, and can be loaded in bulk into servers that handle read-only queries. Since most of these key-value stores are read-only (the files can only be written once by a batch job and are then immutable), the data structures are quite simple.

The handling of output from MapReduce jobs follows the same Unix philosophy. By treating inputs as immutable and avoiding side effects (such as writing to external databases), batch jobs not only achieve good performance but also become much easier to maintain: human fault tolerance, minimizing irreversibility, automatic retry, separating logic from wiring.

Downsides:
	Having to wait until all of the preceding job’s tasks have completed slows down the execution of the workflow as a whole.
	Mappers are often redundant: they just read back the same file that was just written by a reducer, and prepare it for the next stage of partitioning and sorting.
	Storing intermediate state in a distributed filesystem means those files are replicated across several nodes, which is often overkill for such temporary data.

The biggest difference is that MPP databases focus on parallel execution of analytic SQL queries on a cluster of machines, while the combination of MapReduce and a distributed filesystem provides something much more like a general-purpose operating system that can run arbitrary programs. To put it bluntly, Hadoop opened up the possibility of indiscriminately dumping data into HDFS, and only later figuring out how to process it further.
The idea is similar to a data warehouse (see “Data Warehousing”): simply bringing data from various parts of a large organization together in one place is valuable, because it enables joins across datasets that were previously disparate. The careful schema design required by an MPP database slows down that centralized data collection; collecting data in its raw form, and worrying about schema design later, allows the data collection to be speeded up (a concept sometimes known as a “data lake” or “enterprise data hub”).
Indiscriminate data dumping shifts the burden of interpreting the data: instead of forcing the producer of a dataset to bring it into a standardized format, the interpretation of the data becomes the consumer’s problem.

MapReduce gave engineers the ability to easily run their own code over large datasets. Having two processing models, SQL and MapReduce, was not enough. And due to the openness of the Hadoop platform, it was feasible to implement a whole range of approaches, which would not have been possible within the confines of a monolithic MPP database. Crucially, those various processing models can all be run on a single shared-use cluster of machines, all accessing the same files on the distributed filesystem.

The MapReduce approach is more appropriate for larger jobs: jobs that process so much data and run for such a long time that they are likely to experience at least one task failure along the way.

Among open source cluster schedulers, preemption is less widely used. YARN’s CapacityScheduler supports preemption for balancing the resource allocation of different queues, but general priority preemption is not supported in YARN, Mesos, or Kubernetes at the time of writing.

### Others

dataflow engines: Spark, Tez, Flink
	Like MapReduce, they work by repeatedly calling a user-defined function to process one record at a time on a single thread.

Unlike in MapReduce, these functions/operators need not take the strict roles of alternating map and reduce, but instead can be assembled in more flexible ways.
	One option is to repartition and sort records by key
	Another possibility is to take several inputs and to partition them in the same way, but skip the sorting.
	For broadcast hash joins, the same output from one operator can be sent to all partitions of the join operator.

Advantages:
	Expensive work such as sorting need only be performed in places where it is actually required
	There are no unnecessary map tasks, since the work done by a mapper can often be incorporated into the preceding reduce operator
	Because all joins and data dependencies in a workflow are explicitly declared, the scheduler has an overview of what data is required where, so it can make locality optimizations
	It is usually sufficient for intermediate state between operators to be kept in memory or written to local disk
	Operators can start executing as soon as their input is ready; there is no need to wait for the entire preceding stage to finish before the next one starts.
	Existing Java Virtual Machine (JVM) processes can be reused to run new operators, reducing startup overheads

Fault tolerance:
	If a machine fails and the intermediate state on that machine is lost, it is recomputed from other data that is still available. To enable this recomputation, the framework must keep track of how a given piece of data was computed—which input partitions it used, and which operators were applied to it.
	In order to avoid such cascading faults, it is better to make operators deterministic.
	It is probably cheaper to materialize the intermediate data to files than to recompute it.

Graph:
	As an optimization for batch processing graphs, the bulk synchronous parallel (BSP) model of computation has become popular. Among others, it is implemented by Apache Giraph, Spark’s GraphX API, and Flink’s Gelly API. It is also known as the Pregel model, as Google’s Pregel paper popularized this approach for processing graphs.
	In each iteration, a function is called for each vertex, passing the function all the messages that were sent to that vertex—much like a call to the reducer. A vertex remembers its state in memory from one iteration to the next, so the function only needs to process new incoming messages. If no messages are being sent in some part of the graph, no work needs to be done. It’s a bit similar to the actor model.
	Because the programming model deals with just one vertex at a time (sometimes called “thinking like a vertex”), the framework may partition the graph in arbitrary ways.

As Tez emerged, these high-level languages had the additional benefit of being able to move to the new dataflow execution engine without the need to rewrite job code. Spark and Flink also include their own high-level dataflow APIs, often taking inspiration from FlumeJava.
These dataflow APIs generally use relational-style building blocks to express a computation: joining datasets on the value of some field; grouping tuples by key; filtering by some condition; and aggregating tuples by counting, summing, or other functions.
Besides the obvious advantage of requiring less code, these high-level interfaces also allow interactive use, in which you write analysis code incrementally in a shell and run it frequently to observe what it is doing.
There are cost-based query optimizers that the framework can analyze the properties of the join inputs and automatically decide which of the aforementioned join algorithms would be most suitable for the task at hand. This is possible if joins are specified in a declarative way: the application simply states which joins are required, and the query optimizer decides how they can best be executed.

Another domain of increasing importance is statistical and numerical algorithms, which are needed for machine learning applications such as classification and recommendation systems.

### Stream Processing

In reality, a lot of data is unbounded because it arrives gradually over time. Thus, batch processors must artificially divide the data into chunks of fixed duration. To reduce the delay, we can run the processing more frequently. Abandoning the fixed time slices entirely and simply processing every event as it happens is the idea behind stream processing. In general, a “stream” refers to data that is incrementally made available over time.

In a stream processing context,

* a record is more commonly known as an event, but it is essentially the same thing: a small, self-contained, immutable object containing the details of something that happened at some point in time.
* a timestamp indicating when it happened according to a time-of-day clock.
* an event is generated once by a producer (also known as a publisher or sender)
* processed by multiple consumers (subscribers or recipients)
* related events are usually grouped together into a topic or stream

A number of messaging systems use direct network communication between producers and consumers without going via intermediary nodes. Although these direct messaging systems work well in the situations for which they are designed, they generally require the application code to be aware of the possibility of message loss. The faults they can tolerate are quite limited.

A widely used alternative is to send messages via a message broker (also known as a message queue), which is essentially a kind of database that is optimized for handling message streams. By centralizing the data in the broker, these systems can more easily tolerate clients that come and go (connect, disconnect, and crash), and the question of durability is moved to the broker instead.

Log-based message broker: a producer sends a message by appending it to the end of the log, and a consumer receives messages by reading the log sequentially. In order to scale to higher throughput than a single disk can offer, the log can be partitioned.

We can also go in reverse: take ideas from messaging and streams, and apply them to databases. The fact that something was written to a database is an event that can be captured, stored, and processed. In fact, a replication log is a stream of database write events, produced by the leader as it processes transactions.

Change data capture (CDC) is the process of observing all data changes written to a database and extracting them in a form in which they can be replicated to other systems. CDC is especially interesting if changes are made available as a stream, immediately as they are written. Essentially, change data capture makes one database the leader (the one from which the changes are captured), and turns the others into followers. Parsing the replication log can be a more robust approach, although it also comes with challenges, such as handling schema changes. The Kafka Connect framework offers further CDC connectors for various databases.
Change data capture is usually asynchronous. This design has the operational advantage that adding a slow consumer does not affect the system of record too much, but it has the downside that all the issues of replication lag apply.

The snapshot of the database must correspond to a known position or offset in the change log, so that you know at which point to start applying changes after the snapshot has been processed.
To rebuild a derived data system such as a search index, you can start a new consumer from offset 0 of the log-compacted topic, and sequentially scan over all messages in the log.

Increasingly, databases are beginning to support change streams as a first-class interface, rather than the typical retrofitted and reverse-engineered CDC efforts.

Event sourcing, a technique that was developed in the domain-driven design (DDD) community. The application logic is explicitly built on the basis of immutable events that are written to an event log. In this case, the event store is append-only, and updates or deletes are discouraged or prohibited.
The event sourcing philosophy is careful to distinguish between events and commands. The application must first validate that it can execute the command. If the validation is successful and the command is accepted, it becomes an event, which is durable and immutable. At the point when the event is generated, it becomes a fact.
With event sourcing, you can design an event such that it is a self-contained description of a user action. The user action then requires only a single write in one place—namely appending the event to the log—which is easy to make atomic.

By separating mutable state from the immutable event log, you can derive several different read-oriented representations from the same log of events. This works just like having multiple consumers of a stream.
If the event log and the application state are partitioned in the same way, then a straightforward single-threaded log consumer needs no concurrency control for writes.

Many systems that don’t use an event-sourced model nevertheless rely on immutability: various databases internally use immutable data structures or multi-version data to support point-in-time snapshots.

In some circumstances, it’s not sufficient to just append another event to the log to indicate that the prior data should be considered deleted—you actually want to rewrite history and pretend that the data was never written in the first place. Deletion is more a matter of “making it harder to retrieve the data” than actually “making it impossible to retrieve the data.”

The one crucial difference to batch jobs is that a stream never ends.

Streaming Usage:

1. Complex event processing (CEP) is an approach developed in the 1990s for analyzing event streams, especially geared toward the kind of application that requires searching for certain event patterns.
2. The boundary between CEP and stream analytics is blurry, but as a general rule, analytics tends to be less interested in finding specific event sequences and is more oriented toward aggregations and statistical metrics over a large number of events. The time interval over which you aggregate is known as a window. Stream analytics systems sometimes use probabilistic algorithms, such as Bloom filters for set membership, HyperLogLog for cardinality estimation, and various percentile estimation algorithms.
3. In principle, any stream processor could be used for materialized view maintenance.
4. search for individual events based on complex criteria. searching a stream turns the processing on its head: the queries are stored, and the documents run past the queries
5. there is some crossover area between RPC-like systems and stream processing. It is also possible to process streams using actor frameworks.

To handle such straggler events that arrive after the window has already been declared complete:

* Ignore the straggler events
* Publish a correction, an updated value for the window with stragglers included

In some cases it is possible to use a special message to indicate, “From now on there will be no more messages with a timestamp earlier than t,” which can be used by consumers to trigger windows. The consumers might need to keep track of each producer individually. Adding and removing producers is trickier in this case.

To adjust for incorrect device clocks, you can estimate the offset between the device clock and the server clock, then apply that offset to the event timestamp, and thus estimate the true time at which the event actually occurred.

Several types of windows are in common use:

* A tumbling window has a fixed length, and every event belongs to exactly one window.
* A hopping window also has a fixed length, but allows windows to overlap in order to provide some smoothing.
* A sliding window contains all the events that occur within some interval of each other.
* A session window has no fixed duration. Instead, it is defined by grouping together all events for the same user that occur closely together in time, and the window ends when the user has been inactive for some time.

Stream joins:

* stream-stream: a stream processor needs to maintain state
* stream-table: the stream processor’s local copy of the database needs to be kept up to date
* table-table join: materializing and maintaining the views

The three types of joins described here (stream-stream, stream-table, and table-table) have a lot in common: they all require the stream processor to maintain some state (search and click events, user profiles, or follower list) based on one join input, and query that state on messages from the other join input.

If the ordering of events across streams is undetermined, the join becomes nondeterministic. In data warehouses, this issue is known as a slowly changing dimension (SCD), and it is often addressed by using a unique identifier for a particular version of the joined record. This change makes the join deterministic, but has the consequence that log compaction is not possible, since all versions of the records in the table need to be retained.

Fault tolerace:

1. One solution is to break the stream into small blocks, and treat each block like a miniature batch process. This approach is called microbatching, and it is used in Spark Streaming.
2. A variant approach, used in Apache Flink, is to periodically generate rolling checkpoints of state and write them to durable storage. If a stream operator crashes, it can restart from its most recent checkpoint and discard any output generated between the last checkpoint and the crash.

To provide the same exactly-once semantics, also need to ensure that all outputs and side effects of processing an event take effect if and only if the processing is successful. Some implementations do not attempt to provide transactions across heterogeneous technologies, but instead keep the transactions internal by managing both state changes and messaging within the stream processing framework.

Our goal is to discard the partial output of any failed tasks so that they can be safely retried without taking effect twice. Distributed transactions are one way of achieving that goal, but another way is to rely on idempotence. An idempotent operation is one that you can perform multiple times, and it has the same effect as if you performed it only once. Relying on idempotence implies several assumptions: restarting a failed task must replay the same messages in the same order (a log-based message broker does this), the processing must be deterministic, and no other node may concurrently update the same value.

## C12. More

Combining Specialized Tools by Deriving Data
REASONING ABOUT DATAFLOWS
DERIVED DATA VERSUS DISTRIBUTED TRANSACTIONS
THE LIMITS OF TOTAL ORDERING
ORDERING EVENTS TO CAPTURE CAUSALITY

Batch and Stream Processing
MAINTAINING DERIVED STATE
REPROCESSING DATA FOR APPLICATION EVOLUTION
THE LAMBDA ARCHITECTURE
	The lambda architecture proposes running two different systems in parallel: a batch processing system such as Hadoop MapReduce, and a separate stream-processing system such as Storm. In the lambda approach, the stream processor consumes the events and quickly produces an approximate update to the view; the batch processor later consumes the same set of events and produces a corrected version of the derived view.
	Problems: Having to maintain the same logic to run both in a batch and in a stream processing framework is significant additional effort; the stream pipeline and the batch pipeline produce separate outputs and they need to be merged in order to respond to user requests; the batch pipeline often needs to be set up to process incremental batches, handling stragglers, handling windows that cross boundaries between batches, adding complexity, making it more akin to the streaming layer
UNIFYING BATCH AND STREAM PROCESSING

Composing Data Storage Technologies
CREATING AN INDEX
THE META-DATABASE OF EVERYTHING
MAKING UNBUNDLING WORK
UNBUNDLED VERSUS INTEGRATED SYSTEMS
	Federated databases: unifying reads. It is possible to provide a unified query interface to a wide variety of underlying storage engines and processing methods—an approach known as a federated database or polystore
	Unbundled databases: unifying writes. Making it easier to reliably plug together storage systems (e.g., through change data capture and event logs) is like unbundling a database’s index-maintenance features in a way that can synchronize writes across disparate technologies

Designing Applications Around Dataflow
APPLICATION CODE AS A DERIVATION FUNCTION
	custom code is required to handle the application-specific aspects
SEPARATION OF APPLICATION CODE AND STATE
DATAFLOW: INTERPLAY BETWEEN STATE CHANGES AND APPLICATION CODE
	maintaining derived data is not the same as asynchronous job execution, for which messaging systems are traditionally designed
STREAM PROCESSORS AND SERVICES
	The advantage of such a service-oriented architecture over a single monolithic application is primarily organizational scalability through loose coupling: different teams can work on different services, which reduces coordination effort between teams. Composing stream operators into dataflow systems: one-directional, asynchronous message streams rather than synchronous request/response interactions.

Observing Derived State
	write path: whenever some piece of information is written to the system, it may go through multiple stages of batch and stream processing, and eventually every derived dataset is updated to incorporate the data that was written.
	read path: when serving a user request you read from the derived dataset, perhaps perform some more processing on the results, and construct the response to the user.
	the write path is similar to eager evaluation, and the read path is similar to lazy evaluation
	The derived dataset is the place where the write path and the read path meet.
MATERIALIZED VIEWS AND CACHING
	Shifting the boundary between work done on the write path and the read path was in fact the topic of the Twitter example at the beginning of this book. The boundary between write path and read path might be drawn differently for celebrities compared to ordinary users.
STATEFUL, OFFLINE-CAPABLE CLIENTS
	When we move away from the assumption of stateless clients talking to a central database and toward state that is maintained on end-user devices, a world of new opportunities opens up. In particular, we can think of the on-device state as a cache of state on the server. The pixels on the screen are a materialized view onto model objects in the client app; the model objects are a local replica of state in a remote datacenter.
PUSHING STATE CHANGES TO CLIENTS
END-TO-END EVENT STREAMS
	The challenge is that the assumption of stateless clients and request/response interactions is very deeply ingrained in our databases, libraries, frameworks, and protocols. Many datastores support read and write operations where a request returns one response, but much fewer provide an ability to subscribe to changes—i.e., a request that returns a stream of responses over time.
	In order to extend the write path all the way to the end user, we would need to fundamentally rethink the way we build many of these systems: moving away from request/response interaction and toward publish/subscribe dataflow.
READS ARE EVENTS TOO
	When both the writes and the reads are represented as events, and routed to the same stream operator in order to be handled, we are in fact performing a stream-table join between the stream of read queries and the database. A one-off read request just passes the request through the join operator and then immediately forgets it; a subscribe request is a persistent join with past and future events on the other side of the join.
MULTI-PARTITION DATA PROCESSING
	treating queries as streams provides an option for implementing large-scale applications that run against the limits of conventional off-the-shelf solutions

The End-to-End Argument for Databases
EXACTLY-ONCE EXECUTION OF AN OPERATION
DUPLICATE SUPPRESSION
UNIQUELY IDENTIFYING REQUESTS
APPLYING END-TO-END THINKING IN DATA SYSTEMS
	low-level reliability features are not by themselves sufficient to ensure end-to-end correctness
	When we refuse to use distributed transactions because they are too expensive, we end up having to reimplement fault-tolerance mechanisms in application code.

Enforcing Constraints
UNIQUENESS CONSTRAINTS REQUIRE CONSENSUS
	 If you want to be able to immediately reject any writes that would violate the constraint, synchronous coordination is unavoidable
UNIQUENESS IN LOG-BASED MESSAGING
MULTI-PARTITION REQUEST PROCESSING

Timeliness and Integrity
CORRECTNESS OF DATAFLOW SYSTEMS
LOOSELY INTERPRETED CONSTRAINTS
	use compensating transaction. These applications do require integrity, but they don’t require timeliness on the enforcement of the constraint.
COORDINATION-AVOIDING DATA SYSTEMS

Trust, but Verify
MAINTAINING INTEGRITY IN THE FACE OF SOFTWARE BUGS
	When it comes to application code, we have to assume many more bugs. Many applications don’t even correctly use the features that databases offer for preserving integrity.
DON’T JUST BLINDLY TRUST WHAT THEY PROMISE
	Mature systems similarly tend to consider the possibility of unlikely things going wrong, and manage that risk.
A CULTURE OF VERIFICATION
	self-validating or self-auditing systems that continually check their own integrity
DESIGNING FOR AUDITABILITY
	event-based systems can provide better auditability. Being explicit about dataflow makes the provenance of data much clearer, which makes integrity checking much more feasible. A deterministic and well-defined dataflow also makes it easier to debug and trace the execution of a system in order to determine why it did something.
TOOLS FOR AUDITABLE DATA SYSTEMS
	Cryptographic auditing and integrity checking often relies on Merkle trees. Certificate transparency is a security technology that relies on Merkle trees to check the validity of TLS/SSL certificates.

## Quotes

The Internet was done so well that most people think of it as a natural resource like the Pacific Ocean, rather than something that was man-made. When was the last time a technology with a scale like that was so error-free?
> Alan Kay, in interview with Dr Dobb’s Journal (2012)

The limits of my language mean the limits of my world.
> Ludwig Wittgenstein, Tractatus Logico-Philosophicus (1922)

Wer Ordnung hält, ist nur zu faul zum Suchen. (If you keep things tidily ordered, you’re just too lazy to go searching.)
> German proverb

Everything changes and nothing stands still.
    Heraclitus of Ephesus, as quoted by Plato in Cratylus (360 BCE)

The major difference between a thing that might go wrong and a thing that cannot possibly go wrong is that when a thing that cannot possibly go wrong goes wrong it usually turns out to be impossible to get at or repair.
> Douglas Adams, Mostly Harmless (1992)

Clearly, we must break away from the sequential and not limit the computers. We must state definitions and provide for priorities and descriptions of data. We must state relationships, not procedures.
> Grace Murray Hopper, Management and the Computer of the Future (1962)

Is it better to be alive and wrong or right and dead?
> Jay Kreps, A Few Notes on Kafka and Jepsen (2013)

A system cannot be successful if it is too strongly influenced by a single person. Once the initial design is complete and fairly robust, the real test begins as people with many different viewpoints undertake their own experiments.
> Donald Knuth

If a thing be ordained to another as to its end, its last end cannot consist in the preservation of its being. Hence a captain does not intend as a last end, the preservation of the ship entrusted to him, since a ship is ordained to something else as its end, viz. to navigation.
(Often quoted as: If the highest aim of a captain was the preserve his ship, he would keep it in port forever.)
> St. Thomas Aquinas, Summa Theologica (1265–1274)