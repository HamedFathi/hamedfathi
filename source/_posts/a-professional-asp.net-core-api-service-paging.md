---
title: A Professional ASP.NET Core API Service - Paging
date: October 3 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - api
    - paging
    - pagination
    - fakedata
    - bogus
    - testdata
---

Paging refers to **getting partial results from an API**. Imagine having millions of results in the database and having your application try to return all of them at once.

Not only that would be an extremely ineffective way of returning the results, but it could also possibly have devastating effects on the application itself or the hardware it runs on. Moreover, every client has limited memory resources and it needs to restrict the number of shown results.

Thus, we need a way to return a set number of results to the client in order to avoid these consequences.

<!-- more -->

## Fake data generator

To work with a big list for paging we need a library to generate fake data. We use `Bogus` for this goal.

Install below package

```cs
Install-Package Bogus -Version 31.0.2
dotnet add package Bogus --version 31.0.2
<PackageReference Include="Bogus" Version="31.0.2" />
```

**Our Models**

We want to return list of people so I should write the following classes

```cs
// Person.cs
public class Person
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string FamilyName { get; set; }
    public float Age { get; set; }
    public DateTimeOffset BithDate { get; set; }
    public IEnumerable<Phone> Phones { get; set; }
    public IEnumerable<Address> Addresses { get; set; }
    public Person()
    {
        Phones = new List<Phone>();
        Addresses = new List<Address>();
    }
}

// Address.cs
public class Address
{
    public string Country { get; set; }
    public string City { get; set; }
    public string MainStreet { get; set; }
    public string Info { get; set; }
    public string No { get; set; }
}

// Phone.cs
public class Phone
{
    public string Code { get; set; }
    public string Number { get; set; }
}
```

**Generator**

To generate fake data we should do like below

```cs
// PeopleDataGenerator.cs
public static class PeopleDataGenerator
{
    public static IEnumerable<Person> GetPeople(int count = 200)
    {
        var testPhone = new Faker<Phone>()
                .StrictMode(true)
                .RuleFor(p => p.Code, f => f.Address.CountryCode())
                .RuleFor(p => p.Number, f => f.Phone.PhoneNumber())
                ;
        var testAddress = new Faker<Address>()
                .StrictMode(true)
                .RuleFor(a => a.Country, f => f.Address.Country())
                .RuleFor(a => a.City, f => f.Address.City())
                .RuleFor(a => a.No, f => f.Address.BuildingNumber())
                .RuleFor(a => a.Info, f => f.Address.FullAddress())
                .RuleFor(a => a.MainStreet, f => f.Address.StreetAddress())
                ;
        var testPerson = new Faker<Person>()
                .StrictMode(true)
                .RuleFor(p => p.Id, f => Guid.NewGuid())
                .RuleFor(p => p.Name, f => f.Name.FirstName())
                .RuleFor(p => p.FamilyName, f => f.Name.LastName())
                .RuleFor(p => p.Age, f => f.Random.Float(1, 120))
                .RuleFor(p => p.BithDate, f => f.Person.DateOfBirth)
                .RuleFor(p => p.Phones, f => testPhone.Generate(15))
                .RuleFor(p => p.Addresses, f => testAddress.Generate(10))
                ;
        return testPerson.Generate(count);
    }
}
```

`GetPeople()` will generate 200 people in default mode.

**Goal**

Our goal is make pagination for `Get()` action method:

```cs
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    [HttpGet]
    public IEnumerable<Person> Get()
    {
        // OUR GOAL
        var data = PeopleDataGenerator.GetPeople();
        return data;
    }
}
```

## New paging result

It’s always a good practice to add wrappers to your API response. What is a wrapper? Instead of just returning the data in the response, you have a possibility to return other parameters like error messages, response status, page number, data, page size, and so on.

So, write the following classes

```cs
// Response.cs
public class Response<T>
{
    public Response(T data)
    {
        Succeeded = true;
        Message = string.Empty;
        Errors = null;
        Data = data;
    }
    public T Data { get; set; }
    public bool Succeeded { get; set; }
    public string[] Errors { get; set; }
    public string Message { get; set; }
}

// PagedResponse.cs
using System;

public class PagedResponse<T> : Response<T>
{
    public int PageNumber { get; set; }
    public int PageSize { get; set; }
    public Uri FirstPage { get; set; }
    public Uri LastPage { get; set; }
    public int TotalPages { get; set; }
    public int TotalRecords { get; set; }
    public Uri NextPage { get; set; }
    public Uri PreviousPage { get; set; }
    public PagedResponse(T data, int pageNumber, int pageSize) : base(data)
    {
        PageNumber = pageNumber;
        PageSize = pageSize;
        Data = data;
        Message = null;
        Succeeded = true;
        Errors = null;
    }
    public PagedResponse(T data, PaginationFilter paginationFilter) : this(data, paginationFilter.PageNumber, paginationFilter.PageSize)
    {
    }
}
```

To send our filtering config we need another class

```cs
public class PaginationFilter
{
    public int PageNumber { get; set; }
    public int PageSize { get; set; }
    public PaginationFilter()
    {
        PageNumber = 1;
        PageSize = 10;
    }
    public PaginationFilter(int pageNumber, int pageSize)
    {
        PageNumber = pageNumber < 1 ? 1 : pageNumber;
        PageSize = pageSize < 1 ? 1 : pageSize;
    }
}
```

Now, we are able to do something like this:

```cs
// http://localhost:PORT/weatherforecast?pageNumber=2&pageSize=10

[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    [HttpGet]
    public IEnumerable<Person> Get([FromQuery] PaginationFilter filter)
    {
        var data = PeopleDataGenerator.GetPeople()
                .Skip((filter.PageNumber - 1) * filter.PageSize)
                .Take(filter.PageSize);
        return data;
    }
}
```

