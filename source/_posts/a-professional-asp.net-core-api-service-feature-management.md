---
title: A Professional ASP.NET Core API Service - Feature Management
date: October 6 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - webapi
    - featuremanagement
    - feature
    - featureflag
---
 
The .NET Core `Feature Management` libraries provide idiomatic support for implementing feature flags in a .NET or ASP.NET Core application. These libraries allow you to declaratively add feature flags to your code so that you don't have to write all the `if` statements for them manually.

<!-- more -->

Install the below packages

```bash
Install-Package Microsoft.FeatureManagement -Version 2.2.0
dotnet add package Microsoft.FeatureManagement --version 2.2.0
<PackageReference Include="Microsoft.FeatureManagement" Version="2.2.0" />

Install-Package Microsoft.FeatureManagement.AspNetCore -Version 2.2.0
dotnet add package Microsoft.FeatureManagement.AspNetCore --version 2.2.0
<PackageReference Include="Microsoft.FeatureManagement.AspNetCore" Version="2.2.0" />
```

## Reference(s)

Most of the information in this article has gathered from various references.

* https://docs.microsoft.com/en-us/azure/azure-app-configuration/use-feature-flags-dotnet-core
* https://docs.microsoft.com/en-us/azure/azure-app-configuration/quickstart-feature-flag-aspnet-core
* http://dontcodetired.com/blog/post/Using-the-Microsoft-Feature-Toggle-Library-in-ASPNET-Core-(MicrosoftFeatureManagement)
* https://andrewlock.net/introducing-the-microsoft-featuremanagement-library-adding-feature-flags-to-an-asp-net-core-app-part-1/
* https://andrewlock.net/filtering-action-methods-with-feature-flags-adding-feature-flags-to-an-asp-net-core-app-part-2/
* https://andrewlock.net/creating-dynamic-feature-flags-with-feature-filters-adding-feature-flags-to-an-asp-net-core-app-part-3/
* https://andrewlock.net/creating-a-custom-feature-filter-adding-feature-flags-to-an-asp-net-core-app-part-4/
* https://andrewlock.net/keeping-consistent-feature-flags-across-requests-adding-feature-flags-to-an-asp-net-core-app-part-5/
* https://kasunkodagoda.com/2020/01/16/implementing-feature-flags-for-asp-net-core-applications-using-microsoft-featuremanagement-library/