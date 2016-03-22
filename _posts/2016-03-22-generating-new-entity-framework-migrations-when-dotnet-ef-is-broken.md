---
layout: blog
category: blog
splash: ""
tags: null
published: false
title: "Generating new Entity Framework migrations when dotnet-ef is broken"
---

`dotnet-ef` is a handy too to generate Entity Framework migrations and save them to disk.

Due to the instability of the .NET Core RC2 feed at the moment, it often doesn't work.

However we can generate migrations manually using the `Microsoft.EntityFrameworkCore.Commands` package, which is what `dotnet-ef` relies on.

I have put such a package together.

Right now it only is able to add a migration.

### Source code can be found here
[https://github.com/joshcomley/BrandlessOpen/tree/master/Code/dnx/Brandless.EntityFrameworkCore.Migrations]

### Nuget package can be found here
[https://www.nuget.org/packages/Brandless.EntityFrameworkCore.Migrations]

### Usage
Create a new console application in the same project as your context.