---
layout: post
title:  "Cloud Design - Data Management patterns"
categories: Software
tags: design
excerpt: "These design patterns are useful for building reliable	 scalable	 secure applications in the cloud."
---

* content
{:toc}

## Message Patterns

### Asynchronous Request-Reply Pattern

#### Problem

In some scenarios, however, the work done by backend may be long-running, on the order of seconds, or might be a background process that is executed in minutes or even hours. In that case, it isn't feasible to wait for the work to complete before responding to the request. This situation is a potential problem for any synchronous request-reply pattern.

#### Solution

One solution to this problem is to use HTTP polling. Polling is useful to client-side code, as it can be hard to provide call-back endpoints or use long running connections.

1. The client application makes a synchronous call to the API, triggering a long-running operation on the backend.
2. The API should validate both the request and the action to be performed.
3. The API responds synchronously as quickly as possible. It returns an HTTP 202 (Accepted) status code, acknowledging that the request has been received for processing.
4. The response holds a location reference pointing to an endpoint that the client can poll to check for the result of the long running operation.
5. The API offloads processing to another component, such as a message queue.
6. While the work is still pending, the status endpoint returns HTTP 202. Once the work is complete, the status endpoint can either return a resource that indicates completion, or redirect to another resource URL.

#### Considerations

* An HTTP 202 response should indicate the location and frequency that the client should poll for the response.
* You may need to use a processing proxy or facade to manipulate the response headers or payload depending on the underlying services used.
* If the status endpoint redirects on completion, either HTTP 302 or HTTP 303 are appropriate return codes, depending on the exact semantics you support.
* Upon successful processing, the resource specified by the Location header should return an appropriate HTTP response code such as 200 (OK), 201 (Created), or 204 (No Content).
* If an error occurs during processing, persist the error at the resource URL described in the Location header and ideally return an appropriate response code to the client from that resource (4xx code).
* Not all solutions will implement this pattern in the same way and some services will include additional or alternate headers.
* Legacy clients might not support this pattern. In that case, you might need to place a facade over the asynchronous API to hide the asynchronous processing from the original client.
* In some scenarios, you might want to provide a way for clients to cancel a long-running request.

#### Usage

Use this pattern for client-side codes or service calls where it's difficult to provide or fire callbacks.

This pattern might not be suitable when:

* You can use a service built for asynchronous notifications instead.
* Responses must stream in real time to the client.
* The client needs to collect many results, and received latency of those results is important. Consider a service bus pattern instead.
* You can use server-side persistent network connections such as WebSockets or SignalR. These services can be used to notify the caller of the result.
* The network design allows you to open up ports to receive asynchronous callbacks or webhooks.

### Claim-Check Pattern (Reference-Based Messaging)

#### Problem

Sending large messages to the message bus directly is not recommended, because they require more resources and bandwidth to be consumed. Large messages can also slow down the entire solution, because messaging platforms are usually fine-tuned to handle huge quantities of small messages. Also, most messaging platforms have limits on message size, so you may need to work around these limits for large messages.

#### Solution

Store the entire message payload into an external service, such as a database. Get the reference to the stored payload, and send just that reference to the message bus. The reference acts like a claim check used to retrieve a piece of luggage.

#### Considerations

* Consider deleting the message data after consuming it, if you don't need to archive the messages. Deleting the message can be done synchronously by the application that receives and processes the message, or asynchronously by a separate dedicated process.
* Storing and retrieving the message causes some additional overhead and latency. You may want to implement logic in the sending application to use this pattern only when the message size exceeds the data limit of the message bus. The pattern would be skipped for smaller messages. This approach would result in a conditional claim-check pattern.

#### Usage