`[FromQuery]` is necessary because we will send out parameters via query strings.

## Generating Pagination URLs

One of the most challenging sections is building URIs. For this purpose we need to define a `PagedUriService` to generate the URI:

```cs
// IPagedUriService.cs
public interface IPagedUriService
{
    public Uri GetPageUri(PaginationFilter filter, string route);
}

// PagedUriService.cs
public class PagedUriService : IPagedUriService
{
    private readonly string _baseUri;
    public PagedUriService(string baseUri)
    {
        _baseUri = baseUri;
    }
    public Uri GetPageUri(PaginationFilter filter, string route)
    {
        var _enpointUri = new Uri(string.Concat(_baseUri, route));
        var modifiedUri = QueryHelpers.AddQueryString(_enpointUri.ToString(), "pageNumber", filter.PageNumber.ToString());
        modifiedUri = QueryHelpers.AddQueryString(modifiedUri, "pageSize", filter.PageSize.ToString());
        return new Uri(modifiedUri);
    }
}
```

We should add `PagedUriService` to DI.

```cs
// PagingServiceExtension.cs
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.DependencyInjection;

public static class PagingServiceExtension
{
    public static IServiceCollection AddPaging(this IServiceCollection services)
    {
        services.AddHttpContextAccessor();
        services.AddSingleton<IPagedUriService>(o =>
        {
            var accessor = o.GetRequiredService<IHttpContextAccessor>();
            var request = accessor.HttpContext.Request;
            var uri = string.Concat(request.Scheme, "://", request.Host.ToUriComponent());
            return new PagedUriService(uri);
        });
        return services;
    }
}
```

Finally, we need some functionalities to convert our raw list to paged result:

```cs
public static class PagingExtensions
{       
    public static PagedResponse<IEnumerable<T>> ToPagedReponse<T>(this IEnumerable<T> pagedData, PaginationFilter validFilter, int totalRecords, IPagedUriService uriService, string route)
    {
        var respose = new PagedResponse<IEnumerable<T>>(pagedData, validFilter.PageNumber, validFilter.PageSize);
        var totalPages = totalRecords / (double)validFilter.PageSize;
        int roundedTotalPages = Convert.ToInt32(Math.Ceiling(totalPages));
        respose.NextPage =
            validFilter.PageNumber >= 1 && validFilter.PageNumber < roundedTotalPages
            ? uriService.GetPageUri(new PaginationFilter(validFilter.PageNumber + 1, validFilter.PageSize), route)
            : null;
        respose.PreviousPage =
            validFilter.PageNumber - 1 >= 1 && validFilter.PageNumber <= roundedTotalPages
            ? uriService.GetPageUri(new PaginationFilter(validFilter.PageNumber - 1, validFilter.PageSize), route)
            : null;
        respose.FirstPage = uriService.GetPageUri(new PaginationFilter(1, validFilter.PageSize), route);
        respose.LastPage = uriService.GetPageUri(new PaginationFilter(roundedTotalPages, validFilter.PageSize), route);
        respose.TotalPages = roundedTotalPages;
        respose.TotalRecords = totalRecords;
        return respose;
    }
    public static PagedResponse<IEnumerable<T>> ToPagedReponse<T>(this IEnumerable<T> pagedData, int pageNumber, int pageSize, int totalRecords, IPagedUriService uriService, string route)
    {
        return pagedData.ToPagedReponse(new PaginationFilter(pageNumber, pageSize), totalRecords, uriService, route);
    }
    public static IActionResult ToPagedResult<T>(this IEnumerable<T> pagedData, int pageNumber, int pageSize, int totalRecords, IPagedUriService uriService, string route)
    {
        return new OkObjectResult(pagedData.ToPagedReponse(new PaginationFilter(pageNumber, pageSize), totalRecords, uriService, route));
    }
    public static IActionResult ToPagedResult<T>(this IEnumerable<T> pagedData, PaginationFilter validFilter, int totalRecords, IPagedUriService uriService, string route)
    {
        return new OkObjectResult(pagedData.ToPagedReponse(validFilter, totalRecords, uriService, route));
    }
}
```

## How to use?

Using a new paging functionality is so easy, Just follow bellow steps:

First, Register `AddPaging` service.

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    // HERE
    services.AddPaging();
}
```

Second, Pass `IPagedUriService` to the constructor of controller.
Third, Use `Request.Path.Value` to get the route data.
Fourth, Use `ToPagedResult` to convert the list to a paged result as an `IActionResult`.

```cs
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    private readonly IPagedUriService _uriService;
    public WeatherForecastController(IPagedUriService uriService /* HERE */)
    {
        _uriService = uriService;
    }
    [HttpGet]
    public IActionResult Get([FromQuery] PaginationFilter filter)
    {
        var route = Request.Path.Value;
        var data = PeopleDataGenerator.GetPeople();
        var count = data.Count();
        var pagedData = data.ToPagedResult(filter, count, _uriService, route);
        return pagedData;
    }
}
```

## Reference(s)

Most of the information in this article has gathered from various references.

* https://www.codewithmukesh.com/blog/pagination-in-aspnet-core-webapi/
* https://code-maze.com/paging-aspnet-core-webapi/
* https://medium.com/@zarkopafilis/asp-net-core-2-2-3-rest-api-26-pagination-650d0363ccf6
* https://www.carlrippon.com/scalable-and-performant-asp-net-core-web-apis-paging/
* https://schneids.net/paging-in-asp-net-web-api