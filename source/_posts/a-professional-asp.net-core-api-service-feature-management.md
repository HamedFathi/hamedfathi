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
        
        // HERE
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

The `[FeatureGate]` attribute takes an array of feature flags, in its constructor. If `any` of those features are enabled, the controller is enabled.

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

If an action is removed due to a feature being disabled, the default is to generate a 404 response. That may be fine for some applications, especially if you're using error handling middleware to customise error responses to avoid ugly "raw" 404.

However, it's also possible that you may want to generate a different response in this situation. Maybe you want to redirect users to a "stable" page, return a "join the waiting list" view, or simply return a different response, like a 403 Forbidden.

You can achieve any of these approaches by creating a service that implements the `IDisabledFeaturesHandler` interface. Implementers are invoked as part of the action filter pipeline, when an action method is "removed" due to a feature being disabled. In the example below, I show how to generate a 403 Forbidden response, but you have access to the whole `ActionExecutingContext` in the method, so you can do anything you can in a standard action filter:

```cs
using Microsoft.FeatureManagement.Mvc;

public class RedirectDisabledFeatureHandler : IDisabledFeaturesHandler
{
    public Task HandleDisabledFeatures(IEnumerable<string> features, ActionExecutingContext context)
    {
        context.Result = new ForbidResult(); // generate a 403
        return Task.CompletedTask;
    }
}
```

To register the handler, update your call to `AddFeatureManagement()`:

```cs
// Startup.ConfigureServices

using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement()
            .UseDisabledFeaturesHandler(new RedirectDisabledFeatureHandler()); /* HERE */
    }
}
```

With the handler registered, if you now try to access a disabled feature, a 403 response is generated, which is intercepted by the error handling middleware, and you're redirected to the "Access Denied" page for the app:

## Razor

You can use feature flags inside your Views.

**Dependency injection**

 You can `inject` the `IFeatureManager` service into views using dependency injection. You could use the `@inject` directive, and check for the feature manually:
 
 ```xml
 <!-- Inject the service using DI  -->
@inject  Microsoft.FeatureManagement.IFeatureManager _featureManager; 

<nav>
    <ul>
        <!-- Check if the feature is enabled  -->
        @if (await _featureManager.IsEnabledAsync(FeatureFlags.PromotionDiscounts))
        {
            <li class="nav-item">
                <a class="nav-link text-dark" asp-controller="PromotionDiscount" asp-action="Index">Discount</a>
            </li>
        }
    </ul>
</nav>
```

**Using Tag Helper**

If you have any UI elements you want to hide under feature flags you can use the tag helper provided in `Microsoft.FeatureManagement.AspNetCore` library to do that. 

First, you need to add the tag helper to the `_ViewImports.cshtml` so your views can access it.

```cs
@addTagHelper *, Microsoft.FeatureManagement.AspNetCore
```

Next, you can use `<feature>` tag helper to wrap the UI elements you want to put behind a feature flag.

```xml
@model HomeViewModel
@{
    ViewData["Title"] = "Home Page";
}

<div class="text-center">
    <feature name="@Features.PromotionDiscounts" negate="true"><h1 class="display-4">Enjoy the latest music from your favorite artist</h1></feature>
    <feature name="@Features.PromotionDiscounts"><h1 class="display-4">Enjoy 25% off for selected albums from your favorite artist</h1></feature>
    
    <feature name="@Features.UserSuggestions">
        <partial name="_UserSuggestionsPartial" model="@Model.Suggestions" />
    </feature>
</div>
```

We can use `negate` attribute and set it to true if you want to show the content between feature tag helper when the feature is disabled.

## Dynamic Features

We introduce feature filters, which are a much more powerful way of working with feature flags. These let you enable a feature based on arbitrary data. For example, you could enable a feature based on headers in an incoming request, based on the current time, or based on the current user's claims.

```json
{
  "FeatureManagement": {
    "Beta": false
  }
}
```

With this configuration, the Beta feature flag is always false for all users (until configuration changes). While this will be useful in some cases, you may often want to enable features for only some of your users, or only some of the time.

`Microsoft.FeatureManagement` introduces an interface `IFeatureFilter` which can be used to decide whether a feature is enabled or not based on any logic you require.

**Enabling a feature flag based on the current time with TimeWindowFilter**

The `TimeWindowFilter` does as its name suggests - it enables a feature for a given time window. You provide the start and ending DateTime, and any calls to `IFeatureManager.IsEnabledAsync()` for the feature will be true only between those times.

