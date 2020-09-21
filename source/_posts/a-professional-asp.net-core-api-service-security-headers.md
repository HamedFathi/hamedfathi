---
title: A Professional ASP.NET Core API Service - Security Headers
date: September 21 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - api
    - security
    - header
---


With the help of headers, your website could send some useful information to the browser. Let’s see how it is possible to add more protection to your website.

To add a header for each request, we can use middleware.

<!-- more -->

## Headers via middleware

This is my favorite. Specifying headers in middleware can be done in C# code by creating one or more pieces of middleware. Most examples in this post will use this approach. In short, you either create a new middleware class or call the `Use` method directly in the `Configure` method in `Startup.cs`:

```cs
app.Use(async (context, next) =>
{
    if (!context.Response.Headers.ContainsKey("Header-Name"))
    {
        context.Response.Headers.Add("Header-Name", "Header-Value");
    }
    await next();
};
```

The code adds a new header named `Header-Name` to all responses. It's important to call the `Use` method **before** calling `UseEndpoints`, `UseMvc`, and similar.

## Enforce HTTPS

HTTPS is pretty awesome. It not only encrypts the traffic between the client and server so others can't see it, but also prevents others from modifying the content. So it also provides integrity. And being that you can now have HTTPS for free with services like Let's Encrypt, most apps should start to look into using HTTPS.

```cs
// Startup.ConfigureServices
public void ConfigureServices(IServiceCollection services)
{
    // Configure HTTPS redirection
    services.AddHttpsRedirection(options =>
    {
        options.RedirectStatusCode = StatusCodes.Status301MovedPermanently;
        options.HttpsPort = 443;
    });
}

// Startup.Configure
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Configure HTTPS redirection
    app.UseHttpsRedirection();
}
```

## Types of headers

The following list examines an important part of application headers.

**Strict-Transport-Security (HSTS)**

It tells the browser: "You shall only access this URL over a secure connection.". By submitting a Strict-Transport-Security header, the browser saves it and redirects itself to the HTTPS version without making an insecure call.

```cs
// Startup.ConfigureServices
public void ConfigureServices(IServiceCollection services)
{
    // Configure HSTS
    // https://aka.ms/aspnetcore-hsts
    // https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
    services.AddHsts(options =>
    {
        options.MaxAge = TimeSpan.FromDays(90);
        options.IncludeSubDomains = true;
        options.Preload = true;
    });
}

// Startup.Configure
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (!env.IsDevelopment())
    {
        // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
        app.UseHsts();
    }
}
```


**X-Frame-Options**

The `X-Frame-Options` HTTP response header can be used to indicate whether or not a browser should be allowed to render a page in a `<frame>`, `<iframe>`, `<embed>` or `<object>`. Sites can use this to avoid clickjacking attacks, by ensuring that their content is not embedded into other sites.

```cs
context.Response.Headers.Add("x-frame-options", new StringValues("DENY"));
```


## All-In-One as a middleware

```cs
public sealed class SecurityHeadersMiddleware
{
    private readonly RequestDelegate _next;

    public SecurityHeadersMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public Task Invoke(HttpContext context)
    {
        // https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy
        // TODO Change the value depending of your needs
        context.Response.Headers.Add("referrer-policy", new StringValues("strict-origin-when-cross-origin"));

        // https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options
        context.Response.Headers.Add("x-content-type-options", new StringValues("nosniff"));

        // https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
        context.Response.Headers.Add("x-frame-options", new StringValues("DENY"));

        // https://security.stackexchange.com/questions/166024/does-the-x-permitted-cross-domain-policies-header-have-any-benefit-for-my-websit
        context.Response.Headers.Add("X-Permitted-Cross-Domain-Policies", new StringValues("none"));

        // https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection
        context.Response.Headers.Add("x-xss-protection", new StringValues("1; mode=block"));

        // https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expect-CT
        // You can use https://report-uri.com/ to get notified when a misissued certificate is detected
        context.Response.Headers.Add("Expect-CT", new StringValues("max-age=0, enforce, report-uri=\"https://example.report-uri.com/r/d/ct/enforce\""));

        // https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Feature-Policy
        // https://github.com/w3c/webappsec-feature-policy/blob/master/features.md
        // https://developers.google.com/web/updates/2018/06/feature-policy
        // TODO change the value of each rule and check the documentation to see if new features are available
        context.Response.Headers.Add("Feature-Policy", new StringValues(
            "accelerometer 'none';" +
            "ambient-light-sensor 'none';" +
            "autoplay 'none';" +
            "battery 'none';" +
            "camera 'none';" +
            "display-capture 'none';" +
            "document-domain 'none';" +
            "encrypted-media 'none';" +
            "execution-while-not-rendered 'none';" +
            "execution-while-out-of-viewport 'none';" +
            "gyroscope 'none';" +
            "magnetometer 'none';" +
            "microphone 'none';" +
            "midi 'none';" +
            "navigation-override 'none';" +
            "payment 'none';" +
            "picture-in-picture 'none';" +
            "publickey-credentials-get 'none';" +
            "sync-xhr 'none';" +
            "usb 'none';" +
            "wake-lock 'none';" +
            "xr-spatial-tracking 'none';"
            ));

        // https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
        // https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy
        // TODO change the value of each rule and check the documentation to see if new rules are available
        context.Response.Headers.Add("Content-Security-Policy", new StringValues(
            "base-uri 'none';" +
            "block-all-mixed-content;" +
            "child-src 'none';" +
            "connect-src 'none';" +
            "default-src 'none';" +
            "font-src 'none';" +
            "form-action 'none';" +
            "frame-ancestors 'none';" +
            "frame-src 'none';" +
            "img-src 'none';" +
            "manifest-src 'none';" +
            "media-src 'none';" +
            "object-src 'none';" +
            "sandbox;" +
            "script-src 'none';" +
            "script-src-attr 'none';" +
            "script-src-elem 'none';" +
            "style-src 'none';" +
            "style-src-attr 'none';" +
            "style-src-elem 'none';" +
            "upgrade-insecure-requests;" +
            "worker-src 'none';"
            ));

        return _next(context);
    }
}

```

## Validation

You can check you have correctly set the security headers by using the following service: https://securityheaders.com

## Ready to use libraries

**NWebsec**

NWebsec consists of several security libraries for ASP.NET applications.

`Site`: https://github.com/NWebsec/NWebsec

`Docs`: https://docs.nwebsec.com/en/latest/

**NetEscapades.AspNetCore.SecurityHeaders**

Small package to allow adding security headers to ASP.NET Core websites 
 
`Site`: https://github.com/andrewlock/NetEscapades.AspNetCore.SecurityHeaders

## Reference(s)

Most of the information in this article is from various resources.

* https://blog.elmah.io/the-asp-net-core-security-headers-guide/
* https://www.meziantou.net/security-headers-in-asp-net-core.htm
* https://improveandrepeat.com/2019/05/how-to-improve-the-security-headers-for-your-asp-net-application/
* https://www.c-sharpcorner.com/article/asp-net-core-security-headers/
* https://andrewlock.net/adding-default-security-headers-in-asp-net-core/
* https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl
* https://joonasw.net/view/hsts-in-aspnet-core
* https://joonasw.net/view/enforcing-https-in-aspnet-core