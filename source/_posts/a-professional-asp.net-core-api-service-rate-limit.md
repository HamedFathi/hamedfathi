---
title: A Professional ASP.NET Core API Service - Rate Limit
date: September 20 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - api
    - ratelimit
---

`AspNetCoreRateLimit` is an ASP.NET Core rate limiting solution designed to control the rate of requests that clients can make to a Web API or MVC app based on IP address or client ID. The AspNetCoreRateLimit package contains an IpRateLimitMiddleware and a ClientRateLimitMiddleware, with each middleware you can set multiple limits for different scenarios like allowing an IP or Client to make a maximum number of calls in a time interval like per second, 15 minutes, etc. You can define these limits to address all requests made to an API or you can scope the limits to each API URL or HTTP verb and path.

<!-- more -->

Install the below package

```bash
Install-Package AspNetCoreRateLimit -Version 3.0.5
dotnet add package AspNetCoreRateLimit --version 3.0.5
<PackageReference Include="AspNetCoreRateLimit" Version="3.0.5" />
```

You can use two kinds of limitation:

* Rate limiting based on `client IP`
* Rate limiting based on `client ID`

We examine both methods together.

Add the following code 

```cs
// Startup.ConfigureServices

public void ConfigureServices(IServiceCollection services)
{
    // needed to load configuration from appsettings.json
    services.AddOptions();

    // needed to store rate limit counters and ip rules
    services.AddMemoryCache();

    //load general configuration from appsettings.json
    // IP
    services.Configure<IpRateLimitOptions>(Configuration.GetSection("IpRateLimiting"));
    // Client
    services.Configure<ClientRateLimitOptions>(Configuration.GetSection("ClientRateLimiting"));

    // IP
    //load ip rules from appsettings.json
    services.Configure<IpRateLimitPolicies>(Configuration.GetSection("IpRateLimitPolicies"));
    // Client
    //load client rules from appsettings.json
    services.Configure<ClientRateLimitPolicies>(Configuration.GetSection("ClientRateLimitPolicies"));

    // inject counter and rules stores
    services.AddSingleton<IIpPolicyStore, MemoryCacheIpPolicyStore>();
    services.AddSingleton<IRateLimitCounterStore, MemoryCacheRateLimitCounterStore>();

    services.AddControllers();

    // https://github.com/aspnet/Hosting/issues/793
    // the IHttpContextAccessor service is not registered by default.
    // the clientId/clientIp resolvers use it.
    services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();
    
    // configuration (resolvers, counter key builders)
    services.AddSingleton<IRateLimitConfiguration, RateLimitConfiguration>();
}
```

And You should register the middleware before any other components.

```cs
// Startup.Configure

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // IP
    app.UseIpRateLimiting();
    // Client
    app.UseClientRateLimiting();
    
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    app.UseHttpsRedirection();
    app.UseRouting();

    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

If you load-balance your app, you'll need to use `IDistributedCache` with `Redis` or `SQLServer` so that all kestrel instances will have the same rate limit store. Instead of the in-memory stores you should inject the distributed stores like this:

```cs
// inject counter and rules distributed cache stores

// IP
services.AddSingleton<IIpPolicyStore, DistributedCacheIpPolicyStore>();
// Client
services.AddSingleton<IClientPolicyStore, DistributedCacheClientPolicyStore>();

