---
title: A Professional ASP.NET Core - Model Binding
date: October 9 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - model
    - binding
    - modelbinding
---

Controllers and Razor pages work with data that comes from HTTP requests. For example, route data may provide a record key, and posted form fields may provide values for the properties of the model. Writing code to retrieve each of these values and convert them from strings to .NET types would be tedious and error-prone. Model binding automates this process. The `model binding` system:

* Retrieves data from various sources such as route data, form fields, and query strings.
* Provides the data to controllers and Razor pages in method parameters and public properties.
* Converts string data to .NET types.
* Updates properties of complex types.

<!-- more -->

## Model Binding Sources

By default, model binding gets data in the form of key-value pairs from the following sources in an HTTP request (**in order**):

| Order | Approach                                                                        |
|:-----:|---------------------------------------------------------------------------------|
| 1     | Form fields                                                                     |
| 2     | The request body (For `controllers` that have the `[ApiController]` attribute.) |
| 3     | Route data                                                                      |
| 4     | Query string parameters                                                         |
| 5     | Uploaded files                                                                  |

Therefore, model binding engine will try to use any of the above sources that are available in order, unless you refer to a specific source

For each target parameter or property, the sources are scanned in the order indicated in the preceding list. There are a few exceptions:

* Route data and query string values are used `only` for simple types.
* Uploaded files are bound `only` to target types that implement `IFormFile` or `IEnumerable<IFormFile>`.

If the default source is not correct or is not what you want, use one of the following attributes to specify the source:

**Override binding source**

| Attribute      | Description                                          |
|----------------|------------------------------------------------------|
| [FromQuery]    | Gets values from the URL query string.               |
| [FromRoute]    | Gets values from route data.                         |
| [FromForm]     | Gets values from posted form fields. (via HTTP POST) |
| [FromBody]     | Gets values from the request body, based on configured formatter (e.g. JSON, XML). Only one action parameter can have this attribute.                |
| [FromHeader]   | Gets values from HTTP headers.                       |
| [FromServices] | Gets values from DI.                                 |

**Override binding behavior**

| Attribute      | Description                             |
|----------------|-----------------------------------------|
| [BindRequired] | Add model state error if binding fails. |
| [BindNever]    | Ignore the binding of parameter.        |

**Supply custom binding**

| Attribute     | Description                             |
|---------------|-----------------------------------------|
| [ModelBinder] | provide custom model binder.            |

## Model Binding for Simple Types

When Binding Simple Types the framework convert the values into the types of action method's arguments. The Simple Types are: `Boolean`, `Byte`, `SByte`, `Char`, `DateTime`, `DateTimeOffset`, `Decimal`, `Double`, `Enum`, `Guid`, `Int16`, `Int32`, `Int64`, `Single`, `TimeSpan`, `UInt16`, `UInt32`, `UInt64`, `Uri`, `Version`.

## Model Binding for Complex Types

When the argument of the action method is a `complex type like a class object` then Model Binding process gets all the `public properties` of the complex type and performs the `binding for each of them`.

## Default Binding Values

You may wonder what will happen if ASP.NET Core framework does not find the values of the action method's argument in any of the three locations – `Form data values`, `Routing variables` & `Query strings`. In that case it will provide the default values based on the type of the action method's argument. These are:

* For `value types`, the value will be `default(T)`
* `0` for `int`, `float`, `decimal`, `double`, `byte`.
* `01-01-0001 00:00:00` for `DateTime`.
* For `reference types`, the type is created using the `default constructor`.
* `Nullable types` are `null`.
* `null` for `string`.

## Form fields

A `ProductEditModel` object, which contains the details of the product that needs to be created or edited.

**View model**

```cs
// ProductEditModel.cs

public class ProductEditModel
{
  public int Id { get; set; }
  public string Name { get; set; }
  public decimal Rate { get; set; }
  public int Rating { get; set; }
}
```

A `form` is created to which contains three form fields. `Name`, `Rate` and `Rating`.

There are three ways front of us:

**Standard HTML**

```html
@model ProductEditModel
@{
    Layout = "_Layout";
    ViewData["Title"] = "Index";
}

<h2>Product</h2>

<form action="/Home/Create" method="post">
    <label for="Name">Name</label>
    <input type="text" name="Name" />

    <label for="Rate">Rate</label>
    <input type="text" name="Rate" />

    <label for="Rating">Rating</label>
    <input type="text" name="Rating" />

    <input type="submit" name="submit" />
</form>
```

**HTML Helper**

