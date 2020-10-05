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

## In-Memory Caching in ASP.NET Core

With ASP.NET Core, it is now possible to cache the data within the application. This is known as `In-Memory` Caching in ASP.NET Core. The Application stores the data on to the server's instance which in turn drastically improves the application's performance. This is probably the easiest way to implement caching in your application.

**Pros**

* Much quicker than other forms of distributed caching as it avoids communicating over a network.
* Highly reliable.
* Best suited for Small to Mid Scale Applications.

**Cons**

* If configured incorrectly, it can consume your server's resources.
* With the scaling of application and longer caching periods, it can prove to be costly to maintain the server.
* If deployed in the cloud, maintaining consistent caches can be difficult.

## Reference(s)

Most of the information in this article has gathered from various references.

* https://www.codewithmukesh.com/blog/in-memory-caching-in-aspnet-core/