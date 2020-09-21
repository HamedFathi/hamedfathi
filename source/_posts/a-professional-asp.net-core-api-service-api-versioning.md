---
title: A Professional ASP.NET Core API Service - Versioning
date: September 21 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - api
    - versioning
---

When developing APIs, you should keep one thing in mind: **Change is inevitable**. When your API has reached a point where you need to add more responsibilities, you should consider versioning your API. Hence you will need a versioning strategy.

<!-- more -->

Install the below package

```bash
Install-Package Microsoft.AspNetCore.Mvc.Versioning -Version 4.1.1
dotnet add package Microsoft.AspNetCore.Mvc.Versioning --version 4.1.1
<PackageReference Include="Microsoft.AspNetCore.Mvc.Versioning" Version="4.1.1" />
```

Configure API versioning:

```cs
// Startup.ConfigureServices
public void ConfigureServices(IServiceCollection services)
{
    services.AddApiVersioning(config =>
    {
        config.DefaultApiVersion = new ApiVersion(1, 0);
        config.AssumeDefaultVersionWhenUnspecified = true;
        config.ReportApiVersions = true;
        config.ApiVersionReader = new HeaderApiVersionReader(ApiVersionHeader);

    });
}
```

## Reference(s)

Most of the information in this article is from various resources.

* https://www.infoworld.com/article/3562355/how-to-use-api-versioning-in-aspnet-core.html