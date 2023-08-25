---
layout: post
title:  "Cloud Design - Data Management Patterns (1)"
categories: technology
tags: design patterns
excerpt: "These design patterns are useful for building reliable, scalable, secure applications in the cloud."
---

* content
{:toc}

## Data Management patterns

Data management is the key element of cloud applications, and influences most of the quality attributes. Data is typically hosted in different locations and across multiple servers for reasons such as performance, scalability or availability, and this can present a range of challenges. For example, data consistency must be maintained, and data will typically need to be synchronized across different locations.
Additionally data should be protected at rest, in transit, and via authorized access mechanisms to maintain security assurances of confidentiality, integrity, and availability.

    Data Protection:

    1. Discovery, classify and label sensitive data
    2. Protect sensitive data
    3. Monitor for unauthorized transfer of sensitive data
    4. Encrypt sensitive information in transit
    5. Encrypt sensitive data at rest

### Cache-Aside pattern

#### Problem

Applications should implement a strategy that helps to ensure that the data in the cache is as up-to-date as possible, but can also detect and handle situations that arise when the data in the cache has become stale.

#### Solution

An application can emulate the functionality of read-through caching by implementing the cache-aside strategy. This strategy loads data into the cache *on demand* (not in advance). This can improve performance and also helps to maintain consistency between data held in the cache and data in the underlying data store.

1. Determine whether the item is currently held in the cache.
2. If the item is not currently in the cache, read the item from the data store.
3. Store a copy of the item in the cache.
4. Write-through strategy: if an application updates the information, it make the modification to the data store and invalidate the corresponding item in the cache.

#### Considerations

* Lifetime of cached data
    Many caches implement an expiration policy that invalidates data and removes it from the cache if it's not accessed for a specified period. For cache-aside to be effective, ensure that the expiration policy matches the pattern of access for applications that use the data (not too short or too long).
* Evicting data
    Most caches adopt a least-recently-used policy for selecting items to evict, but this might be customizable. Configure the global expiration property and also the property of each cached item, to ensure that the cache is cost effective.
* Priming the cache
    Many solutions prepopulate the cache with the data that an application is likely to need as part of the startup processing. The pattern can be useful if some of this data expires or is evicted.
* Consistency
    External process may change the data store at any time. In a system that replicates data across data stores, this problem can become serious if synchronization occurs frequently.
* Local (in-memory) caching
    A local cache is private and so different application instances could each have a copy of the same cached data. This data could quickly become inconsistent between caches, so it might be necessary to expire data held in a private cache and refresh it more frequently. In these scenarios, consider investigating the use of a shared or a distributed caching mechanism.

#### Usage

Use this pattern when:

* A cache doesn't provide native read-through and write-through operations.
* Resource demand is unpredictable. This pattern enables applications to load data on demand. It makes no assumptions about which data an application will require in advance.

This pattern might not be suitable:

* When the cached data set is static. If the data will fit into the available cache space, prime the cache with the data on startup and apply a policy that prevents the data from expiring.
* For caching session state information in a web application hosted in a web farm. In this environment, you should avoid introducing dependencies based on client-server affinity.

#### Caching

https://docs.microsoft.com/en-us/azure/architecture/best-practices/caching

Considerations:

1. when to cache data
2. how to cache data effectively
3. cache highly dynamic data
4. manage data expiration in a cache
5. invalidate data in a client-side cache

### Command and Query Responsibility Segregation (CQRS) pattern

#### Problem

Read and write workloads are often asymmetrical, with very different performance and scale requirements.

* There is often a mismatch between the read and write representations of the data, such as additional columns or properties that must be updated correctly even though they aren't required as part of an operation.
* Data contention can occur when operations are performed in parallel on the same set of data.
* The traditional approach can have a negative effect on performance due to load on the data store and data access layer, and the complexity of queries required to retrieve information.
* Managing security and permissions can become complex, because each entity is subject to both read and write operations, which might expose data in the wrong context.

#### Solution

CQRS separates reads and writes into different models, using commands to update data, and queries to read data.

* Commands should be task based, rather than data centric. ("Book hotel room", not "set ReservationStatus to Reserved").
* Commands may be placed on a queue for asynchronous processing, rather than being processed synchronously.
* Queries never modify the database. A query returns a DTO that does not encapsulate any domain knowledge.