```html
@model ProductEditModel
@{
    Layout = "_Layout";
    ViewData["Title"] = "Index";
}

<h2>Product</h2>

@using (Html.BeginForm("Create", "Home", FormMethod.Post))
{
    <label for="Name">Name</label>
    <input type="text" name="Name" />

    <label for="Rate">Rate</label>
    <input type="text" name="Rate" />

    <label for="Rating">Rating</label>
    <input type="text" name="Rating" />

    <input type="submit" name="submit" />
}
```

**Tag Helper**

```html
@model ProductEditModel
@{
    Layout = "_Layout";
    ViewData["Title"] = "Index";
}

<h2>Product</h2>

<form asp-controller="Home" asp-action="Create" method="post">
    <label for="Name">Name</label>
    <input type="text" name="Name" />

    <label for="Rate">Rate</label>
    <input type="text" name="Rate" />

    <label for="Rating">Rating</label>
    <input type="text" name="Rating" />

    <input type="submit" name="submit" />
</form>
```

**Route Tag Helper**

```html
@model ProductEditModel
@{
    Layout = "_Layout";
    ViewData["Title"] = "Index";
}

<h2>Product</h2>

<form asp-route="MyCreateRoute" method="post">
    <label for="Name">Name</label>
    <input type="text" name="Name" />

    <label for="Rate">Rate</label>
    <input type="text" name="Rate" />

    <label for="Rating">Rating</label>
    <input type="text" name="Rating" />

    <input type="submit" name="submit" />
</form>
```

If you use above approach, you must set below attribute to your action:

```cs
[Route("/Home/Create", Name = "MyCreateRoute")]
```

**Action**

The `Create` action method in the `HomeController`.

```cs
[HttpPost]
// Just for 'Route Tag Helper' approach
// [Route("/Home/Create", Name = "MyCreateRoute")]
public IActionResult Create(ProductEditModel model)
{
    string message = "";
 
    if (ModelState.IsValid)
    {
        message = "product " + model.Name + " created successfully" ;
    }
    else
    {
        message = "Failed to create the product. Please try again";
    }
    return Content(message);
}
```

Now, When you click on the `submit` button your form information will be sent to the `Create` action and binds to the `ProductEditModel` model based on its `public properties` and corresponding HTML `name` tags.

## Request body

`Request Body` is the part of the HTTP Request where additional content can be sent to the server.

You can use `Postman` to test this approach easily.

![](/images/a-professional-asp.net-core-model-binding/postman.png)

**Request body message**

Our `ProductEditModel` model to create:


```js
// POST http://localhost:PORT/Home/Create
// Body > raw

{
  "name": "hamed",
  "rate": 20.0,
  "rating": 100
}
```

**MVC**

If you are using a `MVC` application, you must add `[FromBody]` on your `model`.

```cs
[HttpPost]
public IActionResult Create([FromBody] ProductEditModel model)
{
    string message = "";

    if (ModelState.IsValid)
    {
        message = "product " + model.Name + " created successfully";
    }
    else
    {
        message = "Failed to create the product. Please try again";
    }
    return Content(message);
}
```

**API**

If you are using an `API` application, you must add `[ApiController]` on your `controller`.

```cs
[ApiController]
public class HomeController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(ProductEditModel model)
    {
        string message = "";

        if (ModelState.IsValid)
        {
            message = "product " + model.Name + " created successfully";
        }
        else
        {
            message = "Failed to create the product. Please try again";
        }
        return Content(message);
    }
}
```

## Route data

`Route values` obtain from `URL segments` or through default values after
matching a route.

**Using optional and default values**

```html
api/{controller}/{action=index}/{id?}
```

* `api`: A literal segment.
* `{controller}`: A requierd route parameter.
* `{action=index}` An optional route parameter with default value if not provided.
* `{id?}`: An optional route parameter.

**Note:** A `segment` is a small contiguous section of a URL. It’s separated from other URL segments by at least one character, often by the `/` character. e.g. `{id}` and `{dogsOnly}` in below example.

Suppose you have the following action method:

```cs
[HttpGet("{id}/{dogsOnly}")] // Route
public ActionResult<Pet> GetById(int id, bool dogsOnly) {}
```

And the app receives a request with this URL:

```html
http://example.com/api/pets/2/true
```

Model binding goes through the following steps after the routing system selects the action method:

* Finds the first parameter of `GetByID`, an integer named id.
* Looks through the available sources in the HTTP request and finds `id = "2"` in route data.
* Converts the string "2" into integer 2.
* Finds the second parameter of `GetByID`, an boolean named dogsOnly.
* Looks through the available sources in the HTTP request and finds `dogsOnly = "true"` in route data.
* Converts the string "true" into boolean true.

