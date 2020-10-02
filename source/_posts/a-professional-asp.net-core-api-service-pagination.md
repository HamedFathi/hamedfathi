---
title: A Professional ASP.NET Core API Service - Paging
date: October 2 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - api
    - paging
    - pagination
---

Paging refers to **getting partial results from an API**. Imagine having millions of results in the database and having your application try to return all of them at once.

Not only that would be an extremely ineffective way of returning the results, but it could also possibly have devastating effects on the application itself or the hardware it runs on. Moreover, every client has limited memory resources and it needs to restrict the number of shown results.

Thus, we need a way to return a set number of results to the client in order to avoid these consequences.

<!-- more -->


## Reference(s)

Most of the information in this article has gathered from various references.

* https://code-maze.com/paging-aspnet-core-webapi/
* https://medium.com/@zarkopafilis/asp-net-core-2-2-3-rest-api-26-pagination-650d0363ccf6
* https://www.carlrippon.com/scalable-and-performant-asp-net-core-web-apis-paging/
* https://schneids.net/paging-in-asp-net-web-api