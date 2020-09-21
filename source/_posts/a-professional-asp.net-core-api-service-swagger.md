---
title: A Professional ASP.NET Core API Service - Swagger
date: September 19 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - api
    - openapi
    - swagger
---


When consuming a web API, understanding its various methods can be challenging for a developer. Swagger, also known as OpenAPI, solves the problem of generating useful documentation and help pages for web APIs. It provides benefits such as interactive documentation, client SDK generation, and API discoverability.

<!-- more -->

`Swashbuckle` can be added with the following approaches:

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

## How to set swagger on default URL?

Update the above code with the following change

```cs
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("swagger/v1/swagger.json", "My API V1");
    c.RoutePrefix = string.Empty; // HERE
});
```

Now you can browse the swagger doc url via `http[s]://localhost:port`

## How to integrate it with ReDoc?

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

## How to use XML comments with Swagger?

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

## Annotations

Install the below package

```bash
Install-Package Swashbuckle.AspNetCore.Annotations -Version 5.6.1
dotnet add package Swashbuckle.AspNetCore.Annotations --version 5.6.1
<PackageReference Include="Swashbuckle.AspNetCore.Annotations" Version="5.6.1" />
```

Enable it

```cs
// Startup.ConfigureServices
services.AddSwaggerGen(config =>
{
   config.EnableAnnotations();
});
```

**Enrich Operation Metadata**

```cs
[HttpPost]
[SwaggerOperation(
    Summary = "Creates a new product",
    Description = "Requires admin privileges",
    OperationId = "CreateProduct",
    Tags = new[] { "Purchase", "Products" }
)]
public IActionResult Create([FromBody]Product product)
```

**Enrich Response Metadata**

```cs
[HttpPost]
[SwaggerResponse(201, "The product was created", typeof(Product))]
[SwaggerResponse(400, "The product data is invalid")]
public IActionResult Create([FromBody]Product product)
```

**Enrich Parameter Metadata**

You can annotate "path", "query" or "header" bound parameters or properties (i.e. decorated with `[FromRoute]`, `[FromQuery]` or `[FromHeader]`) with a `SwaggerParameterAttribute` to enrich the corresponding Parameter metadata that's generated by Swashbuckle:

```cs
[HttpGet]
public IActionResult GetProducts(
    [FromQuery, SwaggerParameter("Search keywords", Required = true)]string keywords)
```

**Enrich RequestBody Metadata**

You can annotate "body" bound parameters or properties (i.e. decorated with `[FromBody]`) with a `SwaggerRequestBodyAttribute` to enrich the corresponding `RequestBody` metadata that's generated by Swashbuckle:

```cs
[HttpPost]
public IActionResult CreateProduct(
    [FromBody, SwaggerRequestBody("The product payload", Required = true)]Product product)
```

**Enrich Schema Metadata**

```cs
[SwaggerSchema(Required = new[] { "Description" })]
public class Product
{
	[SwaggerSchema("The product identifier", ReadOnly = true)]
	public int Id { get; set; }

	[SwaggerSchema("The product description")]
	public string Description { get; set; }

	[SwaggerSchema("The date it was created", Format = "date")]
	public DateTime DateCreated { get; set; }
}
```

**Add Tag Metadata**

```cs
[SwaggerTag("Create, read, update and delete Products")]
public class ProductsController
{
}
```

**List Known Subtypes for Inheritance and Polymorphism**

```cs
// Startup.ConfigureServices
services.AddSwaggerGen(config =>
{
   config.EnableAnnotations(enableAnnotationsForInheritance: true, enableAnnotationsForPolymorphism: true);
});


// Shape.cs
[SwaggerSubType(typeof(Rectangle))]
[SwaggerSubType(typeof(Circle))]
public abstract class Shape
{
}
```

**Enrich Polymorphic Base Classes with Discriminator Metadata**

```cs
// Startup.ConfigureServices
services.AddSwaggerGen(config =>
{
    config.EnableAnnotations(enableAnnotationsForInheritance: true, enableAnnotationsForPolymorphism: true);
});

// Shape.cs
[SwaggerDiscriminator("shapeType")]
[SwaggerSubType(typeof(Rectangle), DiscriminatorValue = "rectangle")]
[SwaggerSubType(typeof(Circle), DiscriminatorValue = "circle")]
public abstract class Shape
{
    public ShapeType { get; set; }
}
```


## Reference(s)

Most of the information in this article is from various resources.

* https://docs.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle
* http://stevenmaglio.blogspot.com/2019/12/using-swagger-ui-and-redoc-in-aspnet.html