---
title: A Professional ASP.NET Core API Service - FluentValidation
date: September 28 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - validation
    - fluentvalidation
---
 
`FluentValidation` is a A .NET library for building strongly-typed validation rules. It uses lambda expressions for building validation rules for your business objects. 

If you want to do simple validation in asp.net mvc application then data annotations validation is good but in case if you want to implement complex validation then you need to use `FluentValidation`.

In the following we will see how it can be added to a project and how it works.

<!-- more -->

Install the below packages

```bash
Install-Package FluentValidation -Version 9.2.2
dotnet add package FluentValidation --version 9.2.2
<PackageReference Include="FluentValidation" Version="9.2.2" />

Install-Package FluentValidation.AspNetCore -Version 9.2.0
dotnet add package FluentValidation.AspNetCore --version 9.2.0
<PackageReference Include="FluentValidation.AspNetCore" Version="9.2.0" />
```

## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.fluentvalidation.net/en/latest/index.html