---
title: A Professional ASP.NET Core - Middleware
date: October 9 2020
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

## Reference(s)

Most of the information in this article has gathered from various references.

* https://adamstorr.azurewebsites.net/blog/aspnetcore-exploring-custom-middleware
* https://www.stevejgordon.co.uk/how-is-the-asp-net-core-middleware-pipeline-built
* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write
* https://www.devtrends.co.uk/blog/conditional-middleware-based-on-request-in-asp.net-core
* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware