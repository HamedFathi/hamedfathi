---
title: The .NET World - C# Source Generator
date: November 29 2020
category: dotnet
tags:
    - dotnet
    - dotnet5
    - sourcegenerator
    - roslyn
    - csharp9
    - csharp
---

I want to talk about one of the most exciting new features in `C# 9`. A way to generate the source code you want and access it instantly in your editor. Stay tuned.

<!-- more -->

## What is a source generator?

A Source Generator is a new kind of component that C# developers can write that lets you do two major things:

1. Retrieve a Compilation object that represents all user code that is being compiled. This object can be inspected and you can write code that works with the syntax and semantic models for the code being compiled, just like with analyzers today.
2. Generate C# source files that can be added to a Compilation object during the course of compilation. In other words, you can provide additional source code as input to a compilation while the code is being compiled.
When combined, these two things are what make Source Generators so useful. You can inspect user code with all of the rich metadata that the compiler builds up during compilation, then emit C# code back into the same compilation that is based on the data you’ve analyzed! If you’re familiar with Roslyn Analyzers, you can think of Source Generators as analyzers that can emit C# source code.

Source generators run as a phase of compilation visualized below:

![](/images/the-dotnet-world-csharp-source-generator/sg.png)

A Source Generator is a .NET Standard 2.0 assembly that is loaded by the compiler along with any analyzers. It is usable in environments where .NET Standard components can be loaded and run.

## What are its prerequisites?

* C# 9.0+ (SDK 5.0.100+)
* Microsoft Visual Studio 16.8.0+ or JetBrains Rider 2020.3.0+

## What are its limitations?

* Source Generators **do not allow** you to **rewrite** user source code. You can only augment a compilation by **adding** C# source files to it.
* Run **un-ordered**, each generator will see the same input compilation, with **no access** to files created by other source generators.

## What is the scenario?

The need to mock static methods in order to add a unit test is a very common problem. It’s often the case that these static methods are in third-party libraries. There are many utility libraries that are completely made up of static methods. While this makes them very easy to use, it makes them really difficult to test.

The way to mock a static method is by creating **a class that wraps the call**, **extracting an interface**, and **passing in the interface**. Then from your unit tests you can create a mock of the interface and pass it in.

In the following, we describe this method.

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

![](/images/the-dotnet-world-csharp-source-generator/solution.png)

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
}
```

**DapperSampleTest**

Install below package 

```bash
Install-Package Moq -Version 4.15.2
dotnet add package Moq --version 4.15.2
<PackageReference Include="Moq" Version="4.15.2" />
```

Then, add `DapperSample` project reference to this.

Now, we are able to test our repository.

```cs
// StudentRepositoryTest.cs

using DapperSample;
using Moq;
using System.Data;
using Xunit;

