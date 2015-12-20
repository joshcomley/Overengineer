---
layout: blog
category: blog
splash: ""
tags: 
  - "null"
published: true
title: Moving your ASP.NET 5 project to nightly builds
---


# Moving your ASP.NET 5 project to nightly builds

So first we need to install the latest nightly builds on your machine. Fire up `Command Prompt` anywhere you like and type in:

`dnvm upgrade -u`

[DNVM is the command line operated .NET Version Manager](http://stackoverflow.com/questions/30131118/what-is-the-dnvm)

You should see something similar to this:

![2015-12-19 21_01_43-C__WINDOWS_system32_cmd.exe - dnvm  upgrade -u.png]({{site.baseurl}}/media/2015-12-19 21_01_43-C__WINDOWS_system32_cmd.exe - dnvm  upgrade -u.png)

Great, now your system has the latest builds.

Now, you're going to need to pull the latest and "greatest" nightly nuget packages into your project, so you'll need to create a custom `NuGet.config` file and put it in the *same folder as your solution file*:

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="aspnet-contrib" value="https://www.myget.org/F/aspnet-contrib/api/v2" />
    <add key="AspNetVNext" value="https://www.myget.org/F/aspnetvnext/api/v2" />
    <add key="WebStack Nightly" value="http://www.myget.org/F/aspnetwebstacknightly/" />
    <add key="Roslyn" value="https://www.myget.org/F/roslyn-nightly" />
    <add key="DotNetCore" value="https://www.myget.org/F/dotnet-core/" />
    <add key="NuGet" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
{% endhighlight %}

Now you'll need to refresh your packages to the latest version using this new `NuGet.config`, so open up `Command Prompt` *at the folder that contains your authentication project's solution file* and type:

`dnu restore --no-cache`

(Note: we're doing the `no-cache` option just to be on the safe side)

You should see some lovely things like this happening:

![2015-12-19 21_16_49-C__Windows_System32_cmd.exe - dnu  restore --no-cache.png]({{site.baseurl}}/media/2015-12-19 21_16_49-C__Windows_System32_cmd.exe - dnu  restore --no-cache.png)

Now, as it stands, your authentication project probably still has the "normal" (at the time of writing, `RC1`) references:

![2015-12-19 21_20_23-OneNote.png]({{site.baseurl}}/media/2015-12-19 21_20_23-OneNote.png)

In `Visual Studio`, open up your project's properties and in the `Application` tab, change `Solution DNX SDK version` to `RC2` (the latest at the time of writing):

![2015-12-19 21_23_02-.png]({{site.baseurl}}/media/2015-12-19 21_23_02-.png)

Next up, open up your `project.json` and find and replace all occurences of `-rc1-final` with `-rc2-*` (or whatever relevant version you want to target at the time you run through this):

![2015-12-19 21_28_40-AuthorisationServer - Microsoft Visual Studio (Administrator).png]({{site.baseurl}}/media/2015-12-19 21_28_40-AuthorisationServer - Microsoft Visual Studio (Administrator).png)

As soon as you save, it'll start downloading the new packages:

![2015-12-19 21_30_57-OneNote.png]({{site.baseurl}}/media/2015-12-19 21_30_57-OneNote.png)

![2015-12-19 21_32_36-AuthorisationServer - Microsoft Visual Studio (Administrator).png]({{site.baseurl}}/media/2015-12-19 21_32_36-AuthorisationServer - Microsoft Visual Studio (Administrator).png)

When it's done, you should have all new `RC2` references:

![2015-12-19 21_33_15-OneNote.png]({{site.baseurl}}/media/2015-12-19 21_33_15-OneNote.png)
