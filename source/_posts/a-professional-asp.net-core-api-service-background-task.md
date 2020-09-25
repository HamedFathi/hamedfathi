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

In `ASP.NET Core`, background tasks can be implemented as `hosted services`. A hosted service is a class with background task logic that implements the `IHostedService` interface. This topic provides three hosted service examples:

* Background task that runs on a timer.
* Hosted service that activates a scoped service. The scoped service can use dependency injection (DI).
* Queued background tasks that run sequentially.

<!-- more -->

## Background task that runs on a timer

A timed background task makes use of the `System.Threading.Timer` class. The timer triggers the task's `DoWork` method. The timer is disabled on `StopAsync` and disposed when the service container is disposed on `Dispose`:



```cs
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using System;
using System.Threading;
using System.Threading.Tasks;

namespace TaskBg.Controllers
{
    public class TimedHostedService : IHostedService, IDisposable
    {
        private int executionCount = 0;
        private readonly ILogger<TimedHostedService> _logger;
        private Timer _timer;

        public TimedHostedService(ILogger<TimedHostedService> logger)
        {
            _logger = logger;
        }

        public Task StartAsync(CancellationToken stoppingToken)
        {
            _logger.LogInformation("Timed Hosted Service running.");

            _timer = new Timer(DoWork, null, TimeSpan.Zero,
                TimeSpan.FromSeconds(5));

            return Task.CompletedTask;
        }

        private void DoWork(object state)
        {
            var count = Interlocked.Increment(ref executionCount);

            _logger.LogInformation(
                "Timed Hosted Service is working. Count: {Count}", count);
        }

        public Task StopAsync(CancellationToken stoppingToken)
        {
            _logger.LogInformation("Timed Hosted Service is stopping.");

            _timer?.Change(Timeout.Infinite, 0);

            return Task.CompletedTask;
        }

        public void Dispose()
        {
            _timer?.Dispose();
        }
    }
}
```

The `Timer` doesn't wait for previous executions of `DoWork` to finish, so the approach shown might not be suitable for every scenario. `Interlocked.Increment` is used to increment the execution counter as **an atomic operation**, which ensures that multiple threads **don't** update `executionCount` concurrently.

The service is registered in:

```cs
// Startup.ConfigureServices

public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    
    // HERE
    services.AddHostedService<TimedHostedService>();
}

// OR
// Program.cs

public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            })
            .ConfigureServices(services =>
            {
                // HERE
                services.AddHostedService<VideosWatcher>();
            });
}
```

## Consuming a scoped service in a background task


















You should implement `IScheduledTask` if you want to have the ability to config the schedule for the task to run, for example 1 a days, every hours, etc. using cron expression.

Implement `BackgroundService` if you want to have a task always run continuously.

## Quartz scheduler 

Install the below packages

```bash
Install-Package Quartz -Version 3.1.0
dotnet add package Quartz --version 3.1.0
<PackageReference Include="Quartz" Version="3.1.0" />

Install-Package Quartz.AspNetCore -Version 3.1.0
dotnet add package Quartz.AspNetCore --version 3.1.0
<PackageReference Include="Quartz.AspNetCore" Version="3.1.0" />
```

```cs
// ExampleJob.cs

public class ExampleJob : IJob
{
    public async Task Execute(IJobExecutionContext context)
    {
        await Console.Out.WriteLineAsync("Greetings from HelloJob!").ConfigureAwait(false);
    }
}

// Startup.ConfigureServices

public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    // base configuration for DI
    services.AddQuartz(q =>
    {
        // handy when part of cluster or you want to otherwise identify multiple schedulers
        q.SchedulerId = "Scheduler-Core";

        // we take this from appsettings.json, just show it's possible
        // q.SchedulerName = "Quartz ASP.NET Core Sample Scheduler";

        // we could leave DI configuration intact and then jobs need to have public no-arg constructor
        // the MS DI is expected to produce transient job instances 
        q.UseMicrosoftDependencyInjectionJobFactory(options =>
        {
            // if we don't have the job in DI, allow fallback to configure via default constructor
            options.AllowDefaultConstructor = true;
        });

        // or 
        // q.UseMicrosoftDependencyInjectionScopedJobFactory();

        // these are the defaults
        q.UseSimpleTypeLoader();
        q.UseInMemoryStore();
        q.UseDefaultThreadPool(tp =>
        {
            tp.MaxConcurrency = 10;
        });

        // configure jobs with code
        var jobKey = new JobKey("awesome job", "awesome group");
        q.AddJob<ExampleJob>(j => j
            .StoreDurably()
            .WithIdentity(jobKey)
            .WithDescription("my awesome job")
        );

        q.AddTrigger(t => t
            .WithIdentity("Simple Trigger")
            .ForJob(jobKey)
            .StartNow()
            .WithSimpleSchedule(x => x.WithInterval(TimeSpan.FromSeconds(1)).RepeatForever())
            .WithDescription("my awesome simple trigger")
        );

    });

    services.AddQuartzServer(options =>
    {
        // when shutting down we want jobs to complete gracefully
        options.WaitForJobsToComplete = true;
    });
}
```


## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services
* https://andrewlock.net/creating-a-quartz-net-hosted-service-with-asp-net-core/
* https://www.infoworld.com/article/3529418/how-to-schedule-jobs-using-quartznet-in-aspnet-core.html
* https://thinkrethink.net/2018/05/31/run-scheduled-background-tasks-in-asp-net-core/
* https://www.stevejgordon.co.uk/asp-net-core-2-ihostedservice
* https://medium.com/@nickfane/introduction-to-worker-services-in-net-core-3-0-4bb3fc631225
* https://blog.maartenballiauw.be/post/2017/08/01/building-a-scheduled-cache-updater-in-aspnet-core-2.html
* https://www.quartz-scheduler.net/
* https://github.com/quartznet/quartznet/tree/master/src/Quartz.Examples.AspNetCore