---
layout: post
title:  "Cloud Design - Data Management patterns"
categories: Software
tags: design
excerpt: "These design patterns are useful for building reliable	 scalable	 secure applications in the cloud."
---

* content
{:toc}

## Design and Implementation Patterns

### Ambassador Pattern

#### Problem

It may be difficult or impossible to update legacy applications or existing code libraries to add network-related features	 and also for cross-application network call configurations.

#### Solution

Put client frameworks and libraries into an external process that acts as a proxy between your application and external services. Deploy the proxy on the same host environment as your application to allow control over routing	 resiliency	 security features	 and to avoid any host-related access restrictions.
You can also use the ambassador pattern to standardize and extend instrumentation. The proxy can monitor performance metrics such as latency or resource usage	 and this monitoring happens in the same host environment as the application.
Features that are offloaded to the ambassador can be managed independently of the application.

Ambassador services can be deployed as a sidecar to accompany the lifecycle of a consuming application or service. Alternatively	 if an ambassador is shared by multiple separate processes on a common host	 it can be deployed as a daemon. If the consuming service is containerized	 the ambassador should be created as a separate container on the same host	 with the appropriate links configured for communication.

#### Considerations

* The proxy adds some latency overhead.
* Consider the possible impact of including generalized features in the proxy	 for example	 retries handling.
* Consider a mechanism to allow the client to pass some context (enough information) to the proxy	 as well as back to the client.
* Consider how you will package and deploy the proxy.
* Consider whether to use a single shared instance (shared proxy) for all clients or an instance for each client.

#### Usage

Use this pattern when you:

* Need to build a common set of client connectivity features for multiple languages or frameworks.
* Need to offload cross-cutting client connectivity concerns to infrastructure developers or other more specialized teams.
* Need to support cloud or cluster connectivity requirements in a legacy application or an application that is difficult to modify.

This pattern may not be suitable:

* When network request latency is critical. A proxy will introduce some overhead	 although minimal	 and in some cases this may affect the application.
* When client connectivity features are consumed by a single language. In that case	 a better option might be a client library that is distributed to the development teams as a package.
* When connectivity features cannot be generalized and require deeper integration with the client application.

### Sidecar Pattern

#### Problem

Applications and services often require related functionality	 such as monitoring	 logging	 configuration	 and networking services. These peripheral tasks can be implemented as separate components or services.
If they are tightly integrated into the application	 they can run in the same process as the application	 making efficient use of shared resources. But the component and the application have close interdependence on each other.
If the application is decomposed into services	 each component has its own dependencies and requires language-specific libraries to access the underlying platform and any resources shared with the parent application. In addition	 deploying these features as separate services can add latency to the application. Managing the code and dependencies for these language-specific interfaces can also add considerable complexity.

#### Solution

Co-locate a cohesive set of tasks with the primary application	 but place them inside their own process or container	 providing a homogeneous interface for platform services across languages.
A sidecar service is not necessarily part of the application	 but is connected to it. It goes wherever the parent application goes. Sidecars are supporting processes or services that are deployed with the primary application. A sidecar service shares the same lifecycle of its parent application. For each instance of the application	 an instance of the sidecar is deployed and hosted alongside it.

Advantages of using a sidecar pattern include:

* A sidecar is independent from its primary application in terms of runtime environment and programming language	 so you don't need to develop one sidecar per language.
* The sidecar can access the same resources as the primary application. For example	 a sidecar can monitor system resources used by both the sidecar and the primary application.
* Because of its proximity to the primary application	 there’s no significant latency when communicating between them.
* Even for applications that don’t provide an extensibility mechanism	 you can use a sidecar to extend functionality by attaching it as its own process in the same host or sub-container as the primary application.
* The sidecar pattern is often used with containers and referred to as a sidecar container or sidekick container.

#### Considerations

* Consider the deployment and packaging format you will use to deploy services	 processes	 or containers. Containers are particularly well suited to the sidecar pattern.
* When designing a sidecar service	 carefully decide on the interprocess communication mechanism. Try to use language- or framework-agnostic technologies unless performance requirements make that impractical.
* Before putting functionality into a sidecar	 consider whether it would work better as a separate service or a more traditional daemon. Also consider whether the functionality could be implemented as a library or using a traditional extension mechanism. Language-specific libraries may have a deeper level of integration and less network overhead.

#### Usage

Use this pattern when:

