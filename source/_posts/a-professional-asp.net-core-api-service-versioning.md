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

## Configure API versioning

```cs
// Startup.ConfigureServices
using Microsoft.AspNetCore.Mvc.Versioning;

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

**Media type**

```cs
// Content-Type: text/plain; charset=utf-8; v=2
// Content-Type: application/json; charset=utf-8; v=2
config.ApiVersionReader = new MediaTypeApiVersionReader();

// Content-Type: text/plain; charset=utf-8; version=2
// Content-Type: application/json; charset=utf-8; version=2
config.ApiVersionReader = new MediaTypeApiVersionReader("version"));
```

**API Version Reader Composition**

```cs
config.ApiVersionReader = ApiVersionReader.Combine(
        new QueryStringApiVersionReader("v"),
        new HeaderApiVersionReader("x-api-version"),
        new MediaTypeApiVersionReader("version")
);
```

**The URL Path**

```cs
// [Route("api/v{version:apiVersion}/[controller]")]  
config.ApiVersionReader = new UrlSegmentApiVersionReader();
```

##  Set version(s) to Controllers and Actions 

Consider the following example:

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

**[ApiVersion("1.0", Deprecated = true)]**

Annotating our controller with, for example, [ApiVersion("1.0")] attribute, means that this controller supports API version 1.0.

To deprecate some version in our API controller, we need to set Deprecated flag to true: [ApiVersion("1.0", Deprecated = true)].

In such cases a `api-deprecated-versions` header will be added to identify deprecated versions.

**[ApiVersion("1.1")]**

**[ApiVersion("2.0")]**

Controllers can support multiple API versions.

**[Route("api/v{version:apiVersion}/[controller]")]**

To implement URL path versioning, You can modify the Route attribute of the controllers to accept API versioning info in the path param.

**[MapToApiVersion("2.0")]**

From the several versions introduced for the controller, you can specify a specific version (eg. version 2.0) for the action. The action is **only** available with this version.

If you try to access it with other versions, you will encounter an `UnsupportedApiVersion` error.

```json
{
    "error": {
        "code": "UnsupportedApiVersion",
        "message": "The HTTP resource that matches the request URI 'http://localhost:5000/api/v1.0/values/4' does not support the API version '1.0'.",
        "innerError": null
    }
}
```

**[ApiVersionNeutral]**

If we have a service that is version-neutral, we will mark that controller with `[ApiVersionNeutral]` attribute.


**How to get the API version inside a controller?**

```cs
var apiVersion = HttpContext.GetRequestedApiVersion();
```

And also

```cs
public IActionResult Get( int id, ApiVersion apiVersion ) {}
```

## API Version Conventions

Besides attributing our controllers and methods, another way to configure service versioning is with API versioning conventions. There are several reasons why would we use API versioning conventions instead of attributes:

* Centralized management and application of all service API versions
* Apply API versions to services defined by controllers in external .NET assemblies
* Dynamically apply API versions from external sources; for example, from the configuration

```cs
services.AddApiVersioning(config =>
{
    config.Conventions.Controller<MyController>().HasApiVersion(1, 0);
});
```

Here's an example of setting deprecated API version as well as versioning controller actions:

```cs
services.AddApiVersioning(config =>
{
    config.Conventions.Controller<MyController>()	   
                       .HasDeprecatedApiVersion(1, 0)
                       .HasApiVersion(1, 1)
                       .HasApiVersion(2, 0)
                       .Action(c => c.Get1_0()).MapToApiVersion(1, 0)
                       .Action(c => c.Get1_1()).MapToApiVersion(1, 1)
                       .Action(c => c.Get2_0()).MapToApiVersion(2, 0);
});
```

There is also an option to define custom conventions.There is a `IControllerConvention` interface for this purpose. Custom conventions are added to the convention builder through the API versioning options:

```cs
services.AddApiVersioning(config =>
{
    config.Conventions.Add(new MyCustomConvention());
});
```

## How to prevent an unavailable or deprecated api version from loading?

Consider the following example

```cs
[ApiController]
[ApiVersion("1.1")]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/[controller]")]
public class ValuesController : ControllerBase
{
    // GET api/v1/values
    [HttpGet]
    [MapToApiVersion("1.0")]
    public ActionResult<IEnumerable<string>> Get()
    {
        return new string[] { "value1", "value2" };
    }

    [HttpGet("{id}")]
    [MapToApiVersion("2.0")]
    public ActionResult<string> Get(int id)
    {
        return "value";
    }
}
```

In this example, we have two valid versions (`1.1`, `2.0`) for all actions but the `Get()` has mapped to version `1.0` which is invalid for us. We want to prevent this method from loading.

To do this, you should write an `ActionFilterAttribute` as following

```cs
// PreventUnavailableApiVersionsAttribute.cs

