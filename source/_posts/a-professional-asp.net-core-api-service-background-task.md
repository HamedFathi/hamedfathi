---
title: A Professional ASP.NET Core API Service - Background Task
date: September 22 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - backgroundtask
    - hostedservices
---

In ASP.NET Core, background tasks can be implemented as `hosted services`. A hosted service is a class with background task logic that implements the `IHostedService` interface. This topic provides three hosted service examples:

* Background task that runs on a timer.
* Hosted service that activates a scoped service. The scoped service can use dependency injection (DI).
* Queued background tasks that run sequentially.

<!-- more -->

## Quartz scheduler 





## Reference(s)

Most of the information in this article is from various sources.

* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-3.1&* tabs=visual-studio
* https://andrewlock.net/creating-a-quartz-net-hosted-service-with-asp-net-core/
* https://www.infoworld.com/article/3529418/how-to-schedule-jobs-using-quartznet-in-aspnet-core.html