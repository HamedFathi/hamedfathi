---
title: A professional ASP.NET Core API service - Swagger
date: September 19 2020
category: aurelia
tags:
	- .net
    - asp.net core
    - web api
    - api
    - open api
    - swagger
---

### Swagger

When consuming a web API, understanding its various methods can be challenging for a developer. Swagger, also known as OpenAPI, solves the problem of generating useful documentation and help pages for web APIs. It provides benefits such as interactive documentation, client SDK generation, and API discoverability.

Swashbuckle can be added with the following approaches:

<!-- more -->

Install the below package

```bash
Install-Package Swashbuckle.AspNetCore -Version 5.6.1
dotnet add package Swashbuckle.AspNetCore --version 5.6.1
<PackageReference Include="Swashbuckle.AspNetCore" Version="5.6.1" />
```

Add the following code 

```cs
// Startup.ConfigureServices

public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    
    // Register the Swagger generator, defining 1 or more Swagger documents
    services.AddSwaggerGen();
}
```

Enable the middleware for serving the generated JSON document and the Swagger UI

```cs
// Startup.Configure

public void Configure(IApplicationBuilder app)
{
    // Enable middleware to serve generated Swagger as a JSON endpoint.
    app.UseSwagger();

    // Enable middleware to serve swagger-ui (HTML, JS, CSS, etc.),
    // specifying the Swagger JSON endpoint.
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
    });

    app.UseRouting();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

Browse Swagger doc via below URL

`http[s]://localhost:port/swagger`

#### How to set swagger on default URL?

Update the above code with the following change

```cs
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("swagger/v1/swagger.json", "My API V1");
    c.RoutePrefix = string.Empty; // HERE
});
```

Now you can browse the swagger doc url via `http[s]://localhost:port`

#### How to integrate it with ReDoc?

[ReDoc](https://github.com/Redocly/redoc) is just another implementation of Swagger UI.

Install the below package

```bash
Install-Package Swashbuckle.AspNetCore.ReDoc -Version 5.6.1
dotnet add package Swashbuckle.AspNetCore.ReDoc --version 5.6.1
<PackageReference Include="Swashbuckle.AspNetCore.ReDoc" Version="5.6.1" />
```

Update your codes

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddSwaggerGen();
}
 
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseSwagger();

    app.UseReDoc(c =>
    {
        c.SpecUrl("../swagger/v1/swagger.json");
    });

    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("v1/swagger.json", "My API V1");
    });

    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

Now you are able to browse them via below urls

Swagger: `http[s]://localhost:port/swagger`

ReDoc: `http[s]://localhost:port/api-docs`

#### How to use XML comments with Swagger?

Right-click the project in Solution Explorer and select Edit `<project_name>.csproj`.

Manually add the following lines to the `.csproj` file:

```xml
<PropertyGroup>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <NoWarn>$(NoWarn);1591</NoWarn>
</PropertyGroup>
```

Then add the below code too

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddSwaggerGen(c =>
    {
        services.AddControllers();

        // HERE
        // Set the comments path for the Swagger JSON and UI.
        var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
        var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
        c.IncludeXmlComments(xmlPath);
    });
}
```

Now you can use it like the following

```cs
/// <summary>
/// Creates a TodoItem.
/// </summary>
/// <remarks>
/// Sample request:
///
///     POST /Todo
///     {
///        "id": 1,
///        "name": "Item1",
///        "isComplete": true
///     }
///
/// </remarks>
/// <param name="item"></param>
/// <returns>A newly created TodoItem</returns>
/// <response code="201">Returns the newly created item</response>
/// <response code="400">If the item is null</response>            
[HttpPost]
[ProducesResponseType(StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public ActionResult<TodoItem> Create(TodoItem item)
{
    _context.TodoItems.Add(item);
    _context.SaveChanges();

    return CreatedAtRoute("GetTodo", new { id = item.Id }, item);
}
```

Enjoy!