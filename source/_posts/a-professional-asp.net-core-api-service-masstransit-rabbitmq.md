---
title: A Professional ASP.NET Core API Service - MassTransit/RabbitMQ
date: October 9 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - masstransit
    - rabbitmq
    - messagebroker
---
 
`RabbitMQ` is an open-source message-broker software that originally implemented the Advanced Message Queuing Protocol and has since been extended with a plug-in architecture to support Streaming Text Oriented Messaging Protocol, MQ Telemetry Transport, and other protocols.

<!-- more -->

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

* https://www.codewithmukesh.com/blog/rabbitmq-with-aspnet-core-microservice/
* https://alvinvafana.blogspot.com/2019/10/messaging-through-service-bus-using.html
