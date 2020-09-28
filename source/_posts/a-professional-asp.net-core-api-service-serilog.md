---
title: A Professional ASP.NET Core API Service - Serilog
date: September 28 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - log
---

`Serilog` is an alternative logging implementation that plugs into ASP.NET Core. It supports the same structured logging APIs, and receives log events from the ASP.NET Core framework class libraries, but adds a stack of features that make it a more appealing choice for some kinds of apps and environments.

<!-- more -->

Install the below packages

```bash
Install-Package Serilog.AspNetCore -Version 3.4.0
dotnet add package Serilog.AspNetCore --version 3.4.0
<PackageReference Include="Serilog.AspNetCore" Version="3.4.0" />
```

## Serilog Enrichers

To enable Structured Logging and to unleash the full potential of `Serilog`, we use `enrichers`. These enrichers give you additional details like Machine Name, ProcessId, Thread Id when the log event had occurred for better diagnostics. It makes a developer’s life quite simpler..

## Serilog Sinks

Serilog `Sinks` in simpler words relate to destinations for logging the data. In the packages that we are going to install to our ASP.NET Core application, Sinks for `Console` and `File` are included out of the box. That means we can write logs to Console and File System without adding any extra packages. Serilog supports various other destinations like `MSSQL`, `SQLite`, `SEQ` and more.

## Logger Initialization

Exceptions thrown during start-up are some of the most disruptive errors your application might face, and so the very first line of code in our Serilog-enabled app will set up logging and make sure any nasties are caught and recorded.

```cs
public class Program
{
    public static void Main(string[] args)
    {
        // HERE
        Log.Logger = new LoggerConfiguration()
            .Enrich.FromLogContext() /*Enricher*/
            .WriteTo.Console() /*Sink*/
            .CreateLogger();
        try
        {
            Log.Information("Starting up");
            CreateHostBuilder(args).Build().Run();
        }
        catch (Exception ex)
        {
            Log.Fatal(ex, "Application start-up failed");
        }
        finally
        {
            Log.CloseAndFlush();
        }
    }
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)

            // HERE
            //Uses Serilog instead of default .NET Logger
            .UseSerilog()
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

**Cleaning the default logger**

The "Logging" section that you’ll find in `appsettings.json` isn't used by `Serilog`, and can be removed:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

After cleaning up here, the configuration looks like this:

```json
{
  "AllowedHosts": "*"
}
```

## Writing your own log events

You should use `ILogger<T>` to log your own events as following

```cs
public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;

    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }

    public IActionResult Index([FromQuery] string name)
    {
        _logger.LogInformation("Hello, {Name}!", name);
        return View();
    } 
}
```

## Log Levels

I also wanted you to know about the various Logging Levels. This is the fundamental concept of logging. When we wrote `_logger.LogInformation("Hello, {Name}!", name);`, we mentioned to the application that this is a log with the log-level set to `Information`. Log levels make sense because it allows you to define the type of log. Is it a critical log? just a debug message? a warning message?

There are 7 log-levels included :

* `Trace`: Detailed messages with sensitive app data.
* `Debug`: Useful for the development environment.
* `Information`: General messages, like the way we mentioned earlier.
* `Warning`: For unexpected events.
* `Error`: For exceptions and errors.
* `Critical`: For failures that may need immediate attention.


## Streamlined request logging

Our log output will now be rather quiet. We didn’t want a dozen log events per request, but chances are, we’ll need to know what requests the app is handling in order to do most diagnostic analysis.

To switch request logging back on, we’ll add `Serilog` to the app's middleware pipeline over in `Startup.cs`. You’ll find a `Configure()` method in there like the following:

```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
        app.UseHsts();
    }
    app.UseHttpsRedirection();
    app.UseStaticFiles();

    // HERE
    app.UseSerilogRequestLogging();

    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller=Home}/{action=Index}/{id?}");
    });
}
```

We'll be a bit tactical about where we add `Serilog` into the pipeline. I tend not to want request logging for static files, so I add `UseSerilogRequestLogging()` **later** in the pipeline, as shown above.

## Read from configuration

Change `Program.Main()` as following

```cs
// Program.cs
public static void Main(string[] args)
{
    // HERE
    //Read Configuration from appSettings
    var config = new ConfigurationBuilder()
        .SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile("appsettings.json")
        .Build();

    Log.Logger = new LoggerConfiguration()
        .MinimumLevel.Debug() // <- Set the minimum level
        .Enrich.FromLogContext()
        .Enrich.WithMachineName()
        .Enrich.WithProcessId()
        .Enrich.WithThreadId()
        .Enrich.WithCorrelationId()
        .WriteTo.Console()
        /*HERE*/
        .ReadFrom.Configuration(config)
        .CreateLogger();
    try
    {
        Log.Information("Starting up");
        CreateHostBuilder(args).Build().Run();
    }
    catch (Exception ex)
    {
        Log.Fatal(ex, "Application start-up failed");
    }
    finally
    {
        Log.CloseAndFlush();
    }
}
```

**Setting up Serilog**

Add `Serilog` section to `appsettings.json`

```json
{
  "AllowedHosts": "*",
  "Serilog": 
  {
    "Using":  [ "Serilog.Sinks.Console", "Serilog.Sinks.File" ],
    "MinimumLevel": {
      "Default": "Debug",
      "Override": 
      {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console"
      },
      {
        "Name": "File",
        "Args": {
          "path": "D:\\Logs\\log.txt",
          "outputTemplate": "{Timestamp} {Message}{NewLine:1}{Exception:1}"
        }
      }
    ],
    "Destructure": [
      { "Name": "With", "Args": { "policy": "Sample.CustomPolicy, Sample" } },
      { "Name": "ToMaximumDepth", "Args": { "maximumDestructuringDepth": 4 } },
      { "Name": "ToMaximumStringLength", "Args": { "maximumStringLength": 100 } },
      { "Name": "ToMaximumCollectionCount", "Args": { "maximumCollectionCount": 10 } }
    ],
    "Enrich": [
      "FromLogContext",
      "WithMachineName",
      "WithProcessId",
      "WithThreadId"
    ],
    "Properties": {
      "ApplicationName": "Serilog.WebApplication"
    }
  }
}
```

## Useful Enrichers

Here's how to add some of the most useful enrichers.

```bash
Install-Package Serilog.Enrichers.Environment -Version 2.1.3
dotnet add package Serilog.Enrichers.Environment --version 2.1.3
<PackageReference Include="Serilog.Enrichers.Environment" Version="2.1.3" />