For greater isolation, you can physically separate the read data from the write data. In that case, the read database can use its own data schema that is optimized for queries, e.g. a materialized view or a different type of data store, or a read-only replica.

Some implementations of CQRS use the Event Sourcing pattern. In a CQRS context, one benefit of Event Sourcing is that the same events can be used to notify other components — in particular, to notify the read model. The read model uses the events to create a snapshot of the current state, which is more efficient for queries. However, Event Sourcing adds complexity to the design.

Benefits:

* Independent scaling.
    CQRS allows the read and write workloads to scale independently, and may result in fewer lock contentions.
* Optimized data schemas.
    The read side can use a schema that is optimized for queries, while the write side uses a schema that is optimized for updates.
* Security.
    It's easier to ensure that only the right domain entities are performing writes on the data.
* Separation of concerns.
    Segregating the read and write sides can result in models that are more maintainable and flexible. Most of the complex business logic goes into the write model. The read model can be relatively simple.
* Simpler queries.
    By storing a materialized view in the read database, the application can avoid complex joins when querying.

#### Considerations

* Complexity
    The basic idea of CQRS is simple. But it can lead to a more complex application design, especially if they include the Event Sourcing pattern.
* Messaging
    Although CQRS does not require messaging, it's common to use messaging to process commands and publish update events. In that case, the application must handle message failures or duplicate messages. See the guidance on Priority Queues for dealing with commands having different priorities.
* Eventual consistency.
    If you separate the read and write databases, the read data may be stale. The read model store must be updated to reflect changes to the write model store, and it can be difficult to detect when a user has issued a request based on stale read data.

#### Usage

Use this pattern for scenarios:

* Collaborative domains where many users access the same data in parallel. CQRS allows you to define commands with enough granularity to minimize merge conflicts at the domain level, and conflicts that do arise can be merged by the command.
* Task-based user interfaces where users are guided through a complex process as a series of steps or with complex domain models. The write model has a full command-processing stack with business logic, input validation, and business validation. The read model has no business logic or validation stack, and just returns a DTO for use in a view model. The read model is eventually consistent with the write model.
* Scenarios where performance of data reads must be fine tuned separately from performance of data writes, especially when the number of reads is much greater than the number of writes.
* Multiple teams
* An evolving system that might contain multiple versions of the model
* Integration with other systems, especially in combination with event sourcing, where the temporal failure of one subsystem shouldn't affect the availability of the others.

This pattern isn't recommended when:

* The domain or the business rules are simple.
* A simple CRUD-style user interface and data access operations are sufficient.

### Event Sourcing pattern

#### Problem

A typical approach of data-related application is to maintain the current state of the data by updating it as users work with it, for example, the traditional CRUD model. But it has limitations: slow down performance and responsiveness, and limit scalability, due to the processing overhead; data update conflicts are more likely because the update operations take place on a single item of data; history is lost unless there is an additional auditing mechanism with detailed logs.

#### Solution

The Event Sourcing pattern defines an approach to handling operations on data that's driven by a sequence of events, each of which is recorded in an append-only store. Application code sends a series of events that imperatively describe each action that has occurred on the data to the event store, where they're persisted. Each event represents a set of changes to the data.
Typical uses of the events published by the event store are to maintain materialized views of entities as actions in the application change them, and for integration with external systems.
In addition, at any point it's possible for applications to read the history of events, and use it to materialize the current state of an entity by playing back and consuming all the events related to that entity, for the case of on demand to materialize a domain object.

Advantages：