* Your primary application uses a heterogeneous set of languages and frameworks. A component located in a sidecar service can be consumed by applications written in different languages using different frameworks.
* A component is owned by a remote team or a different organization.
* A component or feature must be co-located on the same host as the application
* You need a service that shares the overall lifecycle of your main application	 but can be independently updated.
* You need fine-grained control over resource limits for a particular resource or component. For example	 you may want to restrict the amount of memory a specific component uses. You can deploy the component as a sidecar to manage memory usage independently of the main application.

This pattern may not be suitable:

* When interprocess communication needs to be optimized. Communication between a parent application and sidecar services includes some overhead	 notably latency in the calls.
* This may not be an acceptable trade-off for chatty interfaces.
* For small applications where the resource cost of deploying a sidecar service for each instance is not worth the advantage of isolation.
* When the service needs to scale differently than or independently from the main applications. If so	 it may be better to deploy the feature as a separate service.

### Anti-Corruption Layer Pattern

#### Problem

Most applications rely on other systems (legacy or external) for some data or functionality. This is especially true of gradual migrations, where new features must be able to call the legacy system. To interoperate with the legacy system, the new application may need to support outdated infrastructure, protocols, data models, APIs, or other features that you wouldn't otherwise put into a modern application.
When these legacy features have quality issues, supporting them "corrupts" what might otherwise be a cleanly designed modern application.

#### Solution

Isolate the different subsystems by placing an anti-corruption layer between them. This layer translates communications between the two systems, allowing one system to remain unchanged while the other can avoid compromising its design and technological approach.

The anti-corruption layer contains all of the logic necessary to translate between the two systems. The layer can be implemented as a component within the application or as an independent service.

#### Considerations

* The anti-corruption layer may add latency to calls made between the two systems.
* The anti-corruption layer adds an additional service that must be managed and maintained.
* Consider how your anti-corruption layer will scale.
* Consider whether you need more than one anti-corruption layer.
* Consider how the anti-corruption layer will be managed in relation with your other applications or services.
* Make sure transaction and data consistency are maintained and can be monitored.
* Consider whether the anti-corruption layer needs to handle all communication between different subsystems, or just a subset of features.
* If the anti-corruption layer is part of an application migration strategy, consider whether it will be permanent, or will be retired after all legacy functionality has been migrated.

#### Usage

Use this pattern when:

* A migration is planned to happen over multiple stages, but integration between new and legacy systems needs to be maintained.
* Two or more subsystems have different semantics, but still need to communicate.

This pattern may not be suitable if there are no significant semantic differences between new and legacy systems.

### Strangler Fig Pattern

#### Problem

As systems age, the development tools, hosting technology, and even system architectures they were built on can become increasingly obsolete. As new features and functionality are added, the complexity of these applications can increase dramatically, making them harder to maintain or add new features to.

Completely replacing a complex system can be a huge undertaking. During gradual migration, every time a feature or service is migrated, clients need to be updated to point to the new location.

#### Solution

Incrementally replace specific pieces of functionality with new applications and services. Create a façade that intercepts requests going to the backend legacy system. The façade routes these requests either to the legacy application or the new services. Existing features can be migrated to the new system gradually, and consumers can continue using the same interface, unaware that any migration has taken place.

This pattern helps to minimize risk from the migration, and spread the development effort over time. With the façade safely routing users to the correct application, you can add functionality to the new system at whatever pace you like, while ensuring the legacy application continues to function. Over time, as features are migrated to the new system, the legacy system is eventually "strangled" and is no longer necessary. Once this process is complete, the legacy system can safely be retired.

#### Considerations

* Consider how to handle services and data stores that are potentially used by both new and legacy systems. Make sure both can access these resources side-by-side.
* Structure new applications and services in a way that they can easily be intercepted and replaced in **future** strangler fig migrations.
* At some point, when the migration is complete, the strangler fig façade will **either go away or evolve into an adaptor** for legacy clients.
* Make sure the façade keeps up with the migration.
* Make sure the façade doesn't become a single point of failure or a performance bottleneck.

#### Usage

Use this pattern when gradually migrating a back-end application to a new architecture.

This pattern may not be suitable:

* When requests to the back-end system cannot be intercepted.
* For smaller systems where the complexity of wholesale replacement is low.

### Backends for Frontends Pattern

#### Problem

