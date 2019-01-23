---
layout: post
title: Moving EF Core migrations to their own assembly
tags: csharp ef
---

If you follow an introduction to EF Core tutorial then you'll tend to find the migrations end up in the same assembly as the models and DB context. For a lot of people this is fine but personally I prefer to move them off to their own assembly. For me this is about seperating a run time concern (the model and the context) from a deployment concern (the migrations), but this is completely subjective.

So if you've been following the tutorials and have all these concerns in the same assembly what does it take to seperate them? Thankfully not too much.

For the sake of this example we'll assume I have a project `MyMicroService.Core` which has a `Migrations` folder along side our application code as well as `MyMicroService.API` that is the ASP.NET Core project that makes use of the EF Core models.

1. Create the project `MyMicroService.Core.Migrations` to hold the migrations. I deliberately named this to match it up with the existing namespace of the migrations.
2. Add refereces to EF Core and related packages to the migrations project, in my case this included `Npgsql.EntityFrameworkCore.PostgreSQL`.
3. Add a reference from `MyMicroService.Core.Migrations` to `MyMicroService.Core`. The generated migrations reference the `DbContext` in code like the following, so the migrations project needs to reference the core project.
``` csharp
[DbContext(typeof(MyMicroServiceDbContext))]
```
4. Copy the migrations and model snapshot files to the new project. Given the naming we did in step 1 this should just be a copy / paste of the folder in question.
5. Configure the `DbContext` with the new assebmly
``` csharp
options.UseNpgsql(connectionString, o => o.MigrationsAssembly("MyMicroService.Core.Migrations"));
```
6. Add a reference from the startup assembly `MyMicroService.API` to the migrations assembly. This is lets us do things such as check if our database is up to date etc.

And we're done, hope this helps.