* Events are immutable and can be stored using an append-only operation. The user interface, workflow, or process that initiated an event can continue, and tasks that handle the events can run in the background. This, combined with the fact that there's no contention during the processing of transactions, can vastly improve performance and scalability for applications especially for the presentation level or user interface.
* Events are simple objects that describe some action that occurred, together with any associated data required to describe the action represented by the event. Events don't directly update a data store. They're simply recorded for handling at the appropriate time. This can simplify implementation and management.
* Events typically have meaning for a domain expert.
* Event sourcing can help prevent concurrent updates from causing conflicts because it avoids the requirement to directly update objects in the data store. However, the domain model must still be designed to protect itself from requests that might result in an inconsistent state.
* The append-only storage of events provides an audit trail that can be used to monitor actions taken against a data store, regenerate the current state as materialized views or projections by replaying the events at any time, and assist in testing and debugging the system. In addition, the requirement to use compensating events to cancel changes provides a history of changes that were reversed, which wouldn't be the case if the model simply stored the current state. The list of events can also be used to analyze application performance and detect user behavior trends, or to obtain other useful business information.
* The event store raises events, and tasks perform operations in response to those events. This decoupling of the tasks from the events provides flexibility and extensibility. Tasks know about the type of event and the event data, but not about the operation that triggered the event. In addition, multiple tasks can handle each event. This enables easy integration with other services and systems that only listen for new events raised by the event store. However, the event sourcing events tend to be very low level, and it might be necessary to generate specific integration events instead.

#### Considerations

* The system will only be eventually consistent when creating materialized views or generating projections of data by replaying events.
* The event store is the permanent source of information, and so the event data should never be updated.
    The only way to update an entity to undo a change is to add a compensating event to the event store.
    If the format (rather than the data) of the persisted events needs to change, perhaps during a migration, it can be difficult to combine existing events in the store with the new version. -> Consider using a version stamp on each version of the event schema to maintain both the old and the new event formats.
* Multi-threaded applications and multiple instances of applications might be storing events in the event store. The consistency of events in the event store is vital, as is the order of events that affect a specific entity (the order that changes occur to an entity affects its current state). -> Adding a timestamp to every event can help to avoid issues. Another common practice is to annotate each event resulting from a request with an incremental identifier.
* There's no standard approach, or existing mechanisms such as SQL queries, for reading the events to obtain information.
* The length of each event stream affects managing and updating the system. If the streams are large, consider creating snapshots at specific intervals such as a specified number of events.
* Even though event sourcing minimizes the chance of conflicting updates to the data, the application must still be able to deal with inconsistencies that result from eventual consistency and the lack of transactions.
* Event publication might be "at least once," and so consumers of the events must be idempotent. They must not reapply the update described in an event if the event is handled more than once.

#### Usage

Use this pattern for scenarios:

* When you want to capture intent, purpose, or reason in the data.
* When it's vital to minimize or completely avoid the occurrence of conflicting updates to data.
* When you want to record events that occur, and be able to replay them to restore the state of a system, roll back changes, or keep a history and audit log.
* When using events is a natural feature of the operation of the application, and requires little additional development or implementation effort.
* When you need to decouple the process of inputting or updating data from the tasks required to apply these actions. This might be to improve UI performance, or to distribute events to other listeners that take action when the events occur.
* When you want flexibility to be able to change the format of materialized models and entity data if requirements change, or—when used in conjunction with CQRS—you need to adapt a read model or the views that expose the data.
* When used in conjunction with CQRS, and eventual consistency is acceptable while a read model is updated, or the performance impact of rehydrating entities and data from an event stream is acceptable.

This pattern might not be useful in the following situations:

* Small or simple domains, systems that have little or no business logic, or nondomain systems that naturally work well with traditional CRUD data management mechanisms.
* Systems where consistency and real-time updates to the views of the data are required.
* Systems where audit trails, history, and capabilities to roll back and replay actions are not required.
* Systems where there's only a very low occurrence of conflicting updates to the underlying data. For example, systems that predominantly add data rather than updating it.

### Index Table Pattern

#### Problem

Although secondary indexes are common in relational systems, most NoSQL data stores used by cloud applications don't provide an equivalent feature.

#### Solution

If the data store doesn't support secondary indexes, you can emulate them manually by creating your own index tables. An index table organizes the data by a specified key. Three strategies are commonly used for structuring an index table, depending on the number of secondary indexes that are required and the nature of the queries that an application performs.

1. The first strategy is to duplicate the data in each index table but organize it by different keys (complete denormalization).
    This strategy is appropriate if the data is relatively static compared to the number of times it's queried using each key. If the data is more dynamic, the processing overhead of maintaining each index table becomes too large for this approach to be useful. Also, if the volume of data is very large, the amount of space required to store the duplicate data is significant.

