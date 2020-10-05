---
title: A Professional ASP.NET Core API Service - Caching
date: October 5 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - cache
    - caching
    - inmemory
    - distributed
---
 
Caching is a technique of storing the frequently accessed/used data so that the future requests for those sets of data can be served much faster to the client.

In other terms, you take the most frequently used data, which is also least-modified, and copy it temporary storage so that it can be accessed much faster for the future calls from the client. This awesome technique can boost the performance of your application drastically by removing unnecessary and frequent requests to the data source.

It is important to note that applications should be designed in a way that they never depend directly on the cached memory. The Application should use the cache data only if it is available. If the cache data has expired or not available, the application would ideally request the original data source.

<!-- more -->

## Caching in ASP.NET Core

ASP.NET Core has some great out-of-the-box support for various types of caching as follows.

* `In-Memory caching`: Where the data is cached within the server's memory.
* `Distributed caching`: The data is stored external to the application in sources like Redis cache etc.

## In-Memory Caching

With ASP.NET Core, it is now possible to cache the data within the application. This is known as `In-Memory` Caching in ASP.NET Core. The Application stores the data on to the server's instance which in turn drastically improves the application's performance. This is probably the easiest way to implement caching in your application.

**Pros**

* Much quicker than other forms of distributed caching as it avoids communicating over a network.
* Highly reliable.
* Best suited for Small to Mid Scale Applications.

**Cons**

* If configured incorrectly, it can consume your server's resources.
* With the scaling of application and longer caching periods, it can prove to be costly to maintain the server.
* If deployed in the cloud, maintaining consistent caches can be difficult.

## How to add In-Memory caching?

Add `AddMemoryCache` to your services as following

```cs
// Startup.cs

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        
        // HERE
        services.AddMemoryCache();
    }
}
```

## How to use In-Memory caching?

In the Controllers folder, add a new Empty API Controller and name it `CacheController`. Here we will define just 2 endpoints using GET and POST Methods.

The POST Method will be responsible for setting the cache. Now how cache works is quite similar to a C# dictionary. That means you will need 2 parameters, a key, and a value. We will use the key to identify the value (data).

The Cache that we set earlier can be viewed using the GET Endpoint. But this depends on whether the cache is available/expired/exists.

Here is how the controller looks like.

```cs
// CacheRequest.cs

public class CacheRequest
{
    public string key { get; set; }
    public string value { get; set; }
}

// CacheController.cs

[Route("api/[controller]")]
[ApiController]
public class CacheController : ControllerBase
{
    private readonly IMemoryCache memoryCache;
    public CacheController(IMemoryCache memoryCache /* HERE */)
    {
        this.memoryCache = memoryCache;
    }
    [HttpGet("{key}")]
    public IActionResult GetCache(string key)
    {
        string value = string.Empty;
        memoryCache.TryGetValue(key, out value);
        return Ok(value);
    }
    [HttpPost]
    public IActionResult SetCache(CacheRequest data)
    {
        var cacheExpiryOptions = new MemoryCacheEntryOptions
        {
            AbsoluteExpiration = DateTime.Now.AddMinutes(5),
            Priority = CacheItemPriority.High,
            SlidingExpiration = TimeSpan.FromMinutes(2),
            Size = 1024,
        };
        memoryCache.Set(data.key, data.value, cacheExpiryOptions);
        return Ok();
    }
}
```

**Settings**

`MemoryCacheEntryOptions` is used to define the crucial properties of the concerned caching technique. We will be creating an instance of this class and passing it to the memoryCache object later on.

* `Priority`: Sets the priority of keeping the cache entry in the cache. The default setting is `Normal`. Other options are `High`, `Low` and `Never` Remove. This is pretty self-explanatory.
* `Size`: Allows you to set the size of this particular cache entry, so that it doesn't start consuming the server resources.
* `Sliding Expiration`: A defined Timespan within which a cache entry will expire if it is not used by anyone for this particular time period. In our case, we set it to 2 minutes. If, after setting the cache, no client requests for this cache entry for 2 minutes, the cache will be deleted.
* `Absolute Expiration`: The problem with `Sliding Expiration` is that theoretically, it can last forever. Let's say someone requests for the data every 1.59 minutes for the next couple of days, the application would be technically serving an outdated cache for days together. With `Absolute expiration`, we can set the actual expiration of the cache entry. Here it is set as 5 minutes. So, every 5 minutes, without taking into consideration the sliding expiration, the cache will be expired. It's always a good practice to use both these expirations checks to improve performance.

**Note**: The `Absolute Expiration` *should never be less* than the `Sliding Expiration`.

## Practical In-Memory caching implementation

For testing purposes, I have set up an API and configured Entity Framework Core. This API will return a `list of all customers` in the database.

```cs
[Route("api/[controller]")]
[ApiController]
public class CustomerController : ControllerBase
{
    private readonly IMemoryCache memoryCache;
    private readonly ApplicationDbContext context;
    public CustomerController(IMemoryCache memoryCache, ApplicationDbContext context)
    {
        this.memoryCache = memoryCache;
        this.context = context;
    }
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var cacheKey = "customerList";
        if(!memoryCache.TryGetValue(cacheKey, out List<Customer> customerList))
        {
            customerList = await context.Customers.ToListAsync();
            var cacheExpiryOptions = new MemoryCacheEntryOptions
            {
                AbsoluteExpiration = DateTime.Now.AddMinutes(5),
                Priority = CacheItemPriority.High,
                SlidingExpiration = TimeSpan.FromMinutes(2)
            };
            memoryCache.Set(cacheKey, customerList, cacheExpiryOptions);
        }
        return Ok(customerList);
    }
}
```

## Distributed Caching

ASP.NET Core supports not only in-memory application based cache, but also supports `Distribited Caching`. A distributed cache is something external to the application. It does not live within the application and need not be present in the infrastructure of the server machine as well. Distributed cache is a cache that can be shared by one or more applications/servers.
Like in-memory cache, it can improve your application response time quite drastrically. However, the implementation of Distributed Cache is application specific. This means that there are multiple cache providers that support distributed caches.

**Pros**

* Data is consistent throughout multiple servers.
* Multiple Applications / Servers can use one instance of Redis Server to cache data. This reduces the cost of maintanence in the longer run.
* Cache would not be lost on server restart and application deployment as the cache lives external to the application.
* It does not use local server’s resources.

**Cons**

* Since it is kept external, the response time may be a bit slower depending on the connection strength to the redis server.

## How to add Distributed caching?

Add `AddDistributedMemoryCache` to your services as following

```cs
// Startup.cs

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        
        // HERE
        services.AddDistributedMemoryCache();
    }
}
```


## Reference(s)

Most of the information in this article has gathered from various references.

* https://www.codewithmukesh.com/blog/in-memory-caching-in-aspnet-core/
* https://www.codewithmukesh.com/blog/redis-caching-in-aspnet-core/
* https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed