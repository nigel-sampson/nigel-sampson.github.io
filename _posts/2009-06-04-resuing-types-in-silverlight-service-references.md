---
layout: post
title: Resuing types in Silverlight Service References
tags: wcf csharp silverlight
---

A question I see a lot on sites like Stack Overflow and the Silverlight
forums is around sharing or reusing types in WCF Service References,
either between multiple services or between the client and server.

When
you create a Service Reference in a Silverlight project Visual Studio
will generate proxies for the Service and Data Contracts. These are
partial classes so if you want to extend them with some logic (or an
interface for dependency injection) you can pretty easily. This has a
few problems, if you have multiple services that use the same data
contracts then each will have it&#39;s own proxies. For instance an
OrderService and a ProductService will often share the Product type.
Simply generating service references to both of those will cause dual
Product types in your Silverlight client.

Thankfully Visual
Studio has some tooling support to deal with this, however it has some
tricks to make it work correctly. When you create a Service Reference
there is an advanced option titled &quot;Reuse types in referenced
assemblies&quot; with some options about which references to use. The first
trick is that it&#39;s **referenced assemblies** and therefore will not use types in the assembly of the project you&#39;re adding the reference to. 

So which types will it use:

* Types in assemblies referenced by your project.
* Types with the same name and namespace.
* It doesn&#39;t matter if the client side type has the DataContract and DataMember attributes.
* It doesn&#39;t matter if the client side type has the INotifyPropertyChanged interface (although I recommend it does).

It
appears it looks for types that have the same Full Name as the
DataContract type. It doesn&#39;t appear to examine the actual signature of
the type, so if you&#39;re writing your own client types there could be a
problem there if you&#39;re not careful.

So now that we know which
types &quot;Reuse types in referenced assemblies&quot; will use how can be put
this to use. As it only uses types in referenced assemblies we need an
assembly (usually a Silverlight Class Library project) to add these
types to. Normally this isn&#39;t a problem I think only once have I needed
to add a project for just this purpose.

Now while you can create
your own types for the Silverlight Service References I prefer to share
the source code for these between the client and server. The method I
use is &quot;Add Existing Item&quot; - &quot;Add As Link&quot; in Visual Studio. That way the code is only written once and defined on both the client and server. This could cause some problems with having the different CLR versions but for most service objects the shouldn&#39;t be a problem.There are lots of articles out there about sharing code between WPF and Silverlight so find the best method for you. 

