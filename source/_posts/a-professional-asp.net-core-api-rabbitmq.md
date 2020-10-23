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
 
`RabbitMQ` is an open-source message-broker software that originally implemented the Advanced Message Queuing Protocol and has since been extended with a plug-in architecture to support Streaming Text Oriented Messaging Protocol, MQ Telemetry Transport, and other protocols.

<!-- more -->

`RabbitMQ` is a message-queueing software also known as a message broker or queue manager. Simply said; it is software where queues are defined, to which applications connect in order to transfer a message or messages.

A message broker acts as a middleman for various services (e.g. a web application, as in this example). They can be used to reduce loads and delivery times of web application servers by delegating tasks that would normally take up a lot of time or resources to a third party that has no other job.

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
* https://www.codewithmukesh.com/blog/rabbitmq-with-aspnet-core-microservice/
* https://alvinvafana.blogspot.com/2019/10/messaging-through-service-bus-using.html
