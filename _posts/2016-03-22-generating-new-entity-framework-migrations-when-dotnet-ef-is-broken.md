---
layout: blog
category: blog
splash: ""
tags: 
  - "null"
published: true
title: "Generating new Entity Framework migrations when dotnet-ef is broken"
---


`dotnet-ef` is a handy too to generate Entity Framework migrations and save them to disk.

Due to the instability of the .NET Core RC2 feed at the moment, it often doesn't work.

However we can generate migrations manually using the `Microsoft.EntityFrameworkCore.Commands` package, which is what `dotnet-ef` relies on.

I have put such a package together.

Right now it only is able to add a migration.

### Source code can be found here
<https://github.com/joshcomley/BrandlessOpen/tree/master/Code/dnx/Brandless.EntityFrameworkCore.Migrations>

### Nuget package can be found here
<https://www.nuget.org/packages/Brandless.EntityFrameworkCore.Migrations>

### Usage

1 - Create a new console application in the same solution as your data context, and reference the project that contains your data context in your new console app.

2 - In your `Main()` method, add the below code, exchanging the variables to suit your environment:

{% highlight c# %}
var projectPath = Path.GetFullPath(Path.Combine(AppContext.BaseDirectory, @"..\YourApp.DataContext"));
var codeFirstMigrations = new CodeFirstMigrations<YourDbContext>(projectPath);
codeFirstMigrations.Add("NameOfTheMigration");
{% endhighlight %}

3 - Run the console app, you should have a screen a bit like this:

![2016-03-22 20_11_51-C__Windows_system32_cmd.exe.png]({{site.baseurl}}/media/2016-03-22 20_11_51-C__Windows_system32_cmd.exe.png)

Feel free to contribute to the open source project with pull requests, adding other tools from the Entity Framework commands as you might need. I am sure I'll add a few over time.

Enjoy!
