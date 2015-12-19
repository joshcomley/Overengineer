---
layout: blog
category: blog
title: Using OpenIddict to easily add token authentication to your .NET web apps
date: "2015-12-19 20:33"
published: true
splash: ""
tags: 
  - "null"
---


[OpenIddict](https://github.com/openiddict) is a quick and easy way to get your web application talking to an authorisation server using OAuth.

This article assumed you already know what it is, so I'm going to dive straight into talking about each step required to get your authorisation server up and running, starting from `File -> New project` for both the authorisation server and the client web app.

Let's go.

# Authorisation Server

We will make this first, so as promised, fire up Visual Studio and click `File -> New project` and choose `ASP.NET Web Application`:

![2015-12-19 20_41_20-New Project.png]({{site.baseurl}}/media/2015-12-19 20_41_20-New Project.png)

Then choose `Web Application` from under the `ASP.NE 5 Templates` section, with authentication set to *Individual User Accounts* which should be the default:

![2015-12-19 20_42_37-New ASP.NET Project - AuthorisationServer.png]({{site.baseurl}}/media/2015-12-19 20_42_37-New ASP.NET Project - AuthorisationServer.png)

You should now have a nice, fresh authorisation server web application:

![2015-12-19 20_47_09-AuthorisationServer - Microsoft Visual Studio (Administrator).png]({{site.baseurl}}/media/2015-12-19 20_47_09-AuthorisationServer - Microsoft Visual Studio (Administrator).png)

Hit `Ctrl+F5` and you should see your new app running smoothly:

![2015-12-19 20_50_10-Home Page - AuthorisationServer ‎- Microsoft Edge.png]({{site.baseurl}}/media/2015-12-19 20_50_10-Home Page - AuthorisationServer ‎- Microsoft Edge.png)

Now, before we go any further, at the time of typing *OpenIddict* uses the latest nightly builds of ASP.NET. As such we need to update our own code to use the nightly builds.

So, fire up `Command Prompt` and type in:

`dnvm upgrade -u`

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
    <add key="WebStack Nightly" value="http://www.myget.org/f/aspnetwebstacknightly/" />
    <add key="AzureAd Nightly" value="http://www.myget.org/F/azureadwebstacknightly/" />
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

Lovely. So far, so Microsoft. Now let's start adding the *OpenIddict* stuff.

Now, as it stands, your authentication project probably still has RC1 references:

![2015-12-19 21_20_23-OneNote.png]({{site.baseurl}}/media/2015-12-19 21_20_23-OneNote.png)

In `Visual Studio`, open up your project's properties and in the `Application` tab, change `Solution DNX SDK version` to `RC2`:

![2015-12-19 21_23_02-.png]({{site.baseurl}}/media/2015-12-19 21_23_02-.png)

Next up, open up your `project.json` and find and replace all occurences of `-rc1-final` with `-rc2-*`:

![2015-12-19 21_28_40-AuthorisationServer - Microsoft Visual Studio (Administrator).png]({{site.baseurl}}/media/2015-12-19 21_28_40-AuthorisationServer - Microsoft Visual Studio (Administrator).png)

As soon as you save, it'll start downloading the new packages:

![2015-12-19 21_30_57-OneNote.png]({{site.baseurl}}/media/2015-12-19 21_30_57-OneNote.png)