2. The second strategy is to create normalized index tables organized by different keys and reference the original data by using the primary key rather than duplicating it. The original data is called a fact table.
    This technique saves space and reduces the overhead of maintaining duplicate data. The disadvantage is that an application has to perform two lookup operations to find data using a secondary key.

3. The third strategy is to create partially normalized index tables organized by different keys that duplicate frequently retrieved fields. Reference the fact table to access less frequently accessed fields. With this strategy, you can strike a balance between the first two approaches.

Index tables can speed up query operations over sharded data, and are especially useful where the shard key is hashed. This can save the application from repeatedly calculating hash keys (an expensive operation) if it needs to retrieve data that falls within a range, or it needs to fetch data in order of the nonhashed key.

#### Considerations

* The overhead of maintaining secondary indexes can be significant. You must analyze and understand the queries that your application uses. Only create index tables when they're likely to be used regularly.
* Duplicating data in an index table can add significant overhead in storage costs and the effort required to maintain multiple copies of data.
* Implementing an index table as a normalized structure that references the original data requires an application to perform two lookup operations to find data.
* If a system incorporates a number of index tables over very large data sets, it can be difficult to maintain consistency between index tables and the original data. It might be possible to design the application around the eventual consistency model. For example, to insert, update, or delete data, an application could post a message to a queue and let a separate task perform the operation and maintain the index tables that reference this data asynchronously.
* Index tables might themselves be partitioned or sharded.

#### Usage

Use this pattern to improve query performance when an application frequently needs to retrieve data by using a key other than the primary (or shard) key.

This pattern might not be useful when:
* Data is volatile. An index table can become out of date very quickly, making it ineffective or making the overhead of maintaining the index table greater than any savings made by using it.
* A field selected as the secondary key for an index table is nondiscriminating and can only have a small set of values (for example, gender). A similar case, the balance of the data values for a field selected as the secondary key for an index table are highly skewed. For example, if 90% of the records contain the same value in a field.

### Materialized View Pattern

#### Problem

When storing data, the priority for developers and data administrators is often focused on how the data is stored, as opposed to how it's read.  The chosen storage format is usually closely related to the format of the data, requirements for managing data size and data integrity, and the kind of store in use. When a query only needs a subset of the data from some entities, such as a summary of orders for several customers without all of the order details, it must extract all of the data for the relevant entities in order to obtain the required information.

#### Solution

To support efficient querying, a common solution is to generate, in advance, a view that materializes the data in a format suited to the required results set. The Materialized View pattern describes generating prepopulated views of data in environments where the source data isn't in a suitable format for querying, where generating a suitable query is difficult, or where query performance is poor due to the nature of the data or the data store.

Materialized views tend to be specifically tailored to one, or a small number of queries. A materialized view can even be optimized for just a single query.
A key point is that a materialized view and the data it contains is completely disposable because it can be entirely rebuilt from the source data stores. A materialized view is never updated directly by an application, and so it's a specialized cache.

#### Considerations

* How and when the view will be updated. Ideally it'll regenerate in response to an event indicating a change to the source data, although this can lead to excessive overhead if the source data changes rapidly. Alternatively, consider using a scheduled task, an external trigger, or a manual action to regenerate the view.
* In some systems, such as when using the Event Sourcing pattern to maintain a store of only the events that modified the data, materialized views are necessary. Prepopulating views by examining all events to determine the current state might be the only way to obtain information from the event store.
* For other cases, If many queries are used, materialized views can result in unacceptable storage capacity requirements and storage cost.
* Consider the impact on data consistency when generating the view, and when updating the view if this occurs on a schedule. If the source data is changing at the point when the view is generated, the copy of the data in the view won't be fully consistent with the original data.
* Consider where you'll store the view. The view doesn't have to be located in the same store or partition as the original data. It can be a subset from a few different partitions combined.
* It can be stored in a cache or in a less reliable location.
* Where the storage mechanism supports it, consider indexing the materialized view to further increase performance.

#### Usage

This pattern is useful when:

