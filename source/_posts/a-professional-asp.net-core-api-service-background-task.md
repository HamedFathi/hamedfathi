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


You should implement `IScheduledTask` if you want to have the ability to config the schedule for the task to run, for example 1 a days, every hours, etc. using cron expression.

Implement `BackgroundService` if you want to have a task always run continuously.

## Quartz scheduler 


## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-3.1&* tabs=visual-studio
* https://andrewlock.net/creating-a-quartz-net-hosted-service-with-asp-net-core/
* https://www.infoworld.com/article/3529418/how-to-schedule-jobs-using-quartznet-in-aspnet-core.html
* https://thinkrethink.net/2018/05/31/run-scheduled-background-tasks-in-asp-net-core/
* https://www.stevejgordon.co.uk/asp-net-core-2-ihostedservice
* https://medium.com/@nickfane/introduction-to-worker-services-in-net-core-3-0-4bb3fc631225
* https://blog.maartenballiauw.be/post/2017/08/01/building-a-scheduled-cache-updater-in-aspnet-core-2.html