Install-Package Serilog.Enrichers.Process -Version 2.0.2-dev-00731
dotnet add package Serilog.Enrichers.Process --version 2.0.2-dev-00731
<PackageReference Include="Serilog.Enrichers.Process" Version="2.0.2-dev-00731" />

Install-Package Serilog.Enrichers.Thread -Version 3.2.0-dev-00747
dotnet add package Serilog.Enrichers.Thread --version 3.2.0-dev-00747
<PackageReference Include="Serilog.Enrichers.Thread" Version="3.2.0-dev-00747" />

Install-Package Serilog.Enrichers.CorrelationId -Version 3.0.1
dotnet add package Serilog.Enrichers.CorrelationId --version 3.0.1
<PackageReference Include="Serilog.Enrichers.CorrelationId" Version="3.0.1" />
```

Change your `Program.Main()` as following

```cs
// HERE
var config = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json")
    .Build();
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Verbose()
    .Enrich.FromLogContext()
    // Serilog.Enrichers.Environment
    .Enrich.WithMachineName()
    .Enrich.WithEnvironmentUserName()
    // Serilog.Enrichers.Process
    .Enrich.WithProcessId()
    .Enrich.WithProcessName()
    // Serilog.Enrichers.Thread
    .Enrich.WithThreadId()
    .Enrich.WithThreadName()
    // The {ThreadName} property will only be attached when it is not null. Otherwise it will be omitted. If you want to get this property always attached you can use the following:
    .Enrich.WithProperty(ThreadNameEnricher.ThreadNamePropertyName, "MyDefault")
    // Serilog.Enrichers.CorrelationId
    .Enrich.WithCorrelationId()
    // Change the output template to as following, {Properties} placeholder Collects and displays all of the above enrichers.
    .WriteTo.Console(outputTemplate: "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] {Message:lj} {Properties}{NewLine}{Exception}")
    .ReadFrom.Configuration(config)
    .CreateLogger();
```

For `Serilog.Enrichers.CorrelationId`, you need to add `AddHttpContextAccessor` too.

```cs
// Startup.ConfigureServices
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    // HERE
    services.AddHttpContextAccessor();
}
```

## Reference(s)

Most of the information in this article has gathered from various references.

* https://nblumhardt.com/2019/10/serilog-in-aspnetcore-3/
* https://www.codewithmukesh.com/blog/serilog-in-aspnet-core-3-1/
* https://dejanstojanovic.net/aspnet/2018/october/extending-serilog-with-additional-values-to-log/