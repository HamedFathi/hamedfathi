---
title: A Professional ASP.NET Core API Service - External APIs
date: September 27 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - api
    - polly
    - refit
    - resiliency
---

In many projects we want to call external APIs and use their results in our application. In this article, we will address the following:

* HttpClientFactory
* Refit
* Polly

<!-- more -->

Suppose in our project we want to call the following address to get comprehensive information about the country we want.

The documentation and how to call it is as follows:

**Doc**: https://restcountries.eu/

**API**: https://restcountries.eu/rest/v2/name/usa

Based on the documentation and results provided as an example, we want to have a strongly typed output so I used [json2csharp](https://json2csharp.com) to convert `JSON` to `C#` but I made some changes to the result like the following:

* Replace all `int` with `double`
* Removed `Root` class
* Changed `MyArray` to `Country`

`JsonProperty` uses for `Newtonsoft.Json` library but if you want to use the new `System.Text.Json` library, you should change it to `JsonPropertyName`.

**C# Class**

```cs
public class Currency
{
    [JsonProperty("code")]
    public string Code { get; set; }

    [JsonProperty("name")]
    public string Name { get; set; }

    [JsonProperty("symbol")]
    public string Symbol { get; set; }
}

public class Language
{
    [JsonProperty("iso639_1")]
    public string Iso6391 { get; set; }

    [JsonProperty("iso639_2")]
    public string Iso6392 { get; set; }

    [JsonProperty("name")]
    public string Name { get; set; }

    [JsonProperty("nativeName")]
    public string NativeName { get; set; }
}

public class Translations
{
    [JsonProperty("de")]
    public string De { get; set; }

    [JsonProperty("es")]
    public string Es { get; set; }

    [JsonProperty("fr")]
    public string Fr { get; set; }

    [JsonProperty("ja")]
    public string Ja { get; set; }

    [JsonProperty("it")]
    public string It { get; set; }

    [JsonProperty("br")]
    public string Br { get; set; }

    [JsonProperty("pt")]
    public string Pt { get; set; }

    [JsonProperty("nl")]
    public string Nl { get; set; }

    [JsonProperty("hr")]
    public string Hr { get; set; }

    [JsonProperty("fa")]
    public string Fa { get; set; }
}

public class Country
{
    [JsonProperty("name")]
    public string Name { get; set; }

    [JsonProperty("topLevelDomain")]
    public List<string> TopLevelDomain { get; set; }

    [JsonProperty("alpha2Code")]
    public string Alpha2Code { get; set; }

    [JsonProperty("alpha3Code")]
    public string Alpha3Code { get; set; }

    [JsonProperty("callingCodes")]
    public List<string> CallingCodes { get; set; }

    [JsonProperty("capital")]
    public string Capital { get; set; }

    [JsonProperty("altSpellings")]
    public List<string> AltSpellings { get; set; }

    [JsonProperty("region")]
    public string Region { get; set; }

    [JsonProperty("subregion")]
    public string Subregion { get; set; }

    [JsonProperty("population")]
    public double Population { get; set; }

    [JsonProperty("latlng")]
    public List<double> Latlng { get; set; }

    [JsonProperty("demonym")]
    public string Demonym { get; set; }

    [JsonProperty("area")]
    // int => double
    public double Area { get; set; }

    [JsonProperty("gini")]
    public object Gini { get; set; }

    [JsonProperty("timezones")]
    public List<string> Timezones { get; set; }

    [JsonProperty("borders")]
    public List<object> Borders { get; set; }

    [JsonProperty("nativeName")]
    public string NativeName { get; set; }

    [JsonProperty("numericCode")]
    public string NumericCode { get; set; }

    [JsonProperty("currencies")]
    public List<Currency> Currencies { get; set; }

    [JsonProperty("languages")]
    public List<Language> Languages { get; set; }

    [JsonProperty("translations")]
    public Translations Translations { get; set; }

    [JsonProperty("flag")]
    public string Flag { get; set; }

    [JsonProperty("regionalBlocs")]
    public List<object> RegionalBlocs { get; set; }

    [JsonProperty("cioc")]
    public string Cioc { get; set; }
}
```

## HttpClientFactory


## Refit

```bash
<PackageReference Include="refit" Version="5.2.1" />
<PackageReference Include="Refit.HttpClientFactory" Version="5.2.1" />
<PackageReference Include="Newtonsoft.Json" Version="12.0.3" />
```

```cs
public interface ICountryApi
{
    // You have to start the URL with '/'
    [Get("/{version}/name/{country}")]
    Task<List<Country>> GetCountry(string version,string country);
}
```

```json
// appsettings.json
// Don't use '/' at the end of the URL.
"MyRefitOptions": {
  "BaseAddress": "https://restcountries.eu/rest"
}
```

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    
    services.AddRefitClient<ICountryApi>(settings)
            .ConfigureHttpClient(c => c.BaseAddress = new Uri(Configuration["MyRefitOptions:BaseAddress"]))                         
            ;
}
```


```cs
[ApiController]
[Route("[controller]")]
public class CountryController : ControllerBase
{
    private readonly ICountryApi _countryApi;

    public CountryController(ICountryApi countryApi)
    {
        _countryApi = countryApi;
    }

    [HttpGet]
    public IEnumerable<Country> Get()
    {
        var countries = _countryApi.GetCountry("v2", "usa").GetAwaiter().GetResult();
        return countries;
    }
}
```



## Using Refit with the System.Text.Json


```cs
using System.Text.Json;

var options = new JsonSerializerOptions()
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,                
    WriteIndented = true,
};

var settings = new RefitSettings()
{
    ContentSerializer = new SystemTextJsonContentSerializer(options)
};

services.AddRefitClient<ICountryApi>(settings /*HERE*/)
        .ConfigureHttpClient(c => c.BaseAddress = new Uri(Configuration["MyRefitOptions:BaseAddress"]))                         
        ;
```

## Polly

```bash
<PackageReference Include="Polly" Version="7.2.1" />
```

## Polly & HttpClientFactory



## Polly & Refit

```bash
<PackageReference Include="Microsoft.Extensions.Http.Polly" Version="3.1.8" />
```

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    // HERE
    AsyncRetryPolicy<HttpResponseMessage> retryPolicy = HttpPolicyExtensions                
            .HandleTransientHttpError()
            .Or<TimeoutRejectedException>() // Thrown by Polly's TimeoutPolicy if the inner call gets timeout.
            .WaitAndRetryAsync(10, _ => TimeSpan.FromMilliseconds(5))
            ;

    var options = new JsonSerializerOptions()
    {
        PropertyNamingPolicy = JsonNamingPolicy.CamelCase,                
        WriteIndented = true,
    };

    var settings = new RefitSettings()
    {
        ContentSerializer = new SystemTextJsonContentSerializer(options)
    };
    services.AddRefitClient<ICountryApi>(settings)
            .ConfigureHttpClient(c => c.BaseAddress = new Uri(Configuration["MyRefitOptions:BaseAddress"]))                         
            .AddPolicyHandler(retryPolicy) /*HERE*/
            ;
}
```


## Reference(s)

Most of the information in this article has gathered from various references.

* https://json2csharp.com/
* https://github.com/App-vNext/Polly
* https://github.com/reactiveui/refit
* https://github.com/19balazs86/PlayingWithRefit
* https://github.com/App-vNext/Polly.Extensions.Http
* https://anthonygiretti.com/2019/08/31/building-a-typed-httpclient-with-refit-in-asp-net-core-3/
* https://blog.martincostello.com/refit-and-system-text-json/