* Creating materialized views over data that's difficult to query directly, or where queries must be very complex to extract data that's stored in a normalized, semi-structured, or unstructured way.
* Creating temporary views that can dramatically improve query performance, or can act directly as source views or data transfer objects for the UI, for reporting, or for display.
* Supporting occasionally connected or disconnected scenarios where connection to the data store isn't always available. The view can be cached locally in this case.
* Simplifying queries and exposing data for experimentation in a way that doesn't require knowledge of the source data format. For example, by joining different tables in one or more databases, or one or more domains in NoSQL stores, and then formatting the data to fit its eventual use.
* Providing access to specific subsets of the source data that, for security or privacy reasons, shouldn't be generally accessible, open to modification, or fully exposed to users.
* Bridging different data stores, to take advantage of their individual capabilities. For example, using a cloud store that's efficient for writing as the reference data store, and a relational database that offers good query and read performance to hold the materialized views.

This pattern isn't useful in the following situations:

* The source data is simple and easy to query.
* The source data changes very quickly, or can be accessed without using a view. In these cases, you should avoid the processing overhead of creating views.
* Consistency is a high priority. The views might not always be fully consistent with the original data.
* When using microservices. Microservices typically have well defined boundries aligning to domain driven design (DDD).

### Sharding Pattern

#### Problem

A data store hosted by a single server might be subject to the following limitations:

* Storage space. A server typically provides only a finite amount of disk storage
* Computing resource. A single server hosting the data store might not be able to provide the necessary computing power to support this load, resulting in extended response times for users and frequent failures as applications attempting to store and retrieve data time out.
* Network bandwidth. The performance of a data store running on a single server is governed by the rate the server can receive requests and send replies.
* Geography. If the users are dispersed across different countries or regions, it might not be possible to store the entire data for the application in a single data store.

#### Solution

Divide the data store into horizontal partitions or shards. Each shard has the same schema, but holds its own distinct subset of the data. A shard is a data store in its own right (it can contain the data for many entities of different types), running on a server acting as a storage node.

This pattern has the following benefits:

* You can scale the system out by adding further shards running on additional storage nodes.
* A system can use off-the-shelf hardware rather than specialized and expensive computers for each storage node.
* You can reduce contention and improve performance by balancing the workload across shards.
* In the cloud, shards can be located physically close to the users that'll access the data.

When dividing a data store up into shards, decide which data should be placed in each shard based one or more attributes of the data (shard key, or partition key).

Sharding physically organizes the data. When an application stores and retrieves data, the sharding logic (can be either in the application or the data storage system) directs the application to the appropriate shard.

Abstracting the physical location of the data in the sharding logic provides a high level of control over which shards contain which data. It also enables data to migrate between shards without reworking the business logic of an application if the data in the shards need to be redistributed later (for example, if the shards become unbalanced). The tradeoff is the additional data access overhead required in determining the location of each data item as it's retrieved.

To ensure optimal performance and scalability, it's important to split the data in a way that's appropriate for the types of queries that the application performs.

If queries regularly retrieve data using a combination of attribute values, you can likely define a composite shard key by linking attributes together. Alternatively, use a pattern such as Index Table to provide fast lookup to data based on attributes that aren't covered by the shard key.

https://zhuanlan.zhihu.com/p/57185574

##### Strategies

1. The Lookup strategy.
    In this strategy the sharding logic implements a map that routes a request for data to the shard that contains that data using the shard key. In a multi-tenant application all the data for a tenant might be stored together in a shard using the tenant ID as the shard key. Multiple tenants might share the same shard, but the data for a single tenant won't be spread across multiple shards.
    The mapping between the shard key and the physical storage can be based on physical shards where each shard key maps to a physical partition.
    A more flexible technique for rebalancing shards is virtual partitioning, where shard keys map to the same number of virtual shards, which in turn map to fewer physical partitions. Looking up shard locations can impose an additional overhead.
    This offers more control over the way that shards are configured and used.
    Scaling can be carried out at the user level, either online or offline.

2. The Range strategy.
    The Range strategy. This strategy groups related items together in the same shard, and orders them by shard key—the shard keys are sequential. It's useful for applications that frequently retrieve sets of items using range queries.
    This is easy to implement and works well with range queries because they can often fetch multiple data items from a single shard in a single operation. This strategy offers easier data management.
    This strategy doesn't provide optimal balancing between shards. Rebalancing shards is difficult and might not resolve the problem of uneven load if the majority of activity is for adjacent shard keys.
    Scaling must typically be carried out when a part or all of the data store is offline because the data must be split and merged across the shards.

