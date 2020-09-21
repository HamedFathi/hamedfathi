---
title: A Professional ASP.NET Core API Service - Versioning
date: September 21 2020
draft: true
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

## Configure API versioning

```cs
// Startup.ConfigureServices
public void ConfigureServices(IServiceCollection services)
{
    services.AddApiVersioning(config =>
    {
        // You can specify the default version as 1.0.
        config.DefaultApiVersion = new ApiVersion(1, 0);
        // Set defaul version if you dont specify it.
        config.AssumeDefaultVersionWhenUnspecified = true;
        // Let the clients of the API know all supported versions.
        // The consumers could read the 'api-supported-versions' header.
        config.ReportApiVersions = true;
        config.ApiVersionReader = new HeaderApiVersionReader("x-api-version");
    });
}
```

if you don't set these two configurations

```cs
config.DefaultApiVersion = new ApiVersion(1, 0);
config.AssumeDefaultVersionWhenUnspecified = true;
```

And don't set any version for your Controller/Action, You will get the following error when you send a request.

```json
{
    "error": {
        "code": "ApiVersionUnspecified",
        "message": "An API version is required, but was not     
specified.",
        "innerError": null
    }
}
```

## API Version Reader

API Version Reader defines how an API version is read from the HTTP request. If not explicitly configured, the default setting is that our API version will be a query string parameter named `v` (example: ../users?v=2.0). Another, probably more popular option is to store the API version in the HTTP header. We have also the possibility of having an API version both in a query string as well as in an HTTP header.

**Query string parameter**

```cs
// /api/home?v=2.0
config.ApiVersionReader = new QueryStringApiVersionReader("v");
```

**HTTP header**

```cs
// x-api-version: 2.0
config.ApiVersionReader = new HeaderApiVersionReader("x-api-version");
```

**API Version Reader Composition**

```cs
config.ApiVersionReader = ApiVersionReader.Combine(
        new QueryStringApiVersionReader("v"),
        new HeaderApiVersionReader("x-api-version")
);
```

**The URL Path**

```cs
// [Route("api/v{version:apiVersion}/[controller]")]  
config.ApiVersionReader = new UrlSegmentApiVersionReader();
```

##  Set version(s) to Controllers and Actions 

```cs
[ApiController]
// api-supported-versions: 1.1, 2.0
// api-deprecated-versions: 1.0
[ApiVersion("1.0", Deprecated = true)]
[ApiVersion("1.1")]
[ApiVersion("2.0")]
// api/v2/values/4
// api/v2.0/values/4
[Route("api/v{version:apiVersion}/[controller]")]
public class ValuesController : ControllerBase
{
    // GET api/v1/values
    [HttpGet]
    public ActionResult<IEnumerable<string>> Get()
    {
        return new string[] { "value1", "value2" };
    }

    // GET api/v2/values/5
    // GET api/v2.0/values/5
    [HttpGet("{id}")]
    [MapToApiVersion("2.0")]
    public ActionResult<string> Get(int id)
    {
        return "value";
    }

    // POST api/v1/values/5
    // POST api/v1.0/values/5
    // POST api/v1.1/values/5
    // POST api/v2/values/5
    // POST api/v2.0/values/5
    [HttpPost("{id}")]
    public ActionResult<string> Post(int id)
    {
        return "value";
    }
}
```

**Tip:** Since no version number is specified to the actions in `ValuesController`, all the endpoints are assumed to have the default version of `1.0`.


```json
{
    "error": {
        "code": "UnsupportedApiVersion",
        "message": "The HTTP resource that matches the request URI 'http://localhost:5000/api/v1.0/values/4' does not support the API version '1.0'.",
        "innerError": null
    }
}
```


## Integrate with Swagger



## Reference(s)

Most of the information in this article is from various resources.

* https://www.infoworld.com/article/3562355/how-to-use-api-versioning-in-aspnet-core.html
* https://dev.to/99darshan/restful-web-api-versioning-with-asp-net-core-1e8g
* https://dotnetcoretutorials.com/2017/01/17/api-versioning-asp-net-core/
* https://exceptionnotfound.net/overview-of-api-versioning-in-asp-net-core-3-0/
* https://dev.to/htissink/versioning-asp-net-core-apis-with-swashbuckle-making-space-potatoes-v-x-x-x-3po7