namespace DapperSampleTest
{
    public class StudentRepositoryTest
    {
        [Fact]
        public void STUDENT_REPOSITORY_TEST()
        {
            var mockConn = new Mock<IDbConnection>();
            var sut = new StudentRepository(mockConn.Object);
            var stu = sut.GetStudents();
            Assert.NotNull(stu);
        }
    }
}
```

![](/images/the-dotnet-world-csharp-source-generator/solution2.png)

**How is the test run and what happens next?**

Run the test then you will get a error like below

```bash
 DapperSampleTest.StudentRepositoryTest.STUDENT_REPOSITORY_TEST
   Source: StudentRepositoryTest.cs line 11
   Duration: 92 ms

  Message: 
    System.NullReferenceException : Object reference not set to an instance of an object.
  Stack Trace: 
    CommandDefinition.SetupCommand(IDbConnection cnn, Action`2 paramReader) line 113
    SqlMapper.QueryImpl[T](IDbConnection cnn, CommandDefinition command, Type effectiveType)+MoveNext() line 1080
    List`1.ctor(IEnumerable`1 collection)
    Enumerable.ToList[TSource](IEnumerable`1 source)
    SqlMapper.Query[T](IDbConnection cnn, String sql, Object param, IDbTransaction transaction, Boolean buffered, Nullable`1 commandTimeout, Nullable`1 commandType) line 725
    StudentRepository.GetStudents() line 18
    StudentRepositoryTest.STUDENT_REPOSITORY_TEST() line 15
```

You may have guessed why. Because  mock object of `IDbConnection` has no `Query` method in his interface. This is the problem.

**How to fix it?**

Do this step-by-step changes just like above instruction and add them to `DapperSample` project.

1. Extracting an interface.

```cs
// IDapperSqlMapper.cs
using System.Collections.Generic;
using System.Data;

public interface IDapperSqlMapper
{
    IEnumerable<T> Query<T>(IDbConnection cnn, string sql, object param = null, IDbTransaction transaction = null, bool buffered = true, int? commandTimeout = null, CommandType? commandType = null);
}
```

The `Query` is the same as what `Dapper` has.

2. A class that wraps the (static) call.

```cs
// DapperSqlMapper.cs
using Dapper;
using System.Collections.Generic;
using System.Data;

public class DapperSqlMapper : IDapperSqlMapper
{
    public IEnumerable<T> Query<T>(IDbConnection cnn, string sql, object param = null, IDbTransaction transaction = null, bool buffered = true, int? commandTimeout = null, CommandType? commandType = null)
    {
        // Dapper 'Query' method is here.
        return cnn.Query<T>(sql, param, transaction, buffered, commandTimeout, commandType);
    }
}
```

3. Change your `StudentRepository` class.

```cs
// StudentRepository.cs
using System.Collections.Generic;
using System.Data;

public class StudentRepository : IStudentRepository
{
    private readonly IDbConnection _dbConnection;
    private readonly IDapperSqlMapper _dapperSqlMapper;

    public StudentRepository(IDbConnection dbConnection, IDapperSqlMapper dapperSqlMapper)
    {
        _dbConnection = dbConnection;
        _dapperSqlMapper = dapperSqlMapper;
    }

    public IEnumerable<Student> GetStudents()
    {
        return _dapperSqlMapper.Query<Student>(_dbConnection, "SELECT * FROM STUDENT");
    }
}
```

Now change your test in `DapperSampleTest` project as following.

```cs
using DapperSample;
using Moq;
using System.Data;
using Xunit;

namespace DapperSampleTest
{
    public class StudentRepositoryTest
    {
        [Fact]
        public void STUDENT_REPOSITORY_TEST()
        {
            var mockConn = new Mock<IDbConnection>();
            var mockDapper = new Mock<IDapperSqlMapper>();
            var sut = new StudentRepository(mockConn.Object, mockDapper.Object);
            var stu = sut.GetStudents();
            Assert.NotNull(stu);
        }
    }
}
```

Run the test, you will see the green result!

![](/images/the-dotnet-world-csharp-source-generator/solution3.png)

**What is the new problem?**

Everything is good but **very tough repetitive** work especially when you are using external libraries like `Dapper` with a lot of extension (static) methods to use.

What if this repetitive method was already prepared for all methods?

## How does a source generator help us solve this problem?

So far, we have learned about the problem and how to deal with it. But now we want to use a source generator to reduce the set of these repetitive tasks to zero.

What if I have both of the following possibilities?

* Generate a wrapper class like above sample for `Math` class in background.

```cs
// Internal usage

[MockableStatic]
public class Math
{
    public static int Add(int a, int b) { return a + b; }
    public static int Sub(int a, int b) { return a - b; }
}
```

* Generate a wrapper class for `Dapper.SqlMapper` that exists in `Dapper` assembly in background.

```cs
// External usage

[MockableStatic(typeof(Dapper.SqlMapper))]
public class StudentRepositoryTest
{
    // ...
}
```

If you think it's a good idea, stay tuned.

## How to write a source generator?

First of all, go to your `MockableStaticGenerator` project. Create a `Extensions` folder.
Add the following classes:

```cs
// Extensions/StringBuilderExtensions.cs
namespace System.Text
{
    internal static class StringBuilderExtensions
    {
        internal static StringBuilder AppendIf(this StringBuilder @this, Func<string, bool> predicate, params string[] values)
        {
            foreach (var value in values)
                if (predicate(value))
                    @this.Append(value);
            return @this;
        }
        internal static StringBuilder AppendLineIf(this StringBuilder @this, Func<string, bool> predicate, params string[] values)
        {
            foreach (var value in values)
                if (predicate(value))
                    @this.AppendLine(value);
            return @this;
        }

        internal static StringBuilder AppendJoin(this StringBuilder @this, string separator, params string[] values)
        {
            @this.Append(string.Join(separator, values));
            return @this;
        }

        internal static StringBuilder AppendLineJoin(this StringBuilder @this, string separator, params string[] values)
        {
            @this.AppendLine(string.Join(separator, values));
            return @this;
        }
        internal static StringBuilder AppendFormat(this StringBuilder @this, string format, params object[] args)
        {
            @this.Append(string.Format(format, args));
            return @this;
        }
        internal static StringBuilder AppendLineFormat(this StringBuilder @this, string format, params object[] args)
        {
            @this.AppendLine(string.Format(format, args));
            return @this;
        }
        internal static string Substring(this StringBuilder @this, int startIndex)
        {
            return @this.ToString(startIndex, @this.Length - startIndex);
        }
        internal static string Substring(this StringBuilder @this, int startIndex, int length)
        {
            return @this.ToString(startIndex, length);
        }
    }
}

// Extensions/SourceGeneratorExtensions
namespace Microsoft.CodeAnalysis
{
    internal static class SourceGeneratorExtensions
    {
        // ...
    }        
}
```

We use `StringBuilder` a lot so not bad to have some useful extension methods. 
For creating our ultimate goal we need some useful extension methods to make source code sp I will add them to `SourceGeneratorExtensions` class.

Now, Create `SyntaxReceiver` class as following

```cs
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp.Syntax;
using System.Collections.Generic;

namespace MockableStaticGenerator
{
    internal class SyntaxReceiver : ISyntaxReceiver
    {
        internal List<ClassDeclarationSyntax> Classes { get; } = new List<ClassDeclarationSyntax>();

        public void OnVisitSyntaxNode(SyntaxNode syntaxNode)
        {
            if (syntaxNode is ClassDeclarationSyntax classDeclarationSyntax
                && classDeclarationSyntax.AttributeLists.Count > 0)
            {
                Classes.Add(classDeclarationSyntax);
            }
        }
    }
}
```

The purpose of this class is to **identify** the nodes we need to process the current source and generate new code. Here we store all the received classes in the `Classes` property.

Then, Create `MockableGenerator` class with below code.

```cs
using Microsoft.CodeAnalysis;

namespace MockableStaticGenerator
{
    [Generator]
    public class MockableGenerator : ISourceGenerator
    {
        public void Execute(GeneratorExecutionContext context)
        {
            
        }

        public void Initialize(GeneratorInitializationContext context)
        {
            context.RegisterForSyntaxNotifications(() => new SyntaxReceiver());
        }
    }
}
```

As you can see, You should implement `ISourceGenerator` and add `[Generator]` attribute to your source generator class.

There are two methods:

**Initialize(GeneratorInitializationContext context)**


`context.RegisterForSyntaxNotifications(() => new SyntaxReceiver());`

**Execute(GeneratorExecutionContext context)**


For next step, add `Constants` class as following to your project.

```cs
namespace MockableStaticGenerator
{
    internal class Constants
    {
        internal const string MockableStaticAttribute = @"
            namespace System
            {
                [AttributeUsage(AttributeTargets.Class, Inherited = false, AllowMultiple = false)]
                public sealed class MockableStaticAttribute : Attribute
                {
                    public MockableStaticAttribute()
                    {
                    }
                    public MockableStaticAttribute(Type type)
                    {
                    }
                }
            }";
    }
}
```

As I explained before, We need an attribute (`MockableStaticAttribute`) to annotate our classes. So, we will use above source code text in our source generator.  

* `[MockableStatic]` useful for internal usage and current class.
* `[MockableStatic(typeof(Dapper.SqlMapper))]` useful for external usage and make a wrapper for an external type.

![](/images/the-dotnet-world-csharp-source-generator/solution4.png)

Let's go to the `Execute` method, Add below code to it

```cs
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.Text;
using System.Text;

public void Execute(GeneratorExecutionContext context)
{
    context.AddSource(nameof(Constants.MockableStaticAttribute), SourceText.From(Constants.MockableStaticAttribute, Encoding.UTF8));

    if (!(context.SyntaxReceiver is SyntaxReceiver receiver))
        return;

    CSharpParseOptions options = (context.Compilation as CSharpCompilation).SyntaxTrees[0].Options as CSharpParseOptions;
    Compilation compilation = context.Compilation.AddSyntaxTrees(CSharpSyntaxTree.ParseText(SourceText.From(Constants.MockableStaticAttribute, Encoding.UTF8), options));
    INamedTypeSymbol attributeSymbol = compilation.GetTypeByMetadataName($"System.{nameof(Constants.MockableStaticAttribute)}");
}
```

First we added our `MockableStaticAttribute` text source to the project.

Next I checked if there is no `SyntaxReceiver` just return without any generated code.

The most important part is finding our `MockableStaticAttribute` text source from syntax tree.

Now, Add `MockableStaticGenerator` project to `DapperSample` as a reference project but you should update `DapperSample.csproj` file as following. 

```xml
<ItemGroup>
  <ProjectReference Include="..\MockableStaticGenerator\MockableStaticGenerator.csproj" 
                    OutputItemType="Analyzer"
                    ReferenceOutputAssembly="false"/>
</ItemGroup>
```  

This is not a "normal" `ProjectReference`. It needs the additional 'OutputItemType' and 'ReferenceOutputAssmbly' attributes.

## Visual Studio does not detect my source generators, What should I do?

Unfortunately, the current version of Visual Studio (16.8.2) has a lot of problems when you are using code generators, but you can try the following steps.

0. Make sure you follow the steps above correctly.
1. Use `dotnet clean`, Maybe you need to delete all `bin` and `obj` folders.
2. After that, use `dotnet build` to make sure your source code has no error and the problem is caused by Visual Studio.
3. Reset your Visual Studio.

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

![](/images/the-dotnet-world-csharp-source-generator/debugger1.png)

![](/images/the-dotnet-world-csharp-source-generator/debugger2.png)

![](/images/the-dotnet-world-csharp-source-generator/debugger3.png)


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
* https://github.com/dotnet/roslyn/blob/master/docs/features/source-generators.cookbook.md
