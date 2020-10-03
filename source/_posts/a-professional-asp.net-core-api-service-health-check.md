---
title: A Professional ASP.NET Core API Service - HealthCheck
date: September 22 2020
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

## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks
* https://www.codewithmukesh.com/blog/healthchecks-in-aspnet-core-explained/