An application may initially be targeted at a desktop web UI. Typically, a backend service is developed in parallel that provides the features needed for that UI. The backend service becomes a general-purpose backend, serving the requirements of both the desktop and mobile interfaces. But the requirements for a mobile application backend differ from the desktop web UI, resulting in competing requirements for the backend.
As different teams working on different frontend interfaces and backend service, a disconnect between the interface and backend development teams will place a burden on the backend team to balance the competing requirements of the different UI teams. When one interface team requires changes to the backend, those changes must be validated with other interface teams before they can be integrated into the backend.

#### Solution

Create one backend per user interface. Fine-tune the behavior and performance of each backend to best match the needs of the frontend environment, without worrying about affecting other frontend experiences.
Because each backend is specific to one interface, it can be optimized for that interface. As a result, it will be smaller, less complex, and likely faster than a generic backend that tries to satisfy the requirements for all interfaces. Each interface team has autonomy to control their own backend.

#### Considerations

* Consider how many backends to deploy.
* If different interfaces (such as mobile clients) will make the same requests, consider whether it is necessary to implement a backend for each interface, or if a single backend will suffice.
* Code duplication across services is highly likely when implementing this pattern.
* Frontend-focused backend services should only contain client-specific logic and behavior. General business logic and other global features should be managed elsewhere in your application.
* Think about how this pattern might be reflected in the responsibilities of a development team.
* Consider how long it will take to implement this pattern. Will the effort of building the new backends incur technical debt, while you continue to support the existing generic backend?

#### Usage

Use this pattern when:

* A shared or general purpose backend service must be maintained with significant development overhead.
* You want to optimize the backend for the requirements of specific client interfaces.
* Customizations are made to a general-purpose backend to accommodate multiple interfaces.
* An alternative language is better suited for the backend of a different user interface.

This pattern may not be suitable:

* When interfaces make the same or similar requests to the backend.
* When only one interface is used to interact with the backend.

### Compute Resource Consolidation Pattern

#### Problem

In some solutions it makes sense to follow the design principle of separation of concerns initially, and divide these operations into separate computational units that are hosted and deployed individually. Although this strategy can help simplify the logical design of the solution, deploying a large number of computational units as part of the same application can increase runtime hosting costs and make management of the system more complex. Each computational unit consumes chargeable resources, even when it's idle or lightly used. Therefore, this isn't always the most cost-effective solution.

#### Solution

To help reduce costs, increase utilization, improve communication speed, and reduce management it's possible to consolidate multiple tasks or operations into a single computational unit.

Tasks can be grouped according to criteria based on the features provided by the environment and the costs associated with these features. A common approach is to look for tasks that have a similar profile concerning their scalability, lifetime, and processing requirements. Grouping these together allows them to scale as a unit (Pod?).

#### Considerations

* Scalability and elasticity.
    Many cloud solutions implement scalability and elasticity at the level of the computational unit by starting and stopping instances of units. Avoid grouping tasks that have conflicting scalability requirements in the same computational unit.
* Lifetime.
    The cloud infrastructure periodically recycles the virtual environment that hosts a computational unit. When there are many long-running tasks inside a computational unit, it might be necessary to configure the unit to prevent it from being recycled until these tasks have finished. Alternatively, design the tasks by using a check-pointing approach that enables them to stop cleanly, and continue at the point they were interrupted when the computational unit is restarted.
* Release cadence.
    If the implementation or configuration of a task changes frequently, it might be necessary to stop the computational unit hosting the updated code, reconfigure and redeploy the unit, and then restart it. This process will also require that all other tasks within the same computational unit are stopped, redeployed, and restarted.
* Security.
    Tasks in the same computational unit might share the same security context and be able to access the same resources. There must be a high degree of trust between the tasks, and confidence that one task isn't going to corrupt or adversely affect another. Additionally, increasing the number of tasks running in a computational unit increases the attack surface of the unit. Each task is only as secure as the one with the most vulnerabilities.
* Fault tolerance.
    If one task in a computational unit fails or behaves abnormally, it can affect the other tasks running within the same unit. For example, if one task fails to start correctly it can cause the entire startup logic for the computational unit to fail, and prevent other tasks in the same unit from running.
* Contention.
    Avoid introducing contention between tasks that compete for resources in the same computational unit. Ideally, tasks that share the same computational unit should exhibit different resource utilization characteristics.
* Complexity.
    Combining multiple tasks into a single computational unit adds complexity to the code in the unit, possibly making it more difficult to test, debug, and maintain.
* Stable logical architecture.
    Design and implement the code in each task so that it shouldn't need to change, even if the physical environment the task runs in does change.

### External Configuration Store Pattern

#### Problem

