---
layout: blog
category: blog
splash: ""
tags: 
  - "null"
published: true
title: Updating a Visual Studio 2015 template ASP.NET 5 web application from RC1 to RC2
---




There are a few gotchas when migrating Visual Studio 2015's ASP.NET 5 web application template from `RC1` to `RC2`.

This quick guide will help you through them.

# Application Insights
First of all, at the time of writing `Applicaiton Insights` has no `RC2` equivalent. If you try to force the reference in `project.json` to `RC2`, you'll see something like this:

![2015-12-20 12_58_22-Photos.png]({{site.baseurl}}/media/2015-12-20 12_58_22-Photos.png)

To this end, remove all references to `Application Insights` from your app.

## project.json
First up, remove even the reference to `Application Insights` from your `project.json`, deleting the line shown below:

{% highlight json %}
  "dependencies": {
    "Microsoft.ApplicationInsights.AspNet": "1.0.0-rc2-*",
  },
{% endhighlight %}

## Code
Now compile the app and run through any compilation errors that arise due to the now removed `Application Insights` reference.

You should find some references in `Startup.cs`. Remove all the below, if you find them, and any other references that might be in there (in case the template changed since the time of writing):

{% highlight c# %}
public Startup(IHostingEnvironment env)
{
    ...
    if (env.IsDevelopment())
    {
        ...
        // This will push telemetry data through Application Insights pipeline faster, allowing you to view results immediately.
        builder.AddApplicationInsightsSettings(developerMode: true);
        ...
    }
    ...
}

public void ConfigureServices(IServiceCollection services)
{
    ...
    services.AddApplicationInsightsTelemetry(Configuration);
    ...
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    ...
    app.UseApplicationInsightsRequestTelemetry();
    ...
    app.UseApplicationInsightsExceptionTelemetry();
    ...
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
	...
    loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    ...
}
{% endhighlight %}

## cshtml
Next up, some sneaky references still reside in some cshtml files. Delete the following lines where `ApplicationInsights` is in red:

### `\Views\Shared\_Layout.cshtml`

![2015-12-20 13_12_10-Photos.png]({{site.baseurl}}/media/2015-12-20 13_12_10-Photos.png)

### `\Views\_ViewImports.cshtml`

![2015-12-20 13_13_00-Photos.png]({{site.baseurl}}/media/2015-12-20 13_13_00-Photos.png)

`_ViewImports.cshtml` has one other sneaky change in `RC2` that we need to make. The line beginning with `@addTagHelper` needs to have the quotes removed from around `*, Microsoft.AspNet.Mvc.TagHelpers`, so it ends up looking like this:

![2015-12-20 14_14_56-Photos.png]({{site.baseurl}}/media/2015-12-20 14_14_56-Photos.png)

The red squiggly error line is fine. Ignore it. It just wants to scare you but its behind a screen, it can't touch you.

### All cshtml
Run a find and replace on all cshtml files, replacing:

`asp-validation-summary="ValidationSummary.All"`

With:

`asp-validation-summary="All"`

## Development mode

`ASP.NET 5` allows you to write funky code like this:

{% highlight c# %}
if (env.IsDevelopment())
{
    app.UseBrowserLink();
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Home/Error");
}
{% endhighlight %}

This particular example ensures that in developer environments, you see useful exceptions, whereas elsewhere you see friendly error pages.

For this to work in `RC2`, you'll need to add a new environment variable to be sure your `env` *is* `Development`. Go to your project properties, click on the `Debug` tab and add an `Environment Variable` called `ASPNET_ENV` with a value of `Development`:

![2015-12-20 16_27_01-Photos.png]({{site.baseurl}}/media/2015-12-20 16_27_01-Photos.png)

Now your application should run as expected. Enjoy!
