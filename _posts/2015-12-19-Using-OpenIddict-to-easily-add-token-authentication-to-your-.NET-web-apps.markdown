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

First up, add `.AddOpenIddict()` to the end of the `services.AddIdentity<ApplicationUser, IdentityRole>()` line in `ConfigureServices()` method in `Startup.cs` (line 9 below):

{% highlight c# linenos=table hl_lines=9 %}
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

Next up, in your `Configure(..)` method add `app.UseOpenIddict()` (line 7 below). Crucially, this must be added *after* `app.UseIdentity()` is called:

{% highlight c# linenos=table %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
	...
    app.UseIdentity();
    
    // This must be *after* "app.UseIdentity();" above
    app.UseOpenIddict();
    ...
}
{% endhighlight %}

Alright, that's your authentication server configured for `OpenIddict`.

Hit `Ctrl+F5` again and you should see the same screen as before. Nothing should be different.

For more details on configuration options with OpenIddict, see [Confniguration & Options](https://github.com/openiddict/core/wiki/Configuration-&-Options) on the OpenIddict github page.

## What exactly is this code going to do, then?
This code will create the endpoints necessary to receive authentication requests and issues cookies/tokens etc. Think of it as crating a listener in your app that will listen at points like:

`http://localhost:1212/authenticate`

You will now modify your client app so that it will send requests to these endpoints when it wants to sort things out like logging in, logging out, er, logging back in again and so on.

# Client
The client app is a little more involved.

First up, `File -> New project` yourself an `ASP.NET 5 web application`:

![2015-12-20 13_38_13-New Project.png]({{site.baseurl}}/media/2015-12-20 13_38_13-New Project.png)

This time, however, make sure you click `Change authentication` and then select `No authentication`:

![2015-12-20 13_40_10-Change Authentication.png]({{site.baseurl}}/media/2015-12-20 13_40_10-Change Authentication.png)

![2015-12-20 13_41_32-New ASP.NET Project - OpenIddictClient.png]({{site.baseurl}}/media/2015-12-20 13_41_32-New ASP.NET Project - OpenIddictClient.png)

We don't need the authentication code, because we'll be "out sourcing" that to our `OpenIddict` authentication server.

## RC2
At the time of writing, `OpenIddict` uses the `RC2` nightly builds of `ASP.NET 5`, so just as we did with the authentication server follow my guide on [moving your ASP.NET 5 project to nightly builds](/moving-your-asp-net-5-project-to-nightly-builds/).

## project.json
Now we need to add the companion authentication project references to our client web app. OpenIddict uses another project, by the same author, called `Microsoft.AspNet.Authentication.OpenIdConnect`. Add the following dependencies to your `project.json`:

{% highlight json %}
 "dependencies": {
    "Microsoft.AspNet.Authentication.Cookies": "1.0.0-rc2-*",
    "Microsoft.AspNet.Authentication.OpenIdConnect": "1.0.0-rc2-*",
}
{% endhighlight %}

## Startup.cs
Now in your `Startup.cs`, add the following using directives at the top:

{% highlight c# %}
using Microsoft.AspNet.Authentication;
using Microsoft.AspNet.Authentication.Cookies;
using Microsoft.AspNet.Http;
using Microsoft.IdentityModel.Protocols.OpenIdConnect;
{% endhighlight %}

And add the following to each of the below methods:

{% highlight c# %}
public void ConfigureServices(IServiceCollection services)
{
	// Add the below code to the top of this method
    services.Configure<SharedAuthenticationOptions>(options =>
    {
        options.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    });

    services.AddAuthentication();
    ...
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
	...
    app.UseIISPlatformHandler();

    app.UseStaticFiles();
    
    // Add the following code after your calls to "app.UseIISPlatformHandler()" and
    // "app.UseStaticFiles()"

    // Insert a new cookies middleware in the pipeline to store the user
    // identity after he has been redirected from the identity provider.
    app.UseCookieAuthentication(options =>
    {
        options.AutomaticAuthenticate = true;
        options.AutomaticChallenge = true;
        options.LoginPath = new PathString("/signin");
    });

    app.UseOpenIdConnectAuthentication(options =>
    {
        var idServerUrl = "http://localhost:50513/";
        // Note: these settings must match the application details
        // inserted in the database at the server level.
        options.ClientId = "myClient";
        options.ClientSecret = "secret_secret_secret";
        options.PostLogoutRedirectUri = "http://localhost:53984/";

        options.RequireHttpsMetadata = false;
        options.GetClaimsFromUserInfoEndpoint = true;
        options.SaveTokensAsClaims = true;

        // Use the authorization code flow.
        options.ResponseType = OpenIdConnectResponseTypes.Code;

        // Note: setting the Authority allows the OIDC client middleware to automatically
        // retrieve the identity provider's configuration and spare you from setting
        // the different endpoints URIs or the token validation parameters explicitly.
        options.Authority = idServerUrl;

        // Note: the resource property represents the different endpoints the
        // access token should be issued for (values must be space-delimited).
        options.Resource = idServerUrl;

        options.Scope.Add("email");
    });
    
	...
}
{% endhighlight %}

## Create a login page
Either replace the contents of your existing `Index.cshtml` with the following, or create a new page with the following cshtml code:

{% highlight aspx-cs %}
@{
    ViewData["Title"] = "Home Page";
}
@if (User?.Identity?.IsAuthenticated ?? false)
{
    <h1>Welcome, @User.Identity.Name</h1>
    <a class="btn btn-lg btn-danger" href="/signout">Sign out</a>
}
else
{
    <div id="myCarousel" class="carousel slide" data-ride="carousel" data-interval="6000">
        <a class="btn btn-lg btn-success" href="/home/signin">Sign in</a>
    </div>
}
{% endhighlight %}