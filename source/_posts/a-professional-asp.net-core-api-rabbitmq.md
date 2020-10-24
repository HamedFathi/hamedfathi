---
title: A Professional ASP.NET Core API - RabbitMQ
date: October 23 2020
category: aspnetcore-api
tags:
    - dotnet
    - aspnetcore
    - webapi
    - masstransit
    - rabbitmq
    - easynetq
    - amqp
    - queue
    - messagebroker
---
 
`RabbitMQ` is an open-source message-broker software that originally implemented the Advanced Message Queuing Protocol (AMQP) and has since been extended with a plug-in architecture to support Streaming Text Oriented Messaging Protocol, MQ Telemetry Transport, and other protocols.

A `message broker` acts as a `middleman` for various services (e.g. a web application, as in this example). They can be used to reduce loads and delivery times of web application servers by delegating tasks that would normally take up a lot of time or resources to a third party that has no other job.

<!-- more -->

![](/images/a-professional-asp.net-core-api-rabbitmq/1.png)

A `message` can include any kind of information. It could, for example, have information about a process or task that should start on another application (which could even be on another server), or it could be just a simple text message. The queue-manager software stores the messages until a receiving application connects and takes a message off the queue. The receiving application then processes the message.

When the user has entered user information into the web interface, the web application will create a "PDF processing" message that includes all of the important information the user needs into a message and place it onto a queue defined in `RabbitMQ`.

![](/images/a-professional-asp.net-core-api-rabbitmq/2.png)

The basic architecture of a message queue is simple - there are client applications called `producers` that create messages and deliver them to the broker (the message queue). Other applications, called `consumers`, connect to the queue and `subscribe` to the messages to be processed. Software may act as a producer, or consumer, or both a consumer and a producer of messages. Messages placed onto the queue are stored until the consumer retrieves them.

## When and why should you use RabbitMQ?

Message queueing allows web servers to respond to requests quickly instead of being forced to perform resource-heavy procedures on the spot that may delay response time. Message queueing is also good when you want to distribute a message to multiple consumers or to balance loads between workers.

The consumer takes a message off the queue and starts processing the PDF. At the same time, the producer is queueing up new messages. The consumer can be on a totally different server than the producer or they can be located on the same server. The request can be created in one programming language and handled in another programming language. The point is, the two applications will only communicate through the messages they are sending to each other, which means the sender and receiver have `low coupling`.

![](/images/a-professional-asp.net-core-api-rabbitmq/3.png)

1. The user sends a PDF creation request to the web application.
2. The web application (the producer) sends a message to RabbitMQ that includes data from the request such as name and email.
3. An exchange accepts the messages from the producer and routes them to correct message queues for PDF creation.
4. The PDF processing worker (the consumer) receives the task message and starts processing the PDF.

## Exchanges

Messages are not published directly to a queue; instead, the producer sends messages to an exchange. An exchange is responsible for `routing` the messages to different queues with the help of bindings and routing keys.

![](/images/a-professional-asp.net-core-api-rabbitmq/4.png)

* **Binding** is a `link` that you set up to bind a queue to an exchange (A link between a queue and an exchange).
* **Routing key** is a message attribute the exchange looks at when deciding how to route the message to queues (depending on exchange type).

Exchanges, connections, and queues can be configured with parameters such as durable, temporary, and auto delete upon creation. Durable exchanges survive server restarts and last until they are explicitly deleted. Temporary exchanges exist until RabbitMQ is shut down. Auto-deleted exchanges are removed once the last bound object is unbound from the exchange.

In RabbitMQ, there are four different types of exchanges that route the message differently using different parameters and bindings setups. Clients can create their own exchanges or use the predefined default exchanges which are created when the server starts for the first time.

**Message flow in RabbitMQ**

1. The producer publishes a message to an exchange. When creating an exchange, the type must be specified. This topic will be covered later on.
2. The exchange receives the message and is now responsible for routing the message. The exchange takes different message attributes into account, such as the routing key, depending on the exchange type.
3. Bindings must be created from the exchange to queues. In this case, there are two bindings to two different queues from the exchange. The exchange routes the message into the queues depending on message attributes.
4. The messages stay in the queue until they are handled by a consumer
5. The consumer handles the message.

