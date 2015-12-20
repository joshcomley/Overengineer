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
  	...
    // Delete this line:
    "Microsoft.ApplicationInsights.AspNet": "1.0.0-rc2-*",
    ...
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
{% endhighlight %}