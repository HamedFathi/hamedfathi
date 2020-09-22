---
title: A Professional ASP.NET Core API Service - DryIoc
date: September 22 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - di
    - ioc
	- dryioc
---
 
DryIoc is fast, small, full-featured IoC Container for .NET. Designed for low-ceremony use, performance, and extensibility.

<!-- more -->

```cs
public static class DryIocExtensions
{
    public static IHostBuilder UseDryIoc(this IHostBuilder hostBuilder, IServiceProviderFactory<IContainer> factory = null)
    {
        return hostBuilder.UseServiceProviderFactory(factory ?? new DryIocServiceProviderFactory());
    }
}
```

## Reference(s)

Most of the information in this article is from various sources.

* https://github.com/dadhi/DryIoc