**Complex types**

You are able to write route binding for complex type as following:

Create a model binding class

```cs
public class DetailsQuery
{
  [Required]
  public int? ClockNumber { get; set; }
  [Required]
  public int? YearFrom { get; set; }
  [Required]
  public int? YearTo { get; set; }
  [FromQuery] // From query string
  public bool CheckHistoricalFlag { get; set; } = false;
}
```

The action is

```cs
// http://localhost:PORT/api/employees/10/calendar/1966/2009?checkhistoricalflag=true

[HttpGet("/api/employees/{clockNumber:int}/calendar/{yearFrom:int}/{yearTo:int}")]
public ActionResult Get([FromRoute] DetailsQuery query)
{
  return Ok();
}
```

As you can see the binding engine can map each of `DetailsQuery` properties from URL segments.

**Constraints**

You can apply a large number of route constraints to route templates to ensure that route values are convertible to appropriate types. 

| Constraint              | Example              | Match examples                       | Description                                  |
|-------------------------|----------------------|--------------------------------------|----------------------------------------------|
| int                     | {count:int}          | 678, -890, 0                         | Matches any integer                          |
| decimal                 | {rate:decimal}       | 12.3, 88, -5.005                     | Matches any decimal value                    |
| Guid                    | {id:guid}            | 48ac5fbd-fd24-43b5-a742-6aab7fad67f9 | Matches any Guid                             |
| min(value)              | {age:min(22)}        | 18, 20, 21                           | Matches integer values of 22 or greater      |
| length(value)           | {name:length(7)}     | hamed, fathi, 12345                  | Matches string values with a length of 7     |
| optional int            | {count:int?}         | 456, -222, 0, null                   | Optionally matches any integer               |
| optional int max(value) | {count:int:max(15)?} | 7, -660, 0, null                     | Optionally matches any integer of 15 or less |

## Query strings

URL's are made up of several parts, like protocol, hostname, path and so on. The query string is the part of the URL that comes `after a question-mark` character. So, in a URL like this:

```html
https://www.google.com/search?q=test&oq=hello
```

Everything after the `?` character is considered the query string. The query strings are separated by `&`. In this case, there are two parameters: One called `q` and one called `oq`. They have the values "test" and "hello". These would be relevant to the page displayed by the URL.

So, `Query string values` pass at the end of the URL, not used during routing.

**Simple type**

Write an action

```cs
// HomeController.cs

public class HomeController : Controller
{
    public IActionResult QueryS1(float a, string b, bool c)
    {
        // ...
    }
}
```

You can send your values to model binding engine via query string as following

```html
GET: http://localhost:PORT/Home/QueryS1?a=1.1&b=hamed&c=true
```

**Complex type**

Create a view model

```cs
public class User
{
    public long Id { get; set; }
    public string Name { get; set; }
    public DateTime BirthDate { get; set; }
}
```

Pass it to your action

```cs
// HomeController.cs

public class HomeController : Controller
{
    public IActionResult QueryS2(User user)
    {
        // ...
    }
}
```

Call it by query strings

```cs
GET: http://localhost:PORT/Home/QueryS2?id=1&name=hamed&birthdate=1980-09-10
```

**Collections**

Suppose the parameter to be bound is an array named `selectedCourses`:

```cs
public IActionResult OnPost(int? id, int[] selectedCourses)
```

Form or query string data can be in one of the following formats:

```html
selectedCourses=1050&selectedCourses=2000 

selectedCourses[0]=1050&selectedCourses[1]=2000

[0]=1050&[1]=2000

selectedCourses[a]=1050&selectedCourses[b]=2000&selectedCourses.index=a&selectedCourses.index=b

[a]=1050&[b]=2000&index=a&index=b
```

**Dictionaries**

Suppose the target parameter is a `Dictionary<int, string>` named `selectedCourses`:

```cs
public IActionResult OnPost(int? id, Dictionary<int, string> selectedCourses)
```

Query string data can look like one of the following examples:

```html
selectedCourses[1050]=Chemistry&selectedCourses[2000]=Economics

[1050]=Chemistry&selectedCourses[2000]=Economics

selectedCourses[0].Key=1050&selectedCourses[0].Value=Chemistry&
selectedCourses[1].Key=2000&selectedCourses[1].Value=Economics

[0].Key=1050&[0].Value=Chemistry&[1].Key=2000&[1].Value=Economics
```

## Uploaded files

## Specific Sources

## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
* https://www.tektutorialshub.com/asp-net-core/asp-net-core-model-binding/
* https://wakeupandcode.com/forms-and-fields-in-asp-net-core/