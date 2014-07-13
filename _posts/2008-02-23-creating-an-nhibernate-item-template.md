---
layout: post
title: Creating an NHibernate Item Template
tags: nhibernate csharp
---

Visual Studio has some excellent facilities to create simple item and
project templates, usually just create the class or project you want
and select Export Template from the File Menu. However sometimes we
want to do just a bit more than that. For instance when creating an
NHibernate mapped class class we typically create the *.cs file and a
related *.hbm.xml file, all the things we&#39;d like to automate can&#39;t be
handled by the Export Template wizard so we&#39;ll have to do a bit more
work. These are automatically inserting the assembly attribute into the
.hbm.xml, setting the build action to &quot;Embedded Resource&quot; and the
custom tool to &quot;NHibernateQueryGenerator&quot;.

There are a number of great resources out there around creating Project
and Item templates including &quot;[Creating Reusable Project and Item Templates for your Development Team](http://msdn.microsoft.com/msdnmag/issues/06/01/CodeTemplates/)&quot; which can get you started in
making the basics. Once you&#39;ve created your class template you can add
another file to the template simply by adding the file to the zip and
modifying the.vstemplate like so.

``` xml
<TemplateContent>
  <References />
  <ProjectItem SubType="Code" TargetFileName="$fileinputname$.cs" ReplaceParameters="true">Class1.cs</ProjectItem>
  <ProjectItem SubType="" TargetFileName="$fileinputname$.hbm.xml" ReplaceParameters="true">Class1.hbm.xml</ProjectItem>
</TemplateContent>
```

And the contents for our .hbm.xml is

``` xml
<?xml version="1.0" encoding="utf-8" ?>
<hibernate-mapping assembly="$assemblyName$" namespace="$rootnamespace$" xmlns="urn:nhibernate-mapping-2.0">
  <class name="$safeitemname$">
 
  </class>
</hibernate-mapping>
```

Notice that for the assembly attribute I&#39;ve used a non-standard symbol, we&#39;ll be writing some code to populate the symbol soon.

The .vstemplate has a WizardExtension setting to point to a Wizard
class that will handle any extra logic for us. This is where we&#39;ll be
doing the extra work, this class inherits from
Microsoft.VisualStudio.TemplateWizard.IWizard, the article linked above
can also give you the basics on this so I&#39;ll just focus on the
important parts. To get our symbol $assemblyName$ to work correctly we
need to add it to the replacementsDictionary parameter of the
RunStarted method. The code below determines the project we&#39;re adding
the file to and uses the AssemblyName property to create our dictionary
entry.

``` csharp
public void RunStarted(object automationObject, Dictionary<string, string> replacementsDictionary, WizardRunKind runKind, object[] customParams)
{
    if(!(automationObject is DTE2))
    {
        throw new ArgumentException("automationObject is not of type DTE2");
    }
 
    // The generated assembly name for the project is not able to be inserted via a built in replacement
    // we need to create our own
    application = automationObject as DTE2;
 
    // The project item that the item template is being created in, could be a project or a folder
    // We will need to handle each case seperatly
    SelectedItem item = application.SelectedItems.Item(1); // non-zero index
 
    Project project = null;
 
    if(item.ProjectItem != null) // We are adding the item template to something like a folder
        project = item.ProjectItem.ContainingProject;
    else if(item.Project != null) // We are adding the item template to a project
        project = item.Project;
 
    if(project == null)
        throw new ApplicationException("Could not determine project");
 
    replacementsDictionary.Add("$assemblyName$", project.Properties.Item("AssemblyName").Value.ToString());
 
}
```

The second part is in ProjectItemFinishedGenerating, we simply modify
two of the properties on the project item representing the .hbm.xml
file to the desired settings like so.

``` csharp
// Set the build action to EmbeddedResource
projectItem.Properties.Item("BuildAction").Value = 3;
projectItem.Properties.Item("CustomTool").Value = "NHibernateQueryGenerator";
```

Originally I had written some code move the .hbm.xml file so that it
was a child of the .cs file in the project tree (in the same way an
.aspx.cs file is a child of the .aspx file). However I encountered some
strange behavior when I did, the .hbm.xml files were losing their
extensions when embedded as a resource, this meant NHibernate wouldn&#39;t
automatically detect them, still don&#39;t know why this behavior exists. I&#39;ll post the complete source and templates soon.