Often changes to the configuration files require the application be redeployed, often resulting in unacceptable downtime and other administrative overhead. Local configuration files also limit the configuration to a single application. Many configuration systems don't support different versions of configuration information.

#### Solution

Store the configuration information in external storage, and provide an interface that can be used to quickly and efficiently read and update configuration settings. The type of external store depends on the hosting and runtime environment of the application. In a cloud-hosted scenario it's typically a cloud-based storage service, but could be a hosted database or other system.

The backing store you choose for configuration information should have an interface that provides consistent and easy-to-use access. It should expose the information in a correctly typed and structured format. The implementation might also need to authorize users’ access in order to protect configuration data, and be flexible enough to allow storage of multiple versions of the configuration. It might be helpful to implement a caching mechanism within the external configuration store

#### Considerations

* Choose a backing store 
    that offers acceptable performance, high availability, robustness, and can be backed up as part of the application maintenance and administration process. In a cloud-hosted application, using a cloud storage mechanism is usually a good choice to meet these requirements.
* Design the schema of the backing store 
    to allow flexibility in the types of information it can hold. Ensure that it provides for all configuration requirements such as typed data, collections of settings, multiple versions of settings, and any other features that the applications using it require. The schema should be easy to extend to support additional settings as requirements change.
* Consider how the configuration interface will permit control of the scope and inheritance of configuration settings.
* Consider how to protect the configuration data to allow access to only the appropriate users and applications.
* Consider how the configuration store interface will behave when settings contain errors, or don't exist in the backing store. It might be appropriate to return default settings and log errors.
* If an application caches configuration information, the application needs to be alerted if the configuration changes.

#### Usage

This pattern is useful for:

* Configuration settings that are shared between multiple applications and application instances, or where a standard configuration must be enforced across multiple applications and application instances.
* A standard configuration system that doesn't support all of the required configuration settings, such as storing images or complex data types.
* As a complementary store for some of the settings for applications, perhaps allowing applications to override some or all of the centrally-stored settings.
* As a way to simplify administration of multiple applications, and optionally for monitoring use of configuration settings by logging some or all types of access to the configuration store.

### Gateway Aggregation Pattern

#### Problem

An application that relies on many services to perform a task must expend resources on each request. This chattiness between a client and a backend can adversely impact the performance and scale of the application. Microservice architectures have made this problem more common.

#### Solution

Use a gateway to reduce chattiness between the client and the services. The gateway receives client requests, dispatches requests to the various backend systems, and then aggregates the results and sends them back to the requesting client.
This pattern can reduce the number of requests that the application makes to backend services, and improve application performance over high-latency networks.

#### Considerations
* The gateway should not introduce service coupling across the backend services.
* The gateway should be located near the backend services to reduce latency as much as possible.
* The gateway service may introduce a single point of failure. Ensure the gateway is properly designed to meet your application's availability requirements.
* The gateway may introduce a bottleneck. Ensure the gateway has adequate performance to handle load and can be scaled to meet your anticipated growth.
* Perform load testing against the gateway to ensure you don't introduce cascading failures for services.
* Implement a resilient design, using techniques such as bulkheads, circuit breaking, retry, and timeouts.
* If one or more service calls takes too long, it may be acceptable to timeout and return a partial set of data. Consider how your application will handle this scenario.
* Use asynchronous I/O to ensure that a delay at the backend doesn't cause performance issues in the application.
* Implement distributed tracing using correlation IDs to track each individual call.
* Monitor request metrics and response sizes.
* Consider returning cached data as a failover strategy to handle failures.
* Instead of building aggregation into the gateway, consider placing an aggregation service behind the gateway. Request aggregation will likely have different resource requirements than other services in the gateway and may impact the gateway's routing and offloading functionality.

#### Usage

Use this pattern when:

* A client needs to communicate with multiple backend services to perform an operation.
* The client may use networks with significant latency, such as cellular networks.

This pattern may not be suitable when:

* You want to reduce the number of calls between a client and a single service across multiple operations. In that scenario, it may be better to add a batch operation to the service.
* The client or application is located near the backend services and latency is not a significant factor.

### Gateway Routing Pattern

#### Problem

When a client needs to consume multiple services, setting up a separate endpoint for each service and having the client manage each endpoint can be challenging. If an API changes, the client must be updated as well.

#### Solution

Place a gateway in front of a set of applications, services, or deployments. Use application Layer 7 routing to route the request to the appropriate instances.

With this pattern, the client application only needs to know about and communicate with a single endpoint. If a service is consolidated or decomposed, the client does not necessarily require updating. It can continue making requests to the gateway, and only the routing changes.

