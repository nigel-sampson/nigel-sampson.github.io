---
layout: post
title: A better approach to GraphQL renames
tags: csharp graphql
---

As it typically happens I write a post on how I do something, it gets some decent traffic and then I find a better way to do it.

In this case I'm referring to a post from a few weeks ago about [Handling name collisions in GraphQL schema stitching][original], in that post I highlighted a way to use `AddMergedDocumentRewriter` to rewrite the merged schema document. It turns out there's a better way...

[Hot Chocolate][hc] supports an `ITypeRewriter` interface that when registered will be called for each type in the merged schema. Our `RenameTypeRewriter` looks like the following which is a lot cleaner in my opinion.

``` csharp
public class RenameTypeRewriter : ITypeRewriter
{
	public ITypeDefinitionNode Rewrite(ISchemaInfo schema, ITypeDefinitionNode typeDefinition)
	{
		var renameDirective = typeDefinition.Directives.SingleOrDefault(d => d.Name.Value == RenameDirectiveType.DirectiveName);

		if (renameDirective != null) {
			var newNameArgumment = renameDirective.Arguments.Single(a => a.Name.Value == RenameDirectiveType.ArgumentName );

			if (newNameArgumment.Value is StringValueNode stringValue) {
				return typeDefinition.Rename(stringValue.Value, schema.Name);
			}
		}

		return typeDefinition;
	}
}
```

Instead of our call to `AddMergedDocumentRewriter` we now have

``` csharp
.AddTypeRewriter(new RenameTypeRewriter())
```

[original]: /blog/posts/stitched-graphql-rename
[hc]: https://hotchocolate.io/