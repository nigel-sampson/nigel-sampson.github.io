---
layout: post
title: Controlling the output path in Visual Studio 2017
tags: csharp xamarin
---

One thing you'll notice if you're experimenting with the new `csproj` project structure used in .NET Standard is the difference in **Output Path**. Typically the default output path for a new full .NET Framework assembly would be `bin\$(Configuration)\` resulting in `bin\Debug\` and `\bin\Release\`. In .NET Standard projects by default this output path isn't defined in the `csproj` but defaults to much the same **except** that the Target Framework is also appended to the path. So for example if I created a new project targeting .NET Standard 1.4 like follows:

``` xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard1.4</TargetFramework>
  </PropertyGroup>
</Project>
```

then the default output path would be `bin\Debug\netstandard1.4`. Nothing too different but something to watch out for. This makes sense for when instead of having one Target Framework we convert the project to target multiple frameworks, by default then each framework would have it's own output folder and we wouldn't have any file clashes.

What's also very important to note is that this appending on of the Target Framework happens automatically even when the output path is defined by you. For instance the output path of the following would be `build\Debug\netstandard1.4`.

``` xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard1.4</TargetFramework>
    <OutputPath>build\$(Configuration)</OutputPath>
  </PropertyGroup>
</Project>
```

If you want to disable this automatic appending, for instance you're only going to be using one target framework or you're defining a different output path per framework then you can use `AppendTargetFrameworkToOutputPath`.

``` xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard1.4</TargetFramework>
    <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
  </PropertyGroup>
</Project>
```