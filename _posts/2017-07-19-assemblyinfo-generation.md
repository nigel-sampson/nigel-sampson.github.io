---
layout: post
title: Generation of Assembly Info in Visual Studio 2017
tags: csharp
---

One feature of the new `csproj` format (is there an official name for these?) that I wasn't aware of is the automatic generation of the `assmebly:` attributes we would normally see in `AssemblyInfo.cs`. This can catch you by surprise with some odd errors, especially when migrating existing projects to the format.

This feature allows you to define assembly properties such as title, description and version as project properties defined in the `.csproj` that looks like the following.

``` xml
<PropertyGroup>
  <Company>Compiled Expericne</Company>
  <Authors>Nigel Sampson</Authors>
  <PackageId>AssemblyDemo</PackageId>
  <Version>1.0.0</Version>
  <AssemblyVersion>1.0.1.0</AssemblyVersion>
  <FileVersion>1.0.1.0</FileVersion>
</PropertyGroup>
```
If you don't want to edit the project file manually these can be set through the project properties and the Package tab. One of the reasons for this is that it combines both the assembly and package information (now the build system can create your nuget packages for you) letting you define shared properties in one place.

At compile the following `AssemblyInfo.cs` is generated in the `obj` folder.

``` csharp
using System;
using System.Reflection;

[assembly: System.Reflection.AssemblyCompanyAttribute("Compiled Expericne")]
[assembly: System.Reflection.AssemblyConfigurationAttribute("Debug")]
[assembly: System.Reflection.AssemblyDescriptionAttribute("Package Description")]
[assembly: System.Reflection.AssemblyFileVersionAttribute("1.0.1.0")]
[assembly: System.Reflection.AssemblyInformationalVersionAttribute("1.0.0")]
[assembly: System.Reflection.AssemblyProductAttribute("AssemblyTest")]
[assembly: System.Reflection.AssemblyTitleAttribute("AssemblyTest")]
[assembly: System.Reflection.AssemblyVersionAttribute("1.0.1.0")]
```

If you're migrating an exsiting project you'll most likely have an existing `AssemblyInfo.cs` or using one due to properties being shared across projects with a `GlobalAssemblyInfo.cs` (what I was doing) meaning you'll see erorrs such as `CS0579	Duplicate 'System.Reflection.AssemblyCompanyAttribute' attribute`.

You can either shift to setting the new project properties and remove your existing file or turn off the new feature (which I chose to do) with the following project property.

``` xml
<PropertyGroup>
   <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
</PropertyGroup>
```

Now the `AssemblyInfo.cs` won't be automatically generated  and conflict with your already defined attributes.