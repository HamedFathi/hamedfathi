---
title: A Professional ASP.NET Core API Service - Model Binding
date: October 9 2020
category: aspnetcoreapi
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

By default, model binding gets data in the form of key-value pairs from the following sources in an HTTP
request (in order):

* Form fields
* The request body (For controllers that have the `[ApiController]` attribute.)
* Route data
* Query string parameters
* Uploaded files

For each target parameter or property, the sources are scanned in the order indicated in the preceding list. There are a few exceptions:

* Route data and query string values are used `only` for simple types.
* Uploaded files are bound `only` to target types that implement `IFormFile` or `IEnumerable<IFormFile>`.

If the default source is not correct, use one of the following attributes to specify the source:

* `[FromQuery]`: Gets values from the query string.
* `[FromRoute]`: Gets values from route data.
* `[FromForm]`: Gets values from posted form fields.
* `[FromBody]`: Gets values from the request body.
* `[FromHeader]`: Gets values from HTTP headers.
* `[FromServices]`: Gets values from DI.

## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/aspnet/core/mvc/models/model-binding