A gateway also lets you abstract backend services from the clients, allowing you to keep client calls simple while enabling changes in the backend services behind the gateway. Client calls can be routed to whatever service or services need to handle the expected client behavior, allowing you to add, split, and reorganize services behind the gateway without changing the client.

Routing lets you control what version of the service is presented to the clients, giving you the flexibility to use various release strategies, whether incremental, parallel, or complete rollouts of updates. Any issues discovered after the new service is deployed can be quickly reverted by making a configuration change at the gateway, without affecting clients.

#### Considerations

* The gateway service may introduce a single point of failure. Ensure it is properly designed to meet your availability requirements.
* Consider resiliency and fault tolerance capabilities when implementing.
* The gateway service may introduce a bottleneck. Ensure the gateway has adequate performance to handle load and can easily scale in line with your growth expectations.
* Perform load testing against the gateway to ensure you don't introduce cascading failures for services.
* Gateway routing is level 7. It can be based on IP, port, header, or URL.

#### Usage

Use this pattern when:

* A client needs to consume multiple services that can be accessed behind a gateway.
* You wish to simplify client applications by using a single endpoint.
* You need to route requests from externally addressable endpoints to internal virtual endpoints, such as exposing ports on a VM to cluster virtual IP addresses.

This pattern may not be suitable when you have a simple application that uses only one or two services.

### Gateway Offloading Pattern

#### Problem

Some features are commonly used across multiple services. A shared or specialized service that is distributed with every application deployment increases the administrative overhead, the likelihood of deployment error, and difficulty to implement and manage.

#### Solution

Offload some features into a gateway, particularly cross-cutting concerns such as certificate management, authentication, SSL termination, monitoring, protocol translation, or throttling.

Benefits of this pattern include:

* Simplify the development of services by removing the need to distribute and maintain supporting resources, such as web server certificates and configuration for secure websites. Simpler configuration results in easier management and scalability and makes service upgrades simpler.
* Allow dedicated teams to implement features that require specialized expertise, such as security. This allows your core team to focus on the application functionality, leaving these specialized but cross-cutting concerns to the relevant experts.
* Provide some consistency for request and response logging and monitoring. Even if a service is not correctly instrumented, the gateway can be configured to ensure a minimum level of monitoring and logging.

Benefits of this pattern include:

* Simplify the development of services by removing the need to distribute and maintain supporting resources, such as web server certificates and configuration for secure websites. Simpler configuration results in easier management and scalability and makes service upgrades simpler.
* Allow dedicated teams to implement features that require specialized expertise, such as security. This allows your core team to focus on the application functionality, leaving these specialized but cross-cutting concerns to the relevant experts.
* Provide some consistency for request and response logging and monitoring. Even if a service is not correctly instrumented, the gateway can be configured to ensure a minimum level of monitoring and logging.

#### Considerations

* Ensure the gateway is highly available and resilient to failure. Avoid single points of failure by running multiple instances of your gateway.
* Ensure the gateway is designed for the capacity and scaling requirements of your application and endpoints. Make sure the gateway does not become a bottleneck for the application and is sufficiently scalable.
* Only offload features that are used by the entire application, such as security or data transfer.
* Business logic should never be offloaded to the gateway.
* If you need to track transactions, consider generating correlation IDs for logging purposes.

#### Usage

Use this pattern when:

* An application deployment has a shared concern such as SSL certificates or encryption.
* A feature that is common across application deployments that may have different resource requirements, such as memory resources, storage capacity or network connections.
* You wish to move the responsibility for issues such as network security, throttling, or other network boundary concerns to a more specialized team.

This pattern may not be suitable if it introduces coupling across services.

### Leader Election Pattern

#### Problem

A typical cloud application has many tasks acting in a coordinated manner. These tasks could all be instances running the same code and requiring access to the same resources, or they might be working together in parallel to perform the individual parts of a complex calculation. The task instances might run separately for much of the time, but it might also be necessary to coordinate the actions of each instance. The task instances are all peers, so there isn't a natural leader that can act as the coordinator or aggregator.

#### Solution

A single task instance should be elected to act as the leader, and this instance should coordinate the actions of the other subordinate task instances. If all of the task instances are running the same code, they are each capable of acting as the leader. Therefore, the election process must be managed carefully to prevent two or more instances taking over the leader role at the same time.

The system must provide a robust mechanism for selecting the leader. This method has to cope with events such as network outages or process failures.