services.AddSingleton<IRateLimitCounterStore,DistributedCacheRateLimitCounterStore>();
```

### IP-based:

**General rules appsettings.json:**

```json
"IpRateLimiting": {
  "EnableEndpointRateLimiting": false,
  "StackBlockedRequests": false,
  "RealIpHeader": "X-Real-IP",
  "ClientIdHeader": "X-ClientId",
  "HttpStatusCode": 429,
  "IpWhitelist": [ "127.0.0.1", "::1/10", "192.168.0.0/24" ],
  "EndpointWhitelist": [ "get:/api/license", "*:/api/status" ],
  "ClientWhitelist": [ "dev-id-1", "dev-id-2" ],
  "GeneralRules": [
    {
      "Endpoint": "*:/fw/*",
      "Period": "1m",
      "Limit": 30
    },
    {
      "Endpoint": "*:/api/values",
      "Period": "15m",
      "Limit": 5
    },
    {
      "Endpoint": "*",
      "Period": "1s",
      "Limit": 2
    },
    {
      "Endpoint": "*",
      "Period": "15m",
      "Limit": 100
    },
    {
      "Endpoint": "*",
      "Period": "12h",
      "Limit": 1000
    },
    {
      "Endpoint": "*",
      "Period": "7d",
      "Limit": 10000
    }
  ]
}
```

**Specific IPs appsettings.json:**

```json
"IpRateLimitPolicies": {
  "IpRules": [
    {
      "Ip": "84.247.85.224",
      "Rules": [
        {
          "Endpoint": "*",
          "Period": "1s",
          "Limit": 10
        },
        {
          "Endpoint": "*",
          "Period": "15m",
          "Limit": 200
        }
      ]
    },
    {
      "Ip": "192.168.3.22/25",
      "Rules": [
        {
          "Endpoint": "*",
          "Period": "1s",
          "Limit": 5
        },
        {
          "Endpoint": "*",
          "Period": "15m",
          "Limit": 150
        },
        {
          "Endpoint": "*",
          "Period": "12h",
          "Limit": 500
        }
      ]
    }
  ]
}
```

### Client-based

**General rules appsettings.json:**

```json
"ClientRateLimiting": {
  "EnableEndpointRateLimiting": false,
  "StackBlockedRequests": false,
  "ClientIdHeader": "X-ClientId",
  "HttpStatusCode": 429,
  "EndpointWhitelist": [ "get:/api/license", "*:/api/status" ],
  "ClientWhitelist": [ "dev-id-1", "dev-id-2" ],
  "GeneralRules": [
    {
      "Endpoint": "*",
      "Period": "1s",
      "Limit": 2
    },
    {
      "Endpoint": "*",
      "Period": "15m",
      "Limit": 100
    },
    {
      "Endpoint": "*",
      "Period": "12h",
      "Limit": 1000
    },
    {
      "Endpoint": "*",
      "Period": "7d",
      "Limit": 10000
    }
  ]
}
```

**Specific IPs appsettings.json:**

```json
"ClientRateLimitPolicies": {
  "ClientRules": [
    {
      "ClientId": "client-id-1",
      "Rules": [
        {
          "Endpoint": "*",
          "Period": "1s",
          "Limit": 10
        },
        {
          "Endpoint": "*",
          "Period": "15m",
          "Limit": 200
        }
      ]
    },
    {
      "ClientId": "client-id-2",
      "Rules": [
        {
          "Endpoint": "*",
          "Period": "1s",
          "Limit": 5
        },
        {
          "Endpoint": "*",
          "Period": "15m",
          "Limit": 150
        },
        {
          "Endpoint": "*",
          "Period": "12h",
          "Limit": 500
        }
      ]
    }
  ]
}
```

### Update rate limits at runtime

**IP-based**

```cs
public class IpRateLimitController : Controller
{
	private readonly IpRateLimitOptions _options;
	private readonly IIpPolicyStore _ipPolicyStore;

	public IpRateLimitController(IOptions<IpRateLimitOptions> optionsAccessor, IIpPolicyStore ipPolicyStore)
	{
		_options = optionsAccessor.Value;
		_ipPolicyStore = ipPolicyStore;
	}

	[HttpGet]
	public IpRateLimitPolicies Get()
	{
		return _ipPolicyStore.Get(_options.IpPolicyPrefix);
	}

	[HttpPost]
	public void Post()
	{
		var pol = _ipPolicyStore.Get(_options.IpPolicyPrefix);

		pol.IpRules.Add(new IpRateLimitPolicy
		{
			Ip = "8.8.4.4",
			Rules = new List<RateLimitRule>(new RateLimitRule[] {
				new RateLimitRule {
					Endpoint = "*:/api/testupdate",
					Limit = 100,
					Period = "1d" }
			})
		});

		_ipPolicyStore.Set(_options.IpPolicyPrefix, pol);
	}
}
```

**Client-based**

```cs
public class ClientRateLimitController : Controller
{
	private readonly ClientRateLimitOptions _options;
	private readonly IClientPolicyStore _clientPolicyStore;

	public ClientRateLimitController(IOptions<ClientRateLimitOptions> optionsAccessor, IClientPolicyStore clientPolicyStore)
	{
		_options = optionsAccessor.Value;
		_clientPolicyStore = clientPolicyStore;
	}

	[HttpGet]
	public ClientRateLimitPolicy Get()
	{
		return _clientPolicyStore.Get($"{_options.ClientPolicyPrefix}_cl-key-1");
	}

	[HttpPost]
	public void Post()
	{
		var id = $"{_options.ClientPolicyPrefix}_cl-key-1";
		var clPolicy = _clientPolicyStore.Get(id);
		clPolicy.Rules.Add(new RateLimitRule
		{
			Endpoint = "*/api/testpolicyupdate",
			Period = "1h",
			Limit = 100
		});
		_clientPolicyStore.Set(id, clPolicy);
	}
}
```

### Quota exceeded response


You can customize the throttled response using the QuotaExceededResponse property of the IpRateLimiting or ClientRateLimiting configuration sections:

```json
"IpRateLimiting": {
    "QuotaExceededResponse": {
      "Content": "{{ \"message\": \"Whoa! Calm down, cowboy!\", \"details\": \"Quota exceeded. Maximum allowed: {0} per {1}. Please try again in {2} second(s).\" }}",
      "ContentType": "application/json",
      "StatusCode": 429
    },
}
```

* {0} - rule.Limit
* {1} - rule.Period
* {2} - retryAfter

### How to write a custom IP rate limit?

?

Enjoy!