[AttributeUsage(AttributeTargets.Class)]
public class PreventUnavailableApiVersionsAttribute : ActionFilterAttribute
{
    public string Header { get; set; } = "x-api-version";
    public string QueryString { get; set; } = "v";
    public string UrlSegment { get; set; } = "version";
    public bool IsADeprecatedVersionValid { get; set; } = true;
    private string GetUrl(HttpRequest request)
    {
        return $"{request.Scheme}://{request.Host}{request.Path}{request.QueryString}";
    }
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        var props = context.ActionDescriptor.Properties;
        var url = GetUrl(context.HttpContext.Request);
        var headerVersion = context.HttpContext.Request.Headers.Count(x => string.Equals(x.Key, Header.Trim(), StringComparison.InvariantCultureIgnoreCase));
        var routeVersion = context.RouteData.Values[UrlSegment.Trim()];
        var queryVersion = context.HttpContext.Request.QueryString.Value.Trim();
        var matchedQuery = queryVersion.Replace("?", "").Split('&').FirstOrDefault(x => x.StartsWith($"{QueryString}="));
        var isSkippable = routeVersion == null && headerVersion == 0 && string.IsNullOrEmpty(matchedQuery);
        if (isSkippable) return;
        var version = "";
        if (routeVersion != null)
        {
            version = routeVersion.ToString();
        }
        if (headerVersion > 0)
        {
            version = context.HttpContext.Request.Headers["x-api-version"].ToString();
        }
        if (!string.IsNullOrEmpty(matchedQuery))
        {
            version = matchedQuery.Replace($"{QueryString}=", "");
        }
        version = version.Contains('.') ? version : $"{version}.0";
        foreach (var prop in props)
        {
            var apiVersionModel = prop.Value as ApiVersionModel;
            if (apiVersionModel != null)
            {
                if (apiVersionModel.IsApiVersionNeutral) return;
                var deprecated = apiVersionModel.DeprecatedApiVersions.Select(x => x.ToString());
                var impl = apiVersionModel.ImplementedApiVersions.Select(x => x.ToString());
                var supported = apiVersionModel.SupportedApiVersions.Select(x => x.ToString());
                var isSupported = IsADeprecatedVersionValid ? impl.Contains(version) : supported.Contains(version);
                if (!isSupported)
                {
                    context.Result = new JsonResult(new ApiVersionUnavailable
                    {
                        Code = "UnavailableApiVersion",
                        AvailableVersions = supported,
                        Message = $"The HTTP resource that matches the request URI '{url}' does not available by the API version '{version}'."
                    });
                    context.HttpContext.Response.StatusCode = (int)HttpStatusCode.BadRequest;
                }
            }
        }
    }
}
```

And consume it on top of your controller

```cs
[PreventUnavailableApiVersions]
[ApiController]
[ApiVersion("1.1")]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/[controller]")]
public class ValuesController : ControllerBase
{
    // GET api/v1/values
    [HttpGet]
    [MapToApiVersion("1.0")]
    public ActionResult<IEnumerable<string>> Get()
    {
        return new string[] { "value1", "value2" };
    }

    [HttpGet("{id}")]
    [MapToApiVersion("2.0")]
    public ActionResult<string> Get(int id)
    {
        return "value";
    }
}
```

Now, whenever you call the `Get()` action, the `PreventUnavailableApiVersions` checks the version you requested (`1.0`) with versions  of your controller (`1.1`, `2.0`), If the requested version is not in the list, it returns the same error as below.

```json
// Status code: 400 Bad Request
{
    "code": "UnavailableApiVersion",
    "message": "The HTTP resource that matches the request URI 'http://localhost:5000/api/v1.0/values' does not available by the API version '1.0'.",
    "availableVersions": [
        "1.1",
        "2.0"
    ]
}
```

**Deprecated versions**

As a default behaviour deprecated versions are in valid list it means if you write like the below code

```cs
[PreventUnavailableApiVersions]
[ApiController]
[ApiVersion("1.0", Deprecated = true)]
[ApiVersion("1.1")]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/[controller]")]
public class ValuesController : ControllerBase
{
    // GET api/v1/values
    [HttpGet]
    [MapToApiVersion("1.0")]
    public ActionResult<IEnumerable<string>> Get()
    {
        return new string[] { "value1", "value2" };
    }

    [HttpGet("{id}")]
    [MapToApiVersion("2.0")]
    public ActionResult<string> Get(int id)
    {
        return "value";
    }
}
```

You will see the result of action method without any unavailable error but you can config to remove deprecated versions from list of valid versions.
You just need to write

```cs
// IsADeprecatedVersionValid: default is true
[PreventUnavailableApiVersions(IsADeprecatedVersionValid = false)]
```

Now, the action filter consider the version `1.0` as an unavailable and you will get an `UnavailableApiVersion` response.

**Configuration**

`PreventUnavailableApiVersions` action filter supports `header`, `query string` and `url segment` mode so you can configure any of these just like what you did in `ApiVersionReader`.

```cs
// QueryStringApiVersionReader("ver")
[PreventUnavailableApiVersions(QueryString = "ver")] // Default is 'v'

// HeaderApiVersionReader("api-version-header")
[PreventUnavailableApiVersions(Header = "ver")] // Default is 'x-api-version'

// UrlSegmentApiVersionReader
// [Route("api/v{myversion:apiVersion}/[controller]")]
[PreventUnavailableApiVersions(UrlSegment = "myversion")] // Default is 'version',  [Route("api/v{version:apiVersion}/[controller]")]
```

## Reference(s)

Most of the information in this article is from various resources.

* https://www.infoworld.com/article/3562355/how-to-use-api-versioning-in-aspnet-core.html
* https://dev.to/99darshan/restful-web-api-versioning-with-asp-net-core-1e8g
* https://dotnetcoretutorials.com/2017/01/17/api-versioning-asp-net-core/
* https://exceptionnotfound.net/overview-of-api-versioning-in-asp-net-core-3-0/
* https://github.com/microsoft/aspnet-api-versioning/wiki/API-Version-Reader
* 