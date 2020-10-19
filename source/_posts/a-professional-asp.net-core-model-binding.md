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

| Order | Approach                                                        |
|:-----:|-----------------------------------------------------------------|
| 1     | Form fields                                                                   |
| 2     | The request body (For `controllers` that have the `[ApiController]` attribute.) |
| 3     | Route data                                                                    |
| 4     |  Query string parameters                                                      |
| 5     | Uploaded files                                                                |

Therefore, model binding engine will try to use any of the above sources that are available in order, unless you refer to a specific source

For each target parameter or property, the sources are scanned in the order indicated in the preceding list. There are a few exceptions:

* Route data and query string values are used `only` for simple types.
* Uploaded files are bound `only` to target types that implement `IFormFile` or `IEnumerable<IFormFile>`.

If the default source is not correct or is not what you want, use one of the following attributes to specify the source:

| Attribute      | Description                          |
|:--------------:|--------------------------------------|
| [FromQuery]    | Gets values from the query string.   |
| [FromRoute]    | Gets values from route data.         |
| [FromForm]     | Gets values from posted form fields. |
| [FromBody]     | Gets values from the request body.   |
| [FromHeader]   | Gets values from HTTP headers.       |
| [FromServices] | Gets values from DI.                 |

## Model Binding for Simple Types

When Binding Simple Types the framework convert the values into the types of action method's arguments. The Simple Types are - `string`, `int`, `bool`, `float`, `datetime`, `decimal`, etc.

## Model Binding for Complex Types

When the argument of the action method is a `complex type like a class object` then Model Binding process gets all the `public properties` of the complex type and performs the `binding for each of them`.

## Default Binding Values

You may wonder what will happen if ASP.NET Core framework does not find the values of the action method's argument in any of the three locations – `Form data values`, `Routing variables` & `Query strings`. In that case it will provide the default values based on the type of the action method's argument. These are:

* `0` for `int`, `float`, `decimal`, `double`, `byte`.
* `null` for `string`.
* `01-01-0001 00:00:00` for `DateTime`.
* `Nullable types` are `null`.

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

## Query strings

## Uploaded files

## Specific Source

## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
* https://www.tektutorialshub.com/asp-net-core/asp-net-core-model-binding/
* https://wakeupandcode.com/forms-and-fields-in-asp-net-core/