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
First of all, at the time of writing `Applicaiton Insights` has no `RC2` equivalent. To this end, remove all references to `Application Insights` from your app.

If you try to force the reference in `project.json` to `RC2`, you'll see something like this:

![2015-12-20 12_58_22-Photos.png]({{site.baseurl}}/media/2015-12-20 12_58_22-Photos.png)

## project.json
As such, first up, remove even the reference to `Application Insights` from your `project.json`, deleting the line shown below:

{% highlight json %}
  "dependencies": {
  	...
    // Delete this line:
    "Microsoft.ApplicationInsights.AspNet": "1.0.0-rc2-*",
    ...
  },
{% endhighlight %}