* This pattern should be used whenever a message cannot fit the supported message limit of the chosen message bus technology.
* The pattern can also be used if the payload should be accessed only by services that are authorized to see it. By offloading the payload to an external resource, stricter authentication and authorization rules can be put in place, to ensure that security is enforced when sensitive data is stored in the payload.
* [Dealing with large Service Bus messages using claim check pattern](https://www.serverless360.com/blog/deal-with-large-service-bus-messages-using-claim-check-pattern)

### Choreography Pattern

#### Problem

The orchestrator pattern reduces point-to-point communication between services but has some drawbacks because of the tight coupling between the orchestrator and other services that participate in processing of the business transaction.
To execute tasks in a sequence, the orchestrator needs to have some domain knowledge about the responsibilities of those services. If you want to add or remove services, existing logic will break, and you'll need to rewire portions of the communication path.

#### Solution

Let each service decide when and how a business operation is processed, instead of depending on a central orchestrator. One way to implement choreography is to use the asynchronous messaging pattern to coordinate the business operations.

A client request publishes messages to a message queue. As messages arrive, they are pushed to subscribers, or services, interested in that message. Each subscribed service does their operation as indicated by the message and responds to the message queue with success or failure of the operation. In case of success, the service can push a message back to the same queue or a different message queue so that another service can continue the workflow if needed. If an operation fails, the message bus can retry that operation.

This way, the services choreograph the workflow among themselves without depending on an orchestrator or having direct communication between them.

Because there isn't point-to-point communication, this pattern helps reduce coupling between services. Also, it can remove the performance bottleneck caused by the orchestrator when it has to deal with all transactions.

#### Usage

* Use the choreography pattern if you expect to update, remove, or add new services frequently. The entire app can be modified with lesser effort and minimal disruption to existing services.
* Consider this pattern if you experience performance bottlenecks in the central orchestrator.
* This pattern is a natural model for the serverless architecture where all services can be short lived, or event driven. Services can spin up because of an event, do their task, and are removed when the task is finished.

#### Considerations

Decentralizing the orchestrator can cause issues while managing the workflow.

* If a service fails to complete a business operation, it can be difficult to recover from that failure.
* The workflow can become complicated when choreography needs to occur in a sequence. One approach is to have multiple message buses that get messages in the required order.
* The choreography pattern becomes a challenge if the number of services grow rapidly. Given the high number of independent moving parts, the workflow between services tends to get complex. Also, distributed tracing becomes difficult.
* For choreography, the role is distributed between all services and resiliency becomes less robust.
* Each service isn't only responsible for the resiliency of its operation but also the workflow. This responsibility can be burdensome for the service and hard to implement. Each service must retry transient, nontransient, and time-out failures, so that the request terminates gracefully, if needed. Also, the service must be diligent about communicating the success or failure of the operation so that other services can act accordingly.

### Publisher-Subscriber Pattern

#### Problem

In cloud-based and distributed applications, components of the system often need to provide information to other components as events happen.
Using a dedicated message queue for each consumer does not effectively scale to many consumers, and some of the consumers might be interested in only a subset of the information.

#### Solution

Introduce an asynchronous messaging subsystem that includes:

* An input messaging channel used by the sender. The sender packages events into messages, using a known message format, and sends these messages via the input channel. The sender in this pattern is also called the publisher.
* One output messaging channel per consumer. The consumers are known as subscribers.
* A mechanism for copying each message from the input channel to the output channels for all subscribers interested in that message. This operation is typically handled by an intermediary such as a message broker or event bus.

Benefits:

* It decouples subsystems that still need to communicate. Subsystems can be managed independently, and messages can be properly managed even if one or more receivers are offline.
* It increases scalability and improves responsiveness of the sender. The sender can quickly send a single message to the input channel, then return to its core processing responsibilities. The messaging infrastructure is responsible for ensuring messages are delivered to interested subscribers.
* It improves reliability. Asynchronous messaging helps applications continue to run smoothly under increased loads and handle intermittent failures more effectively.
* It allows for deferred or scheduled processing. Subscribers can wait to pick up messages until off-peak hours, or messages can be routed or processed according to a specific schedule.
* It enables simpler integration between systems using different platforms, programming languages, or communication protocols, as well as between on-premises systems and applications running in the cloud.
* It facilitates asynchronous workflows across an enterprise.
* It improves testability. Channels can be monitored and messages can be inspected or logged as part of an overall integration test strategy.
* It provides separation of concerns for your applications. Each application can focus on its core capabilities, while the messaging infrastructure handles everything required to reliably route messages to multiple consumers.

#### Considerations

* Existing technologies. It is strongly recommended to use available messaging products and services that support a publish-subscribe model, rather than building your own.
* Subscription handling. The messaging infrastructure must provide mechanisms that consumers can use to subscribe to or unsubscribe from available channels.
* Security. Connecting to any message channel must be restricted by security policy to prevent eavesdropping by unauthorized users or applications.
* Subsets of messages. Subscribers are usually only interested in subset of the messages distributed by a publisher. Messaging services often allow subscribers to narrow the set of messages received by topics or content filtering.
* Wildcard subscribers. Consider allowing subscribers to subscribe to multiple topics via wildcards.
* Bi-directional communication. The channels in a publish-subscribe system are treated as unidirectional. If a specific subscriber needs to send acknowledgment or communicate status back to the publisher, consider using the Request/Reply Pattern.
* Message ordering. The order in which consumer instances receive messages isn't guaranteed, and doesn't necessarily reflect the order in which the messages were created. Design the system to ensure that message processing is idempotent to help eliminate any dependency on the order of message handling.
* Message priority. Some solutions may require that messages are processed in a specific order. The Priority Queue pattern provides a mechanism for ensuring specific messages are delivered before others.
* Poison messages. A malformed message, or a task that requires access to resources that aren't available, can cause a service instance to fail. The system should prevent such messages being returned to the queue. Instead, capture and store the details of these messages elsewhere so that they can be analyzed if necessary.
* Repeated messages. The same message might be sent more than once. For example, the sender might fail after posting a message. Then a new instance of the sender might start up and repeat the message. The messaging infrastructure should implement duplicate message detection and removal (also known as de-duping) based on message IDs in order to provide at-most-once delivery of messages.
* Message expiration. A message might have a limited lifetime. If it isn't processed within this period, it might no longer be relevant and should be discarded. A sender can specify an expiration time as part of the data in the message. A receiver can examine this information before deciding whether to perform the business logic associated with the message.
* Message scheduling. A message might be temporarily embargoed and should not be processed until a specific date and time. The message should not be available to a receiver until this time.

#### Usage

Use this pattern when:

* An application needs to broadcast information to a significant number of consumers.
* An application needs to communicate with one or more independently-developed applications or services, which may use different platforms, programming languages, and communication protocols.
* An application can send information to consumers without requiring real-time responses from the consumers.
* The systems being integrated are designed to support an eventual consistency model for their data.
* An application needs to communicate information to multiple consumers, which may have different availability requirements or uptime schedules than the sender.

This pattern might not be useful when:

* An application has only a few consumers who need significantly different information from the producing application.
* An application requires near real-time interaction with consumers.

### Queue-Based Load Leveling Pattern

#### Problem

If a service is used by a number of tasks running concurrently, it can be difficult to predict the volume of requests to the service at any time. A service might experience peaks in demand that cause it to overload and be unable to respond to requests in a timely manner. Flooding a service with a large number of concurrent requests can also result in the service failing if it's unable to handle the contention these requests cause.

#### Solution

Refactor the solution and introduce a queue between the task and the service. The task and the service run asynchronously. The queue acts as a buffer, storing the message until it's retrieved by the service.
The queue decouples the tasks from the service, and the service can handle the messages at its own pace regardless of the volume of requests from concurrent tasks. Additionally, there's no delay to a task if the service isn't available at the time it posts a message to the queue.

Benifits:

* It can help to maximize availability because delays arising in services won't have an immediate and direct impact on the application, which can continue to post messages to the queue even when the service isn't available or isn't currently processing messages.
* It can help to maximize scalability because both the number of queues and the number of services can be varied to meet demand.
* It can help to control costs because the number of service instances deployed only have to be adequate to meet average load rather than the peak load.

#### Considerations

* It's necessary to implement application logic that controls the rate at which services handle messages to avoid overwhelming the target resource. Avoid passing spikes in demand to the next stage of the system. Test the system under load to ensure that it provides the required leveling, and adjust the number of queues and the number of service instances that handle messages to achieve this.
* Message queues are a one-way communication mechanism. If a task expects a reply from a service, it might be necessary to implement a mechanism that the service can use to send a response.
* Be careful if you apply autoscaling to services that are listening for requests on the queue. This can result in increased contention for any resources that these services share and diminish the effectiveness of using the queue to level the load.

#### Usage

This pattern is useful to any application that uses services that are subject to overloading.
This pattern isn't useful if the application expects a response from the service with minimal latency.

### Priority Queue Pattern

#### Problem

In some cases, it's necessary to prioritize specific requests. These requests should be processed earlier than lower priority requests that were sent previously by the application.

#### Solution

A queue is usually a first-in, first-out (FIFO) structure, and consumers typically receive messages in the same order that they were posted to the queue. However, some message queues support priority messaging. The application posting a message can assign a priority and the messages in the queue are automatically reordered so that those with a higher priority will be received before those with a lower priority. The figure illustrates a queue with priority messaging.

n systems that don't support priority-based message queues, an alternative solution is to maintain a separate queue for each priority. The application is responsible for posting messages to the appropriate queue. Each queue can have a separate pool of consumers. Higher priority queues can have a larger pool of consumers running on faster hardware than lower priority queues.

A variation on this strategy is to have a single pool of consumers that check for messages on high priority queues first, and only then start to fetch messages from lower priority queues.

Benefits:

* It allows applications to meet business requirements that require prioritization of availability or performance, such as offering different levels of service to specific groups of customers.
* It can help to minimize operational costs.
* The multiple message queue approach can help maximize application performance and scalability by partitioning messages based on processing requirements.

#### Considerations

* Define the priorities in the context of the solution. For example, high priority could mean that messages should be processed within ten seconds. Identify the requirements for handling high priority items, and the other resources that should be allocated to meet these criteria.
* Decide if all high priority items must be processed before any lower priority items. If the messages are being processed by a single pool of consumers, you have to provide a mechanism that can preempt and suspend a task that's handling a low priority message if a higher priority message becomes available.
* In the multiple queue approach, when using a single pool of consumer processes that listen on all queues rather than a dedicated consumer pool for each queue, the consumer must apply an algorithm that ensures it always services messages from higher priority queues before those from lower priority queues.
* Monitor the processing speed on high and low priority queues to ensure that messages in these queues are processed at the expected rates.
* If you need to guarantee that low priority messages will be processed, it's necessary to implement the multiple message queue approach with multiple pools of consumers. Alternatively, in a queue that supports message prioritization, it's possible to dynamically increase the priority of a queued message as it ages.
* Using a separate queue for each message priority works best for systems that have a small number of well-defined priorities.
* Message priorities can be determined logically by the system.
* There might be a financial and processing cost associated with checking a queue for a message.
* It's possible to dynamically adjust the size of a pool of consumers based on the length of the queue that the pool is servicing.

#### Usage

* The system must handle multiple tasks that have different priorities.
* Different users or tenants should be served with different priority.

### Scheduler Agent Supervisor Pattern

#### Problem

If the application detects a more permanent fault it can't easily recover from, it must be able to restore the system to a consistent state and ensure integrity of the entire operation.

#### Solution

The Scheduler Agent Supervisor pattern defines the following actors. These actors orchestrate the steps to be performed as part of the overall task.

* The Scheduler arranges for the steps that make up the task to be executed and orchestrates their operation. These steps can be combined into a pipeline or workflow. The Scheduler is responsible for ensuring that the steps in this workflow are performed in the right order. As each step is performed, the Scheduler records the state of the workflow. The state information should also include an upper limit of the time allowed for the step to finish, called the complete-by time. If a step requires access to a remote service or resource, the Scheduler invokes the appropriate Agent, passing it the details of the work to be performed. The Scheduler typically communicates with an Agent using asynchronous request/response messaging. This can be implemented using queues, although other distributed messaging technologies could be used instead.

* The Agent contains logic that encapsulates a call to a remote service, or access to a remote resource referenced by a step in a task. Each Agent typically wraps calls to a single service or resource, implementing the appropriate error handling and retry logic. If the steps in the workflow being run by the Scheduler use several services and resources across different steps, each step might reference a different Agent.

* The Supervisor monitors the status of the steps in the task being performed by the Scheduler. It runs periodically and examines the status of steps maintained by the Scheduler. If it detects any that have timed out or failed, it arranges for the appropriate Agent to recover the step or execute the appropriate remedial action. The Supervisor should simply request that these actions be performed.

The Scheduler, Agent, and Supervisor are logical components and their physical implementation depends on the technology being used.

#### Considerations

* This pattern can be difficult to implement and requires thorough testing of each possible failure mode of the system.
* The recovery/retry logic implemented by the Scheduler is complex and dependent on state information held in the state store. It might also be necessary to record the information required to implement a compensating transaction in a durable data store.
* How often the Supervisor runs will be important. It should run often enough to prevent any failed steps from blocking an application for an extended period, but it shouldn't run so often that it becomes an overhead.
* The steps performed by an Agent could be run more than once. The logic that implements these steps should be idempotent.

#### Usage

Use this pattern when a process that runs in a distributed environment, such as the cloud, must be resilient to communications failure and/or operational failure.

This pattern might not be suitable for tasks that don't invoke remote services or access remote resources.

### Sequential Convoy Pattern

Process a set of related messages in a defined order, without blocking processing of other groups of messages.

-> Push related messages into categories within the queuing system, and have the queue listeners lock and pull only from one category, one message at a time

Considerations:

* Category/scale unit: session unit
* Throughput
* Service capabilities: message bus allow for one-at-a-time processing of messages within a queue or category of a queue
* Evolvability: add a new catogory / session id
* Special "end of sequence" flag in the last message of a transaction