3. The Hash strategy.
    The purpose of this strategy is to reduce the chance of hotspots. The sharding logic computes the shard to store an item in based on a hash of one or more attributes of the data. The chosen hashing function should distribute data evenly across the shards, possibly by introducing some random element into the computation.
    This strategy offers a better chance of more even data and load distribution. Request routing can be accomplished directly by using the hash function. There's no need to maintain a map. Note that computing the hash might impose an additional overhead. Also, rebalancing shards is difficult.
    Scaling is complex but does not require state maintenance.

4. Shard data based on workload, segregate the data for highly volatile tenants in separate shards. The speed of data access for other tenants might be improved as a result.

5. Shard data based on the location of tenants. You can take the data for tenants in a specific geographic region offline for backup and maintenance during off-peak hours in that region, while the data for tenants in other regions remains online and accessible during their business hours.

6. Shard data based on the value of tenants, high-value tenants could be assigned their own private, high performing, lightly loaded shards, whereas lower-value tenants might be expected to share more densely-packed, busy shards.

7. Shard data based on the requirement of data isolation and privacy.

#### Considerations

* Sharding is complementary to other forms of partitioning, such as vertical partitioning and functional partitioning.
    https://docs.microsoft.com/en-us/previous-versions/msp-n-p/dn589795(v=pandp.10)
* Keep shards balanced so they all handle a similar volume of I/O.
   It's necessary to periodically rebalance the shards guarantee an even distribution and to reduce the chance of hotspots. Also, plan the storage well for growth and develop strategies and scripts to quickly rebalance shards.
* Use stable and unique data for the shard key.
* Shard the data to support the most frequently performed queries, and if necessary create secondary index tables to support queries that retrieve data using criteria based on attributes that aren't part of the shard key.
* A single shard can contain the data for multiple types of entities. Consider denormalizing your data to keep related entities that are commonly queried together in the same shard to reduce the number of separate reads that an application performs. (You can also keep the shard key of the second entity as part of the schema).
* Fetch multiple shards using parallel tasks, like fan-out queries.
* Consider replicating reference data to all shards for static or slow-moving data.
* You should minimize operations that affect data in multiple shards for better referential integrity and consistency between shards.
* Tasks such as monitoring, backing up, checking for consistency, and logging or auditing must be accomplished on multiple shards and servers, possibly held in multiple locations, for configuring and mantenance.

#### Usage

Use this pattern when a data store is likely to need to scale beyond the resources available to a single storage node, or to improve performance by reducing contention in a data store.

### Static Content Hosting Pattern

#### Problem

Although web servers are optimized for dynamic rendering and output caching, they still have to handle requests to download static content. This consumes processing cycles that could often be put to better use.

#### Solution

In most cloud hosting environments, you can put some of an application's resources and static pages in a storage service. The storage service can serve requests for these resources, reducing load on the compute resources that handle other web requests. The cost for cloud-hosted storage is typically much less than for compute instances.

#### Considerations

* The hosted storage service must expose an HTTP/HTTPS endpoint that users can access to download the static resources.
* For maximum performance and availability, consider using (and paying for) a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.
* Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.
* Perform separate deployments, and version the application and content to manage content updates in different places more easily, especially when the static content includes script files or UI components.
* Storage services might not support the use of custom domain names.
* The storage containers must be configured for public read access, but never public write access.
* Consider using a valet key or token to control access to resources that shouldn't be available anonymously.

#### Usage

This pattern is useful for:

* Minimizing the hosting cost for websites and applications that contain some static resources, or consist of only static content and resources. It might be possible to entirely host a fully static website in a storage account, depending on the capabilities of the hosting provider's storage system.
* Exposing static resources and content for applications running in other hosting environments or on-premises servers.
* Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.
* Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.

This pattern might not be useful in the following situations:

* The application needs to perform some processing on the static content before delivering it to the client. For example, it might be necessary to add a timestamp to a document.
* The volume of static content is very small, comparing to the overhead of retrieving this content from separate storage other than the compute resource.

