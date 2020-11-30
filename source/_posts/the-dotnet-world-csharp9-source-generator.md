---
title: The .NET World - C# 9 Source generator
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

* C# 9+
* Microsoft Visual Studio 16.8.0+ or JetBrains Rider 2020.3.0+

## What are its limitations?

Source Generators **do not allow** you to **rewrite** user source code. You can only augment a compilation by **adding** C# source files to it.

## What is the scenario?

So, The way to mock a static method is by creating a class that wraps the call, extracting an interface, and passing in the interface. Then from your unit tests you can create a mock of the interface and pass it in.

## How is it implemented?

**Solution structure**



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

