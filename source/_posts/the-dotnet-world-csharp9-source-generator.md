---
title: The .NET World - C# 9 Source Generator
date: November 29 2020
category: dotnet
tags:
    - dotnet
    - dotnet5
    - sourcegenerator
    - roslyn
    - csharp9
---

I want to talk about one of the most exciting new features in `C# 9`. A way to generate the source code you want and access it instantly in your editor. Stay tuned.

<!-- more -->

## What is a source generator?

A Source Generator is a new kind of component that C# developers can write that lets you do two major things:

1. Retrieve a Compilation object that represents all user code that is being compiled. This object can be inspected and you can write code that works with the syntax and semantic models for the code being compiled, just like with analyzers today.
2. Generate C# source files that can be added to a Compilation object during the course of compilation. In other words, you can provide additional source code as input to a compilation while the code is being compiled.
When combined, these two things are what make Source Generators so useful. You can inspect user code with all of the rich metadata that the compiler builds up during compilation, then emit C# code back into the same compilation that is based on the data you’ve analyzed! If you’re familiar with Roslyn Analyzers, you can think of Source Generators as analyzers that can emit C# source code.

Source generators run as a phase of compilation visualized below:

![](/images/the-dotnet-world-csharp9-source-generator/sg.png)

A Source Generator is a .NET Standard 2.0 assembly that is loaded by the compiler along with any analyzers. It is usable in environments where .NET Standard components can be loaded and run.

## What are its prerequisites?

* C# 9.0+ (SDK 5.0.100+)
* Microsoft Visual Studio 16.8.0+ or JetBrains Rider 2020.3.0+

## What are its limitations?

Source Generators **do not allow** you to **rewrite** user source code. You can only augment a compilation by **adding** C# source files to it.

## What is the scenario?

The need to mock static methods in order to add a unit test is a very common problem. It’s often the case that these static methods are in third-party libraries. There are many utility libraries that are completely made up of static methods. While this makes them very easy to use, it makes them really difficult to test.

So, The way to mock a static method is by creating **a class that wraps the call**, **extracting an interface**, and **passing in the interface**. Then from your unit tests you can create a mock of the interface and pass it in.

**What is Dapper?**

> A simple object mapper for .Net.

```cs
public class Dog
{
    public int? Age { get; set; }
    public Guid Id { get; set; }
    public string Name { get; set; }
    public float? Weight { get; set; }

    public int IgnoredProperty { get { return 1; } }
}

var guid = Guid.NewGuid();
var dog = connection.Query<Dog>("select Age = @Age, Id = @Id", new { Age = (int?)null, Id = guid });
```

`Dapper` contains a lot of extension (static) methods so I'm going to look at how to mock its methods with the instruction above.

**Solution structure**

Make `MockableStaticGenerator` solution with these projects:

| Name                    | Template           | Target         |
|-------------------------|--------------------|----------------|
| MockableStaticGenerator | class library      | netstandard2.0 |
| DapperSample            | class library      | netstandard2.0 |
| DapperSampleTest        | xUnit test project | net5.0         |

![](/images/the-dotnet-world-csharp9-source-generator/solution.png)

**MockableStaticGenerator**

Open `MockableStaticGenerator` project and add the following configuration to `csproj` file.

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <Version>0.0.1</Version>
    <PackageId>MockableStaticGenerator</PackageId>
    <LangVersion>latest</LangVersion>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
    <IncludeBuildOutput>false</IncludeBuildOutput>
  </PropertyGroup>
  <PropertyGroup>
    <RestoreAdditionalProjectSources>https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet5/nuget/v3/index.json ;$(RestoreAdditionalProjectSources)</RestoreAdditionalProjectSources>
  </PropertyGroup>
  <ItemGroup>
    <None Include="$(OutputPath)\$(AssemblyName).dll" Pack="true" PackagePath="analyzers/dotnet/cs" Visible="false" />
  </ItemGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.Workspaces" Version="3.8.0" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.1">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>
</Project>
```

**DapperSample**

Install the below package

```bash
Install-Package Dapper -Version 2.0.78
dotnet add package Dapper --version 2.0.78
<PackageReference Include="Dapper" Version="2.0.78" />
```

Then, make each below file in the project.

```cs
// Student.cs
using System;

public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Family { get; set; }
    public DateTime BirthDate { get; set; }
}

// IStudentRepository.cs
using System.Collections.Generic;

public interface IStudentRepository
{
    IEnumerable<Student> GetStudents();
    void SaveStudent(Student student);
}

// StudentRepository.cs
using Dapper;
using System.Collections.Generic;
using System.Data;

public class StudentRepository : IStudentRepository
{
    private readonly IDbConnection _dbConnection;

    public StudentRepository(IDbConnection dbConnection)
    {
        _dbConnection = dbConnection;
    }

    public IEnumerable<Student> GetStudents()
    {
        return _dbConnection.Query<Student>("SELECT * FROM STUDENT");
    }

    public void SaveStudent(Student student)
    {
        _dbConnection.Query("SELECT * FROM STUDENT");
    }
}
```

## How does a source generator help us solve this problem?



## How to debug it?

To start debug you can add `System.Diagnostics.Debugger.Launch();` as following:

```cs
public void Initialize(GeneratorInitializationContext context)
{
    System.Diagnostics.Debugger.Launch(); // HERE
    context.RegisterForSyntaxNotifications(() => new SyntaxReceiver());
}
```

Run the debugger and you will see it stops at `System.Diagnostics.Debugger.Launch()` line.

## How to work with files?

```xml
<ItemGroup>
    <AdditionalFiles Include="People.csv" CsvLoadType="Startup" />
    <AdditionalFiles Include="Cars.csv" CsvLoadType="OnDemand" CacheObjects="true" />
</ItemGroup>
```

## How to publish it through Nuget?



## Reference(s)

Some of the information in this article has gathered from various references.

* https://makolyte.com/how-to-mock-static-methods/
* https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/
* https://devblogs.microsoft.com/dotnet/new-c-source-generator-samples/