There are several strategies for electing a leader among a set of tasks in a distributed environment, including:

* Selecting the task instance with the lowest-ranked instance or process ID.
* Racing to acquire a shared, distributed mutex. The first task instance that acquires the mutex is the leader.
* Implementing one of the common leader election algorithms such as the Bully Algorithm or the Ring Algorithm. These algorithms assume that each candidate in the election has a unique ID, and that it can communicate with the other candidates reliably.

#### Considerations

* The process of electing a leader should be resilient to transient and persistent failures.
* How quickly detection (whether the leader has failed or has become otherwise unavailable) is needed is system dependent.
* In a system that implements horizontal autoscaling, the leader could be terminated if the system scales back and shuts down some of the computing resources.
* Using a shared, distributed mutex introduces a dependency on the external service that provides the mutex. The service constitutes a single point of failure.
* Using a single dedicated process as the leader is a straightforward approach. However, if the process fails there could be a significant delay while it's restarted. The resulting latency can affect the performance and response times of other processes if they're waiting for the leader to coordinate an operation.
* Implementing one of the leader election algorithms manually provides the greatest flexibility for tuning and optimizing the code.

#### Usage

Use this pattern when the tasks in a distributed application, such as a cloud-hosted solution, need careful coordination and there's no natural leader. Avoid making the leader a bottleneck in the system.

This pattern might not be useful if:
There's a natural leader or dedicated process that can always act as the leader.
The coordination between tasks can be achieved using a more lightweight method.
A third-party solution is more appropriate.

### Pipes and Filters Pattern

#### Problem

An application is required to perform a variety of tasks of varying complexity on the information that it processes. A monolithic module reduce the opportunities for refactoring the code, optimizing it, or reusing it.
The processing tasks performed by each module, or the deployment requirements for each task, could change as business requirements are updated. Some tasks might be compute intensive and could benefit from running on powerful hardware, while others might not require such expensive resources. Also, additional processing might be required in the future, or the order in which the tasks performed by the processing could change.
A solution is required that addresses these issues, and increases the possibilities for code reuse.

#### Solution

Break down the processing required for each stream into a set of separate components (or filters), each performing a single task. By standardizing the format of the data that each component receives and sends, these filters can be combined together into a pipeline. This helps to avoid duplicating code, and makes it easy to remove, replace, or integrate additional components if the processing requirements change.

A key advantage of the pipeline structure is that it provides opportunities for running parallel instances of slow filters, enabling the system to spread the load and improve throughput. The filters that make up a pipeline can run on different machines, enabling them to be scaled independently and take advantage of the elasticity that many cloud environments provide.
If the input and output of a filter are structured as a stream, it's possible to perform the processing for each filter in parallel (all the filters are not idle).
Another benefit is the resiliency that this model can provide.

#### Considerations

* Complexity.
    The increased flexibility that this pattern provides can also introduce complexity, especially if the filters in a pipeline are distributed across different servers.
* Reliability.
    Use an infrastructure that ensures that data flowing between filters in a pipeline won't be lost.
* Idempotency.
    When failure and retry happens, part of the work might be updated repeatedly (maybe by another instance of the filter). Therefore filters in a pipeline should be designed to be idempotent. For more information, see [Idempotency Patterns](https://blog.jonathanoliver.com/idempotency-patterns) on Jonathan Oliver’s blog.
* Repeated messages.
    Same as above, two instances of the same message might be passed to the next filter. To avoid this, the pipeline should detect and eliminate duplicate messages.
* Context and state.
    In a pipeline, each filter essentially runs in isolation and shouldn't make any assumptions about how it was invoked. This means that each filter should be provided with sufficient context to perform its work. This context could include a large amount of state information.

#### Usage

Use this pattern when:

* The processing required by an application can easily be broken down into a set of independent steps.
* The processing steps performed by an application have different scalability requirements.
    > It's possible to group filters that should scale together in the same process. For more information, see the Compute Resource Consolidation pattern.
* Flexibility is required to allow reordering of the processing steps performed by an application, or the capability to add and remove steps.
* The system can benefit from distributing the processing for steps across different servers.
* A reliable solution is required that minimizes the effects of failure in a step while data is being processed.

This pattern might not be useful when:

* The processing steps performed by an application aren't independent, or they have to be performed together as part of the same transaction.
* The amount of context or state information required by a step makes this approach inefficient. It might be possible to persist state information to a database instead, but don't use this strategy if the additional load on the database causes excessive contention.
