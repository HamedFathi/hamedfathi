---
title: A Professional ASP.NET Core API Service - Localization
date: September 27 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - api
    - localization
---

Globalization and localization are two important concepts that you should be aware of to internationalize your applications. In essence, globalization and localization are concepts that help you reach a wider audience. The former relates to building applications that support various cultures and the latter relates to how you can build your application that can support a particular locale and culture. In other words, an application takes advantage of globalization to be able to cater to different languages based on user choice. Localization is adopted by the application to adapt the content of a website to various regions or cultures.

<!-- more -->

`IStringLocalizer` and `IStringLocalizer<T>` were architected to improve productivity when developing localized apps. `IStringLocalizer` uses the `ResourceManager` and `ResourceReader` to provide culture-specific resources at run time. The interface has an indexer and an IEnumerable for returning localized strings. `IStringLocalizer` doesn't require storing the default language strings in a resource file. You can develop an app targeted for localization and not need to create resource files early in development. The code below shows how to wrap the string "About Title" for localization.

Register `AddLocalization` service.

```cs
// Startup.ConfigureServices

public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    // HERE
    services.AddLocalization();
}
```

And, Use it via `IStringLocalizer`

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Localization;

namespace WebApplicationSample.Controllers
{
    [Route("api/[controller]")]
    public class AboutController : Controller
    {
        private readonly IStringLocalizer<AboutController> _localizer;

        public AboutController(IStringLocalizer<AboutController> localizer)
        {
            _localizer = localizer;
        }

        [HttpGet]
        public string Get()
        {
            return _localizer["About Title"];
        }
    }
}
```

There are three methods used to configure localization in ASP.NET Core. These include the following:

* `AddDataAnnotationsLocalization`: This method is used to provide support for DataAnnotations validation messages.
* `AddLocalization`: This method is used to add localization services to the services container.
* `AddViewLocalization`: This method is used to provide support for localized views.

## Define the Allowed Cultures


```cs
// Startup.cs

private RequestLocalizationOptions GetLocalizationOptions()
{
    var supportedCultures = new List<CultureInfo>
    {
        new CultureInfo("en-US"),
        new CultureInfo("de-DE"),
        new CultureInfo("fr-FR"),
        new CultureInfo("en-GB")
    };
    var options = new RequestLocalizationOptions
    {
        DefaultRequestCulture = new RequestCulture("en-GB"),
        SupportedCultures = supportedCultures,
        SupportedUICultures = supportedCultures
    };
    return options;
}
```

Add above option to `UseRequestLocalization` middleware

```cs
// Startup.Configure

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    // HERE
    app.UseRequestLocalization(GetLocalizationOptions());

    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

The middleware adds three providers for the request culture by default:

* `QueryStringRequestCultureProvider`: Gets the culture from query string values
* `CookieRequestCultureProvider`: Gets the culture from a cookie
* `AcceptLanguageHeaderRequestCultureProvider`: Gets the culture from the **Accept-Language** request header

## Create Resource Files for Each Locale

There are various ways in which you can create resource files. In this example, you'll take advantage of the Visual Studio Resource Designer to create an XML-based `.resx` file.

To specify a specific rsource folder, we should change our service settings like below

```cs
// Startup.ConfigureServices

public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    // HERE
    services.AddLocalization(opt => opt.ResourcesPath = "Resources");
}
```

`ResourcesPath` property has been used to set the path to the folder where resource files (for various locales) will reside. If you don't specify any value for this property, the application will expect the resource files to be available in the application's `root directory`.

Select the project in the Solution Explorer Window and create a new folder named `Resources` in it. Resources in .NET are comprised of key/value pair of data that are compiled to a `.resources` file. A resource file is one where you can store strings, images, or object data - resources of the application.

Next, add a resources file into the newly created folder. Name the resource files as 

* `Controllers.AboutController.en-GB.resx`
* `Controllers.AboutController.en-US.resx`
* `Controllers.AboutController.de-DE.resx`
* `Controllers.AboutController.fr-FR.resx`

## Resource file naming

`Resources` are named for **the full type name of their class minus the assembly name**. For example, a French resource in a project whose main assembly is `LocalizationWebsite.Web.dll` for the class `LocalizationWebsite.Web.Startup` would be named `Startup.fr.resx`. A resource for the class `LocalizationWebsite.Web.Controllers.HomeController` would be named `Controllers.HomeController.fr.resx`. If your targeted class's namespace isn't the same as the assembly name you will need the full type name. For example, in the sample project a resource for the type `ExtraNamespace.Tools` would be named `ExtraNamespace.Tools.fr.resx`.