### Valet Key Pattern

#### Problem

Data stores have the ability to handle upload and download of data directly, without requiring that the application perform any processing to move this data, to minimize data transfer costs and the requirement to scale out the application, and to maximize performance. But, this typically requires the client to have access to the security credentials for the store. After the client has a connection to the data store for direct access, the application can't act as the gatekeeper. It's no longer in control of the process and can't prevent subsequent uploads or downloads from the data store.

#### Solution

One typical solution is to restrict access to the data store’s public connection and provide the client with a key or token that the data store can validate.
This key or token is usually referred to as a valet key.

It provides time-limited access to specific resources and allows only predefined operations. Applications can create and issue valet keys to client devices and web browsers quickly and easily.
The key can also be invalidated by the application.

Using this pattern can simplify managing access to resources. It also makes it easy to limit the location, the permission, and the validity period—all by simply generating a key at runtime.

#### Considerations

* Manage the validity status and period of the key.  Specify a short validity period to minimize the risk of allowing unauthorized operations to take place against the data store, but not too short for client to complete the operation. Allow authorized users to renew the key before the validity period expires if multiple accesses to the protected resource are required.
  * There might be out of synchronization between server and client clock
* Control the level of access the key will provide. It's critical to accurately specify the resource or the set of resources to which the key applies.
* Consider how to control users’ behavior. For example, how to limit the number of write times, the size of the data, or the quotas for individual users in a multi-tenant scenario where the same key is used by all the users from one tenant.
  * Force the user to notify the server of completion of the operation
  * Policies should not limit the operations
* Validate, and optionally sanitize, all uploaded data to avoid malicious user data, invalid authorized user data or errors.
* Audit all operations. Many key-based mechanisms can log operations such as uploads, downloads, and failures. These logs can usually be incorporated into an audit process, and also used for billing if the user is charged based on file size or data volume. Use the logs to detect authentication failures that might be caused by issues with the key provider, or accidental removal of a stored access policy.
* Deliver the key securely. It can be embedded in a URL that the user activates in a web page, or it can be used in a server redirection operation so that the download occurs automatically. Always use HTTPS to deliver the key over a secure channel.
  * The URL containing the key will be recorded in server log files and it should be kept secured (expire the key before transmitting the log data)
* Protect sensitive data in transit. Sensitive data delivered through the application will usually take place using SSL or TLS, and this should be enforced for clients accessing the data store directly.
* If the client code runs in a web browser, the browser might need to support cross-origin resource sharing (CORS) to enable code that executes within the web browser to access data in a different domain from the one that served the page.

#### Usage

Similar to the previous one.
This pattern is useful for the following situations:

* To minimize resource loading and maximize performance and scalability. Using a valet key doesn't require the resource to be locked, no remote server call is required, there's no limit on the number of valet keys that can be issued, and it avoids a single point of failure resulting from performing the data transfer through the application code. Creating a valet key is typically a simple cryptographic operation of signing a string with a key.
* To minimize operational cost. Enabling direct access to stores and queues is resource and cost efficient, can result in fewer network round trips, and might allow for a reduction in the number of compute resources required.
* When clients regularly upload or download data, particularly where there's a large volume or when each operation involves large files.
* When the application has limited compute resources available, either due to hosting limitations or cost considerations. In this scenario, the pattern is even more helpful if there are many concurrent data uploads or downloads because it relieves the application from handling the data transfer.
* When the data is stored in a remote data store or a different datacenter. If the application was required to act as a gatekeeper, there might be a charge for the additional bandwidth of transferring the data between datacenters, or across public or private networks between the client and the application, and then between the application and the data store.

This pattern might not be useful in the following situations:

* If the application must perform some task on the data before it's stored or before it's sent to the client.
* If the design of an existing application makes it difficult to incorporate the pattern. Using this pattern typically requires a different architectural approach for delivering and receiving data.
* If it's necessary to maintain audit trails or control the number of times a data transfer operation is executed, and the valet key mechanism in use doesn't support notifications that the server can use to manage these operations.
* If it's necessary to limit the size of the data, especially during upload operations. The only solution to this is for the application to check the data size after the operation is complete, or check the size of uploads after a specified period or on a scheduled basis.
