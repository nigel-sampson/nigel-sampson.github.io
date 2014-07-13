---
layout: post
title: Creating a Dependency Property of type System.Type in Silverlight
tags: csharp silverlight
---

Last week I was looking at building a declarative data source control
similar to the ObjectDataSource in ASP.NET and ran into a roadblock
pretty quickly. Obviously for a control of this nature I want a few
properties of type System.Type. 

Simply declaring the property and trying to use it fails with a Xaml
parser exception, not entirely surprising as it looks like the Xaml
parser doesn&#39;t know how to convert the string representation of the
type to it the actual Type object. We can give it a helping hand using
a TypeConvertor, below is the code for a simple StringToTypeConvertor
to get around this.

``` csharp
public class StringToTypeConverter : TypeConverter
{
    public override bool CanConvertFrom(ITypeDescriptorContext context, Type sourceType)
    {
        return sourceType.IsAssignableFrom(typeof(string));
    }
 
    public override object ConvertFrom(ITypeDescriptorContext context, CultureInfo culture, object value)
    {
        var typeName = value as string;
 
        if(String.IsNullOrEmpty(typeName))
            return null;
 
        return Type.GetType(typeName);
    }
}
```

*Side note:* This isn&#39;t my idea solution, ideally I&#39;d want to use
a similar syntax to the property TargetType on Style, but alas it uses
some internal sealed classes (and I suspect some hard coding in the
Xaml parser) to achieve what it is. Obviously this style of property is
going to be used more and more, especially in the upcoming Alexandria
release of Silverlight. I really hope Microsoft don&#39;t continue to keep
this rather useful piece of functionality to themselves.

Once we&#39;ve applied our new TypeConverter as shown below we shouldn&#39;t be
receiving exceptions from the Xaml parser, but more than likely our
property is null or you&#39;re receiving exceptions from Type.GetType (this
depends on what you have in your Xaml as the type string and where the
type is). 

``` csharp
[TypeConverter(typeof(StringToTypeConverter))]
public Type DataSourceType
{
    get
    {
        return (Type)GetValue(DataSourceTypeProperty);
    }
    set
    {
        SetValue(DataSourceTypeProperty, value);
    }
}
```

From what I can gather the behavior of Type.GetType is different in
than in the standard .NET runtime. When referencing a custom type most
of the time you could write something along the lines of &quot;**CompiledExperience.Core.MyCustomType**&quot; if we are in the same assembly or &quot;**CompiledExperience.Examples.Animation.Page, CompiledExperience.Examples**&quot; if we&#39;re referencing a type in a separate assembly.

In
Silverlight the former will work, the latter however will cause an
FileLoadException in Type.GetType. What will working however is the
fully qualified assembly name &quot;**CompiledExperience.Examples.Animation.Page, CompiledExperience.Examples, Version 1.0.0.0, Culture = neutral, PublicKey = null**&quot;. Bit of a mouthful and stuff I&#39;d really prefer not have to in my Xaml (especially when Microsoft don&#39;t need to).

``` xml
DataSourceType="SilverlightExperiments.PeopleData, SilverlightExperiments, Version=1.0.0.0, Culture=neutral, PublicKey=null"
```

So
in conclusion you can have dependency properties of System.Type using a
TypeConvertor to get past Xaml parsing and using fully qualified type
names to avoid differences in Type.GetType.

Fingers crossed this changes in later versions of Silverlight.


