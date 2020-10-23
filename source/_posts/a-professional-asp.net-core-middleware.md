---
title: A Professional ASP.NET Core - Middleware
date: October 11 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - middleware
    - pipeline
    - request
    - response
---

Middleware is software that's assembled into an app pipeline to handle requests and responses. Each component:

* Chooses whether to pass the request to the next component in the pipeline.
* Can perform work before and after the next component in the pipeline.

Request delegates are used to build the request pipeline. The request delegates handle each HTTP request.

Request delegates are configured using `Run`, `Map`, and `Use` extension methods. An individual request delegate can be specified in-line as an anonymous method (called in-line middleware), or it can be defined in a reusable class. These reusable classes and in-line anonymous methods are middleware, also called middleware components. Each middleware component in the request pipeline is responsible for invoking the next component in the pipeline or short-circuiting the pipeline. When a middleware short-circuits, it's called a terminal middleware because it prevents further middleware from processing the request.

<!-- more -->

## Create a middleware pipeline with IApplicationBuilder

The ASP.NET Core request pipeline consists of a sequence of request delegates, called one after the other. The following diagram demonstrates the concept. The thread of execution follows the black arrows.

![](/images/a-professional-asp.net-core-middleware/request-delegate-pipeline.png)

Each delegate can perform operations before and after the next delegate. Exception-handling delegates should be called early in the pipeline, so they can catch exceptions that occur in later stages of the pipeline.

The simplest possible ASP.NET Core app sets up a single request delegate that handles all requests. This case doesn't include an actual request pipeline. Instead, a single anonymous function is called in response to every HTTP request.

```cs
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        // HERE
        app.Run(async context =>
        {
            await context.Response.WriteAsync("Hello, World!");
        });
    }
}
```

## Common middlewares (in order)

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
    app.UseCookiePolicy();
 
    app.UseRouting();
    app.UseRequestLocalization();
    app.UseCors();

    app.UseAuthentication();
    app.UseAuthorization();
 
    app.UseSession();
    app.UseResponseCaching();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller=Home}/{action=Index}/{id?}");
    });
}
```

Chain multiple request delegates together with `Use`. The next parameter represents the `next` delegate in the pipeline. You can short-circuit the pipeline by **not** calling the next parameter. You can typically perform actions both before and after the next delegate, as the following example demonstrates:

```cs
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        // HERE
        app.Use(async (context, next) =>
        {
            // Do work that doesn't write to the Response.
            await next.Invoke();
            // Do logging or other work that doesn't write to the Response.
        });

        app.Run(async context =>
        {
            await context.Response.WriteAsync("Hello from 2nd delegate.");
        });
    }
}
```

When a delegate doesn't pass a request to the `next` delegate, it's called `short-circuiting` the request pipeline. Short-circuiting is often desirable because it avoids unnecessary work. For example, Static File Middleware can act as a terminal middleware by processing a request for a static file and short-circuiting the rest of the pipeline. Middleware added to the pipeline before the middleware that terminates further processing still processes code after their `next.Invoke` statements. However, see the following warning about attempting to write to a response that has already been sent.

`Run` delegates don't receive a next parameter. The first `Run` delegate is always terminal and terminates the pipeline. Run is a convention. Some middleware components may expose `Run[Middleware]` methods that run at the end of the pipeline:

## Middleware order

The following diagram shows the complete request processing pipeline for ASP.NET Core MVC and Razor Pages apps. You can see how, in a typical app, existing middlewares are ordered and where custom middlewares are added. You have full control over how to reorder existing middlewares or inject new custom middlewares as necessary for your scenarios.

![](/images/a-professional-asp.net-core-middleware/order.png)

The `Endpoint` middleware in the preceding diagram executes the filter pipeline for the corresponding app type—MVC or Razor Pages.

![](/images/a-professional-asp.net-core-middleware/endpoint.png)

## Define a basic custom middleware






## Componentising custom middleware





## Adding custom options to middleware






## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware
* https://adamstorr.azurewebsites.net/blog/aspnetcore-exploring-custom-middleware
* https://www.stevejgordon.co.uk/how-is-the-asp-net-core-middleware-pipeline-built
* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write
* https://www.devtrends.co.uk/blog/conditional-middleware-based-on-request-in-asp.net-core
* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware