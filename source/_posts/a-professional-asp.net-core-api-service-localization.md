---
title: A Professional ASP.NET Core API Service - Localization
date: September 20 2020
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

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Localization;

namespace Localization.Controllers
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


## JSON Localization Resources

Install below package

```bash
Install-Package My.Extensions.Localization.Json -Version 2.1.0
dotnet add package My.Extensions.Localization.Json --version 2.1.0
<PackageReference Include="My.Extensions.Localization.Json" Version="2.1.0" />
```




## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization
* https://www.codemag.com/Article/2009081/A-Deep-Dive-into-ASP.NET-Core-Localization
* https://github.com/hishamco/My.Extensions.Localization.Json