In the sample project, the `ConfigureServices` method sets the `ResourcesPath` to "Resources", so the project relative path for the home controller's French resource file is `Resources/Controllers.HomeController.fr.resx`. Alternatively, you can use folders to organize resource files. For the about controller, the path would be `Resources/Controllers/AboutController.fr.resx`. If you don't use the `ResourcesPath` option, the `.resx` file would go in the project base directory. The resource file for `AboutController` would be named `Controllers.HomeController.fr.resx`. The choice of using the `dot` or `path` naming convention depends on how you want to organize your resource files.

In our sample, `WebApplicationSample` is the assembly name so we should create our resources inside `Resources` folder with this way.

**[NamespaceWithoutAssemblyName].[ControllerName].[Culture].resx**
Or
**[NamespaceWithoutAssemblyName]/[ControllerName].[Culture].resx**

## How to use resource files?

Write below key-values:

`Controllers.AboutController.en-GB.resx`

| Key             | Value     |
|-----------------|-----------|
| GreetingMessage | Hello {0} |
| SayHello        | Hello     |

`Controllers.AboutController.de-DE.resx`

| Key             | Value     |
|-----------------|-----------|
| GreetingMessage | Hallo {0} |
| SayHello        | Hallo     |

Now, You can use them via controllers

```cs
namespace WebApplicationSample.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class AboutController : ControllerBase
    {
        private readonly IStringLocalizer<AboutController> _localizer;

        public AboutController(IStringLocalizer<AboutController> localizer)
        {
            _localizer = localizer;
        }

        // http://localhost:PORT/weatherforecast
        // http://localhost:PORT/weatherforecast?culture=en-GB
        // http://localhost:PORT/weatherforecast?culture=de-DE
        [HttpGet]
        public string Get()
        {
            return _localizer["SayHello"];

        }

        // http://localhost:PORT/weatherforecast/hamed
        // http://localhost:PORT/weatherforecast/hamed?culture=en-GB
        // http://localhost:PORT/weatherforecast/hamed?culture=de-DE
        [HttpGet("{name}")]
        public string Get(string name)
        {

            return _localizer[string.Format(_localizer["GreetingMessage"], name)];

        }
    }
}
```

`SayHello` returns a simple text based on your culture.
`GreetingMessage` returns a text but accept variable too. You can use unlimited place holders `({0} {1} {2} {3} , ...)` and pass your variables via `string.Format()`.

If `IStringLocalizer` does not find any value for the key, It will return `the key` itself as a result.

## JSON Localization Resources

You may want to use `.json` files as a resource instead of `.resx` files, so

Install below package 

```bash
Install-Package My.Extensions.Localization.Json -Version 2.1.0
dotnet add package My.Extensions.Localization.Json --version 2.1.0
<PackageReference Include="My.Extensions.Localization.Json" Version="2.1.0" />
```

Remove `services.AddLocalization();` and replace it with `services.AddJsonLocalization()`:

```cs
// Startup.ConfigureServices

using System.IO;

public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    // REMOVE THIS
    // services.AddLocalization(opt => opt.ResourcesPath = "Resources");

    // HERE
    var directory = Path.Combine(Directory.GetCurrentDirectory(), "Resources");
    services.AddJsonLocalization(opt => opt.ResourcesPath = directory);
}
```

Write below `JSON` files:

`Controllers.AboutController.en-GB.json`

```json
{
  "GreetingMessage": "Hello {0}",
  "SayHello": "Hello"
}
```

`Controllers.AboutController.de-DE.json`

```json
{
  "GreetingMessage": "Hallo {0}",
  "SayHello": "Hallo"
}
```

Now, You can use them via controllers

```cs

namespace WebApplicationSample.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class AboutController : ControllerBase
    {
        private readonly IStringLocalizer<AboutController> _localizer;

        public AboutController(IStringLocalizer<AboutController> localizer)
        {
            _localizer = localizer;
        }

        // http://localhost:PORT/weatherforecast
        // http://localhost:PORT/weatherforecast?culture=en-GB
        // http://localhost:PORT/weatherforecast?culture=de-DE
        [HttpGet]
        public string Get()
        {
            return _localizer["SayHello"];

        }

        // http://localhost:PORT/weatherforecast/hamed
        // http://localhost:PORT/weatherforecast/hamed?culture=en-GB
        // http://localhost:PORT/weatherforecast/hamed?culture=de-DE
        [HttpGet("{name}")]
        public string Get(string name)
        {

            return _localizer[string.Format(_localizer["GreetingMessage"], name)];

        }
    }
}
```

## DataAnnotation & Localization


## FluentValidation & Localization


## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization
* https://www.codemag.com/Article/2009081/A-Deep-Dive-into-ASP.NET-Core-Localization
* https://github.com/hishamco/My.Extensions.Localization.Json
* https://joonasw.net/view/aspnet-core-localization-deep-dive