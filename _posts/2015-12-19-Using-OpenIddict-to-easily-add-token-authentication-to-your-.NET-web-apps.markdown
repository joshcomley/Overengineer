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

Now, you're going to need to pull the latest and greatest packages into your project, so you'll need to create a custom `NuGet.config` file and put it in the same folder as your solution file, and you're going to need it to look like this:

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

Lovely. So far, so Microsoft. Now let's start adding the *OpenIddict* stuff.



