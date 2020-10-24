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

Messages are not published directly to a queue; instead, the producer sends messages to an exchange. An exchange is responsible for `routing` the messages to different queues with the help of bindings and routing keys. A binding is a link between a queue and an exchange.

![](/images/a-professional-asp.net-core-api-rabbitmq/4.png)

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
* https://www.thegeekstuff.com/2013/10/enable-rabbitmq-management-plugin/
* https://www.codementor.io/@bosunbolawa/how-to-enable-rabbitmq-management-interface-owc5lzg7f