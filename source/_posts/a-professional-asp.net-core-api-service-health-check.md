---
title: A Professional ASP.NET Core API Service - HealthCheck
date: October 3 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - healthy
    - unhealthy
    - health
    - healthcheck
---
 
ASP.NET Core offers Health Checks Middleware and libraries for reporting the health of app infrastructure components.

Health checks are exposed by an app as HTTP endpoints. Health check endpoints can be configured for a variety of real-time monitoring scenarios:

* Health probes can be used by container orchestrators and load balancers to check an app's status. For example, a container orchestrator may respond to a failing health check by halting a rolling deployment or restarting a container. A load balancer might react to an unhealthy app by routing traffic away from the failing instance to a healthy instance.
* Use of memory, disk, and other physical server resources can be monitored for healthy status.
* Health checks can test an app's dependencies, such as databases and external service endpoints, to confirm availability and normal functioning.

<!-- more -->

Install the below package

```bash
Install-Package Microsoft.AspNetCore.Diagnostics.HealthChecks -Version 2.2.0
dotnet add package Microsoft.AspNetCore.Diagnostics.HealthChecks --version 2.2.0
<PackageReference Include="Microsoft.AspNetCore.Diagnostics.HealthChecks" Version="2.2.0" />
```

To start working with HealthCheck system you should add configs as following

```cs
// Startup.cs

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        
        // HERE
        services.AddHealthChecks();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();
        app.UseAuthorization();
        
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
            
            // HERE
            endpoints.MapHealthChecks("/health");
        });
    }
}
```

Now, You are able to browse `http://localhost:PORT/health` to see `Healthy`!

## Create health checks

Health checks are created by implementing the `IHealthCheck` interface. The `CheckHealthAsync` method returns a `HealthCheckResult` that indicates the health as `Healthy`, `Degraded`, or `Unhealthy`. The result is written as a plaintext response with a configurable status code. `HealthCheckResult` can also return optional key-value pairs.

The following `ExampleHealthCheck` class demonstrates the layout of a health check. The health checks logic is placed in the `CheckHealthAsync` method. The following example sets a dummy variable, healthCheckResultHealthy, to `true`. If the value of healthCheckResultHealthy is set to `false`, the `HealthCheckResult.Unhealthy` status is returned.

```cs
// ExampleHealthCheck.cs

using System.Threading;
using Microsoft.Extensions.Diagnostics.HealthChecks;

public class ExampleHealthCheck : IHealthCheck
{
    public Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        var healthCheckResultHealthy = true;

        if (healthCheckResultHealthy)
        {
            return Task.FromResult(
                HealthCheckResult.Healthy("A healthy result."));
        }

        return Task.FromResult(
            HealthCheckResult.Unhealthy("An unhealthy result."));
    }
}
```

## Register health check services

The `ExampleHealthCheck` type is added to health check services with AddCheck in `Startup.ConfigureServices`:

```cs
// Startup.ConfigureServices.cs

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        
        // HERE
        services.AddHealthChecks()
            .AddCheck<ExampleHealthCheck>("example_health_check") /* HERE */
        ;
    }
}
```

**Tags** 

They can be used to filter health checks.

```cs
// Startup.ConfigureServices.cs

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        
        services.AddHealthChecks()
            .AddCheck<ExampleHealthCheck>(
                "example_health_check",
                failureStatus: HealthStatus.Degraded,
                tags: new[] { "example" });
    }
}
```

**HealthStatus**

Represents the reported status of a health check result.

| Status | Description |
|--------|-------------|
|Degraded|It could be used for checks that did succeed but are slow or unstable. For example, A simple database query did succeed but took more than a second. Moving traffic to another instance is probably a good idea until the problem has resolved.|
|Healthy|Indicates that the health check determined that the component was healthy.|
|Unhealthy|It means that the component does not work at all. For example, A connection to the Redis cache could no be established. Restarting the instance could solve this issue.|

## Customize Health check result

In order to generate a more readable response that makes sense, let's add a bunch of reponse classes.

* Response Class for Overall Health
* Response Class for Component-wise Health

```cs
// HealthCheckResponse.cs
public class HealthCheckResponse
{
    public string Status { get; set; }
    public string Component { get; set; }
    public string Description { get; set; }
    public IEnumerable<string> Tags { get; set; }
    public string Exception { get; set; }
}

// HealthCheckResult.cs
// The aggregation of all health check responses, even if one is unhealthy, 'Status' will be unhealthy.
public class HealthCheckResult
{
    public string Status { get; set; }
    public IEnumerable<HealthCheckResponse> HealthChecks { get; set; }
    public string HealthCheckDuration { get; set; }
}
```

`ResponseWriter` is responsible for how the response is displayed.

```cs
// Startup.Configure

using System.Text.Json;
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using Microsoft.Extensions.Diagnostics.HealthChecks;
using Microsoft.AspNetCore.Http;
using System.Linq;

public class Startup
{
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();
        app.UseAuthorization();
        
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
            
            // HERE
            endpoints.MapHealthChecks("/health", new HealthCheckOptions
            {
                ResponseWriter = async (context, report) =>
                {
                    context.Response.ContentType = "application/json";
                    var response = new HealthCheckResult
                    {
                        Status = report.Status.ToString(),
                        HealthChecks = report.Entries.Select(x => new HealthCheckResponse
                        {
                            Component = x.Key,
                            Status = x.Value.Status.ToString(),
                            Description = x.Value.Description,
                            Exception = x.Value.Exception?.Message,
                            Tags = x.Value.Tags
                        }),
                        HealthCheckDuration = report.TotalDuration.ToString()
                    };
                    await context.Response.WriteAsync(JsonSerializer.Serialize(response));
                }
            });
        });
    }
}
```

Based on above sample the result will be:

```json
{
   "Status":"Healthy",
   "HealthChecks":[
      {
         "Status":"Healthy",
         "Component":"example_health_check",
         "Description":"A healthy result.",
         "Tags":[
            "example"
         ],
         "Exception":null
      }
   ],
   "HealthCheckDuration":"00:00:00.0010732"
}
```

## Enterprise solution

[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) is an enterprise HealthChecks for ASP.NET Core Diagnostics Package.

**HealthChecks**


**HealthChecks UI**


## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks
* https://www.codewithmukesh.com/blog/healthchecks-in-aspnet-core-explained/
* https://blog.zhaytam.com/2020/04/30/health-checks-aspnetcore/
* https://volosoft.com/blog/Using-Health-Checks-in-ASP.NET-Boilerplate