**Types of exchanges**

![](/images/a-professional-asp.net-core-api-rabbitmq/5.png)

* **Direct**: The message is routed to the queues whose binding key exactly matches the routing key of the message. For example, if the queue is bound to the exchange with the binding key pdfprocess, a message published to the exchange with a routing key pdfprocess is routed to that queue.

* **Fanout**: A fanout exchange routes messages to all of the queues bound to it.

* **Topic**: The topic exchange does a wildcard match between the routing key and the routing pattern specified in the binding.

* **Headers**: Headers exchanges use the message header attributes for routing.

## Direct Exchange

A direct exchange delivers messages to queues based on a message routing key. The routing key is a message attribute added to the message header by the producer. Think of the routing key as an "address" that the exchange is using to decide how to route the message. `A message goes to the queue(s) with the binding key that exactly matches the routing key of the message`.

The direct exchange type is useful to distinguish messages published to the same exchange using a simple string identifier.

The default exchange AMQP brokers must provide for the direct exchange is "amq.direct".

Imagine that queue A (create_pdf_queue) in the image below (Direct Exchange Figure) is bound to a direct exchange (pdf_events) with the binding key pdf_create. When a new message with routing key pdf_create arrives at the direct exchange, the exchange routes it to the queue where the binding_key = routing_key, in the case to queue A (create_pdf_queue).

![](/images/a-professional-asp.net-core-api-rabbitmq/direct-exchange.png)

**Scenario 1**

* Exchange: pdf_events
* Queue A: create_pdf_queue
* Binding key between exchange (pdf_events) and Queue A (create_pdf_queue): pdf_create

**Scenario 2**

* Exchange: pdf_events
* Queue B: pdf_log_queue
* Binding key between exchange (pdf_events) and Queue B (pdf_log_queue): pdf_log

**Example**

A message with routing key pdf_log is sent to the exchange pdf_events. The messages is routed to pdf_log_queue because the routing key (pdf_log) matches the binding key (pdf_log).

If the message routing key does not match any binding key, the message is discarded.

**Default Exchange**

The default exchange is a pre-declared `direct exchange with no name`, usually referred by an empty string. When you use default exchange, your message is delivered to the queue with a name equal to the routing key of the message. Every queue is automatically bound to the default exchange with a routing key which is the same as the queue name.

## Topic Exchange

Topic exchanges route messages to queues based on wildcard matches between the routing key and the routing pattern, which is specified by the queue binding. Messages are routed to one or many queues based on a matching between a message routing key and this pattern.

