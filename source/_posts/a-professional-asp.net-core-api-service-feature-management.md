---
title: A Professional ASP.NET Core API Service - Feature Management
date: October 6 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - featuremanagement
    - feature
    - featureflag
---
 
The .NET Core `Feature Management` libraries provide idiomatic support for implementing feature flags in a .NET or ASP.NET Core application. These libraries allow you to declaratively add feature flags to your code so that you don't have to write all the `if` statements for them manually.

<!-- more -->

Install the below packages

```bash
Install-Package Microsoft.FeatureManagement -Version 2.2.0
dotnet add package Microsoft.FeatureManagement --version 2.2.0
<PackageReference Include="Microsoft.FeatureManagement" Version="2.2.0" />

Install-Package Microsoft.FeatureManagement.AspNetCore -Version 2.2.0
dotnet add package Microsoft.FeatureManagement.AspNetCore --version 2.2.0
<PackageReference Include="Microsoft.FeatureManagement.AspNetCore" Version="2.2.0" />
```

The .NET Core feature manager `IFeatureManager` gets feature flags from the framework's native configuration system. As a result, you can define your application's feature flags by using any configuration source that .NET Core supports, including the local appsettings.json file or environment variables. `IFeatureManager` relies on .NET Core dependency injection. You can register the feature management services by using standard conventions:

```cs
// Startup.ConfigureServices

using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement();
    }
}
```

By default, the feature manager retrieves feature flags from the "FeatureManagement" section of the .NET Core configuration data (`appsettings.json`). 

```json
"FeatureManagement": {
  "MoreResults": true
}
```

The following example tells the feature manager to read from a different section called "MyFeatureFlags" instead:

```cs
// Startup.ConfigureServices

using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement(options =>
        {
                options.UseConfiguration(Configuration.GetSection("MyFeatureFlags"));
        });
    }
}
```

```json
"MyFeatureFlags": {
  "MoreResults": true
}
```

## Adding simple feature flags

The `IFeatureManager` service allows you to interrogate the feature management system to identify whether a feature flag is enabled or not. `IFeatureManager` exposes a single method, for checking whether a feature flag is enabled:

```cs
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    private readonly IFeatureManager _featureManager;

    private static readonly string[] Summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };

    public WeatherForecastController(IFeatureManager featureManager /* HERE */)
    {
        _featureManager = featureManager;
    }

    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        var rng = new Random();
        IEnumerable<int> range = null;
        
        //HERE
        if (_featureManager.IsEnabledAsync("MoreResults").GetAwaiter().GetResult())
        {
            range = Enumerable.Range(1, 50);
        }
        else
        {
            range = Enumerable.Range(1, 5);
        }
        return range.Select(index => new WeatherForecast
        {
            Date = DateTime.Now.AddDays(index),
            TemperatureC = rng.Next(-20, 55),
            Summary = Summaries[rng.Next(Summaries.Length)]
        })
        .ToArray();
    }
}
```

## Avoid strings

Feature flags are identified in code using magic-strings: "MoreResults" in the previous example. Instead of scattering these around your code, the official docs recommend creating a `FeatureFlags` `enum`, and calling `nameof()` to reference the values, e.g:

```cs
// Define your flags in an enum
// Be careful not to refactor/rename any typos, as that will break configuration
public enum FeatureFlags
{
    MoreResults
}

// Reference the feature flags using nameof()
var isEnabled = await _featureManager.IsEnabledAsync(nameof(FeatureFlags.MoreResults));
```

**Static class**

Using a static class and string constants, it reduces the verbosity at the call site.

```cs
// Using a static class separates the "name" of the feature flag
// from its string value
public static class FeatureFlags
{
    public const string MoreResults = "MoreResults";
}

// No need for nameof() at the call site
var isEnabled = await _featureManager.IsEnabledAsync(FeatureFlags.MoreResults);
```

## FeatureGate

We can block access to entire controllers or action methods using the `FeatureGate` action filter.

```cs
// BetaController.cs

using Microsoft.AspNetCore.Mvc;
using Microsoft.FeatureManagement.Mvc;

// HERE
[FeatureGate("Beta")] // Beta feature flag must be enabled
public class BetaController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}
```

If you try to navigate to this page (`/Beta`) when the feature is `enabled`, you'll see the View rendered. However, if the Beta feature flag is `disabled`, you'll get a `404` when trying to view the page:

The `[FeatureGate]` attribute takes an array of feature flags, in its constructor. If any of those features are enabled, the controller is enabled.

```cs
[FeatureGate("Beta", "Alpha")]
public class BetaController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}
```

## Custom handling of missing actions





## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/azure/azure-app-configuration/use-feature-flags-dotnet-core
* https://docs.microsoft.com/en-us/azure/azure-app-configuration/quickstart-feature-flag-aspnet-core
* http://dontcodetired.com/blog/post/Using-the-Microsoft-Feature-Toggle-Library-in-ASPNET-Core-(MicrosoftFeatureManagement)
* https://andrewlock.net/introducing-the-microsoft-featuremanagement-library-adding-feature-flags-to-an-asp-net-core-app-part-1/
* https://andrewlock.net/filtering-action-methods-with-feature-flags-adding-feature-flags-to-an-asp-net-core-app-part-2/
* https://andrewlock.net/creating-dynamic-feature-flags-with-feature-filters-adding-feature-flags-to-an-asp-net-core-app-part-3/
* https://andrewlock.net/creating-a-custom-feature-filter-adding-feature-flags-to-an-asp-net-core-app-part-4/
* https://andrewlock.net/keeping-consistent-feature-flags-across-requests-adding-feature-flags-to-an-asp-net-core-app-part-5/
* https://kasunkodagoda.com/2020/01/16/implementing-feature-flags-for-asp-net-core-applications-using-microsoft-featuremanagement-library/