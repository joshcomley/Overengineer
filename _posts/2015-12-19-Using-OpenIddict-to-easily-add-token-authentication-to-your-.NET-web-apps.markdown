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

You can [refer to my previous blog post on how to achieve this](/moving-your-asp-net-5-project-to-nightly-builds). Make sure you follow the tutorial carefully.

Lovely. So far, so Microsoft. Now let's start adding the *OpenIddict* stuff.

Now we need to add `OpenIddict` to our `project.json`:

![2015-12-19 21_34_37-AuthorisationServer - Microsoft Visual Studio (Administrator).png]({{site.baseurl}}/media/2015-12-19 21_34_37-AuthorisationServer - Microsoft Visual Studio (Administrator).png)

Great, save that and `OpenIddict` will automagically be installed. Finally time to plug in `OpenIddict` in the actual code of your new web app.

First up, add `.AddOpenIddict()` to the end of the `services.AddIdentity<ApplicationUser, IdentityRole>()` line in `ConfigureServices()` method in `Startup.cs`:

{% highlight ruby %}
        public void ConfigureServices(IServiceCollection services)
        {
            ...

            services.AddIdentity<ApplicationUser, IdentityRole>()
                .AddEntityFrameworkStores<ApplicationDbContext>()
                .AddDefaultTokenProviders()
                // Add this line here
                .AddOpenIddict();
                
            ...
        }
{% endhighlight %}