Add the feature management services in `Startup.ConfigureServices`, by calling `AddFeatureManagement()`, which returns an IFeatureManagementBuilder. You can enable the time window filter by calling `AddFeatureFilter<>()` on the builder:

```cs
// Startup.ConfigureServices

using Microsoft.FeatureManagement;
using Microsoft.FeatureManagement.FeatureFilters;

public class Startup 
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement()
            .AddFeatureFilter<TimeWindowFilter>(); // HERE
    }
}
```

This adds the `IFeatureFilter` to your app, but you need to configure it using the configuration system. Each `IFeatureFilter` can have an associated "settings" object, depending on the implementation. For the `TimeWindowFilter`, this looks like:

```cs
// TimeWindowSettings.cs

So let's consider a scenario: I want to enable a custom Christmas banner which goes live on boxing day at 2am UTC, and ends three days later at 1am UTC.


public class TimeWindowSettings
{
    public DateTimeOffset? Start { get; set; }
    public DateTimeOffset? End { get; set; }
}
```

So let's consider a scenario: I want to enable a custom Christmas banner which goes live on boxing day at 2am UTC, and ends three days later at 1am UTC.

We'll start by creating a feature flag for it in code called `ChristmasBanner`

```cs
public static class FeatureFlags
{
    public const string ChristmasBanner = "ChristmasBanner";
}
```

Now we'll add the configuration. As before, we nest the configuration under the `FeatureManagement` key and provide the name of the feature. However, instead of using a Boolean for the feature, we use `EnabledFor`, and specify an array of feature filters.

```json
"FeatureManagement": {
  "ChristmasBanner": {
    "EnabledFor": [
      {
        "Name": "Microsoft.TimeWindow",
        "Parameters": {
          "Start": "26 Dec 2019 02:00:00 +00:00",
          "End": "29 Dec 2019 01:00:00 +00:00"
        }
      }
    ]
  }
}
```

It's important you get the configuration correct here. The general pattern is identical for all feature filters:

* The feature name ("ChristmasBanner") should be the key of an object:
* This object should contains a single property, `EnabledFor`, which is an array of objects.
* Each of the objects in the array represents an `IFeatureFilter`. For each filter
    * Provide the `Name` of the filter ("Microsoft.TimeWindow" for the `TimeWindowFilter`)
    * Optionally provide a `Parameters` object, which is bound to the settings object of the feature filter (`TimeWindowSettings` in this case).
* If any of the feature filters in the array are satisfied for a given request, the feature is enabled. It is only disabled if all `IFeatureFilters` indicate it should be disabled.

With this configuration, the `ChristmasBanner` feature flag will return false until `DateTime.UtcNow` falls between the provided dates:

```cs
// WeatherForecastController.cs

using Microsoft.FeatureManagement;

[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    private readonly IFeatureManager _featureManager;
    public WeatherForecastController(IFeatureManager featureManager)
    {
        _featureManager = featureManager;
        // only returns true during provided time window
        var showBanner = _featureManager.IsEnabled(FeatureFlags.ChristmasBanner);
    }
}
```

The real benefit to using `IFeatureFilters` is that you get dynamic behaviour, but you can still control it from configuration.

**Note**: that TimeWindowSettings has nullable values for Start and End, to give you open-ended time windows e.g. always enable until a given date, or only enable from a given date.

**Rolling features out slowly with PercentageFilter**

The `PercentageFilter` also behaves as you might expect - it only enables a feature for x percent of requests, where x is controlled via settings. Enabling the `PercentageFilter` follows the same procedure as for `TimeWindowFilter`.

```cs
// Startup.ConfigureServices

using Microsoft.FeatureManagement;
using Microsoft.FeatureManagement.FeatureFilters;

public class Startup 
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement()
            .AddFeatureFilter<PercentageFilter>(); // HERE
    }
}
```

Create a feature flag:

```cs
public static class FeatureFlags
{
    public const string FancyFonts = "FancyFonts";
}
```

Configure the feature in configuration:

```json
"FeatureManagement": {
  "FancyFonts": {
    "EnabledFor": [
      {
        "Name": "Microsoft.Percentage",
        "Parameters": {
          "Value": 10
        }
      }
    ]
  }
}
```

The `PercentageSettings` object consists of a single int, which is the percentage of the time the flag should be enabled. In the example above, the flag will be enabled for 10% of calls to `IFeatureManager.IsEnabled(FeatureFlags.FancyFonts)``.

## Creating a custom IFeatureFilter



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