The routing key must be a list of words, delimited by a period (.). Examples are agreements.us and agreements.eu.stockholm which in this case identifies agreements that are set up for a company with offices in lots of different locations. The routing patterns may contain an asterisk (“*”) to match a word in a specific position of the routing key (e.g., a routing pattern of "agreements.*.*.b.*" only match routing keys where the first word is "agreements" and the fourth word is "b"). A pound symbol (“#”) indicates a match of zero or more words (e.g., a routing pattern of "agreements.eu.berlin.#" matches any routing keys beginning with "agreements.eu.berlin").

The consumers indicate which topics they are interested in (like subscribing to a feed for an individual tag). The consumer creates a queue and sets up a binding with a given routing pattern to the exchange. All messages with a routing key that match the routing pattern are routed to the queue and stay there until the consumer consumes the message.

The default exchange AMQP brokers must provide for the topic exchange is "amq.topic".

![](/images/a-professional-asp.net-core-api-rabbitmq/topic-exchange.png)

**Scenario 1**

The image to the right shows an example where consumer A is interested in all the agreements in Berlin.

* Exchange: agreements
* Queue A: berlin_agreements
* Routing pattern between exchange (agreements) and Queue A (berlin_agreements): agreements.eu.berlin.#
* Example of message routing key that matches: agreements.eu.berlin and agreements.eu.berlin.headstore

**Scenario 2**

Consumer B is interested in all the agreements.

* Exchange: agreements
* Queue B: all_agreements
* Routing pattern between exchange (agreements) and Queue B (all_agreements): agreements.#
* Example of message routing key that matches: agreements.eu.berlin and agreements.us

**Scenario 3**

Consumer C is interested in all agreements for European head stores.

* Exchange: agreements
* Queue C: headstore_agreements
* Routing pattern between exchange (agreements) and Queue C (headstore_agreements): agreements.eu.*.headstore
* Example of message routing keys that will match: agreements.eu.berlin.headstore and agreements.eu.stockholm.headstore

**Example**

A message with routing key agreements.eu.berlin is sent to the exchange agreements. The messages are routed to the queue berlin_agreements because the routing pattern of "agreements.eu.berlin.#" matches the routing keys beginning with "agreements.eu.berlin". The message is also routed to the queue all_agreements because the routing key (agreements.eu.berlin) matches the routing pattern (agreements.#).

## Fanout Exchange

A fanout exchange copies and routes a received message to all queues that are bound to it regardless of routing keys or pattern matching as with direct and topic exchanges. The keys provided will simply be ignored.

Fanout exchanges can be useful when the same message needs to be sent to one or more queues with consumers who may process the same message in different ways.

The image to the right (Fanout Exchange) shows an example where a message received by the exchange is copied and routed to all three queues bound to the exchange. It could be sport or weather updates that should be sent out to each connected mobile device when something happens, for instance.

The default exchange AMQP brokers must provide for the topic exchange is "amq.fanout".

![](/images/a-professional-asp.net-core-api-rabbitmq/fanout-exchange.png)

**Scenario 1**

* Exchange: sport_news
* Queue A: Mobile client queue A
* Binding: Binding between the exchange (sport_news) and Queue A (Mobile cli
ent queue A)

**Example**

A message is sent to the exchange sport_news. The message is routed to all queues (Queue A, Queue B, Queue C) because all queues are bound to the exchange. Provided routing keys are ignored.

## Headers Exchange

A headers exchange routes messages based on arguments containing headers and optional values. Headers exchanges are very similar to topic exchanges, but route messages based on header values instead of routing keys. A message matches if the value of the header equals the value specified upon binding.

A special argument named "x-match", added in the binding between exchange and queue, specifies if all headers must match or just one. Either any common header between the message and the binding count as a match, or all the headers referenced in the binding need to be present in the message for it to match. The "x-match" property can have two different values: "any" or "all", where "all" is the default value. A value of "all" means all header pairs (key, value) must match, while value of "any" means at least one of the header pairs must match. Headers can be constructed using a wider range of data types, integer or hash for example, instead of a string. The headers exchange type (used with the binding argument "any") is useful for directing messages which contain a subset of known (unordered) criteria.

The default exchange AMQP brokers must provide for the topic exchange is "amq.headers".

![](/images/a-professional-asp.net-core-api-rabbitmq/headers-exchange.png)

**Example**

* Exchange: Binding to Queue A with arguments (key = value): format = pdf, type = report, x-match = all
* Exchange: Binding to Queue B with arguments (key = value): format = pdf, type = log, x-match = any
* Exchange: Binding to Queue C with arguments (key = value): format = zip, type = report, x-match = all

**Scenario 1**

Message 1 is published to the exchange with header arguments (key = value): "format = pdf", "type = report".

Message 1 is delivered to Queue A because all key/value pairs match, and Queue B since "format = pdf" is a match (binding rule set to "x-match =any").

**Scenario 2**

Message 2 is published to the exchange with header arguments of (key = value): "format = pdf".

Message 2 is only delivered to Queue B. Because the binding of Queue A requires both "format = pdf" and "type = report" while Queue B is configured to match any key-value pair (x-match = any) as long as either "format = pdf" or "type = log" is present.

**Scenario 3**

Message 3 is published to the exchange with header arguments of (key = value): "format = zip", "type = log".

Message 3 is delivered to Queue B since its binding indicates that it accepts messages with the key-value pair "type = log", it doesn't mind that "format = zip" since "x-match = any".

Queue C doesn't receive any of the messages since its binding is configured to match all of the headers ("x-match = all") with "format = zip", "type = pdf". No message in this example lives up to these criterias.

It's worth noting that in a header exchange, the actual order of the key-value pairs in the message is irrelevant.

## Dead Letter Exchange

If no matching queue can be found for the message, the message is silently dropped. RabbitMQ provides an AMQP extension known as the "Dead Letter Exchange", which provides the functionality to capture messages that are not deliverable.

## RabbitMQ installation (Windows)

**Install Erlang/OTP**

`RabbitMQ` requires a 64-bit supported version of [Erlang](https://www.erlang.org/downloads) for Windows to be installed. 

Set `ERLANG_HOME` to where you actually put your Erlang installation, e.g. `C:\Program Files\erl{version}` (full path). The RabbitMQ batch files expect to execute `%ERLANG_HOME%\bin\erl.exe`.

Go to `Start > Settings > Control Panel > System > Advanced > Environment Variables`. Create the system environment variable `ERLANG_HOME` and set it to the full path of the directory which contains `bin\erl.exe`.

**Install RabbitMQ Server**

After making sure a supported Erlang version is installed, download [RabbitMQ server](https://github.com/rabbitmq/rabbitmq-server/releases).

**Enable RabbitMQ Management Plugin**

Go to the directory where the `RabbitMQ` is installed.

Now, enable the `rabbitmq_management` plugin using the `rabbitmq-plugins` command as shown below.

```bash
sbin/rabbitmq-plugins enable rabbitmq_management
```

The `rabbitmq_management` plugin is a combination of the following plugins. All of the following plugins will be enabled when you execute the above command:

* mochiweb
* webmachine
* rabbitmq_web_dispatch
* amqp_client
* rabbitmq_management_agent
* rabbitmq_management

After enabling the `rabbitmq_management` plugin you should restart the RabbitMQ server as shown below.

```bash
sbin/rabbitmqctl stop

sbin/rabbitmq-server -detached
```

And we'll get an output like this:

```
Warning: PID file not written; -detached was passed.
```

**Login to RabbitMQ Management Dashboard**

By default the management plugin runs on `15672` HTTP port.

From your browser go to `http://localhost:15672`

The default username and password for RabbitMQ management plugin is: `guest`

![](/images/a-professional-asp.net-core-api-rabbitmq/dashboard.png)

## RabbitMQ Management Dashboard





## EasyNetQ

A Nice .NET API for RabbitMQ.

Install the below packages

```bash
Install-Package EasyNetQ -Version 5.6.0
dotnet add package EasyNetQ --version 5.6.0
<PackageReference Include="EasyNetQ" Version="5.6.0" />

Install-Package EasyNetQ.DI.Microsoft -Version 5.6.0
dotnet add package EasyNetQ.DI.Microsoft --version 5.6.0
<PackageReference Include="EasyNetQ.DI.Microsoft" Version="5.6.0" />
```

## MassTransit

A free, open-source distributed application framework for .NET.

Install the below packages

```bash
Install-Package MassTransit -Version 7.0.4
dotnet add package MassTransit --version 7.0.4
<PackageReference Include="MassTransit" Version="7.0.4" />

Install-Package MassTransit.AspNetCore -Version 7.0.4
dotnet add package MassTransit.AspNetCore --version 7.0.4
<PackageReference Include="MassTransit.AspNetCore" Version="7.0.4" />

Install-Package MassTransit.RabbitMQ -Version 7.0.4
dotnet add package MassTransit.RabbitMQ --version 7.0.4
<PackageReference Include="MassTransit.RabbitMQ" Version="7.0.4" />
```

## Reference(s)

Most of the information in this article has gathered from various references.

* https://www.cloudamqp.com/blog/2015-05-18-part1-rabbitmq-for-beginners-what-is-rabbitmq.html
* https://www.cloudamqp.com/blog/2015-05-27-part3-rabbitmq-for-beginners_the-management-interface.html
* https://www.cloudamqp.com/blog/2015-09-03-part4-rabbitmq-for-beginners-exchanges-routing-keys-bindings.html
* https://www.codewithmukesh.com/blog/rabbitmq-with-aspnet-core-microservice/
* https://alvinvafana.blogspot.com/2019/10/messaging-through-service-bus-using.html
* https://www.rabbitmq.com/install-windows-manual.html
* https://www.thegeekstuff.com/2013/10/enable-rabbitmq-management-plugin/
* https://www.codementor.io/@bosunbolawa/how-to-enable-rabbitmq-management-interface-owc5lzg7f