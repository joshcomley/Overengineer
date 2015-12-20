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

First up, add the following usings to the top of your `Startup.cs`:

{% highlight c# linenos=table %}
using OpenIddict;
using OpenIddict.Models;
{% endhighlight %}

Then, add `.AddOpenIddict()` to the end of the `services.AddIdentity<ApplicationUser, IdentityRole>()` line in `ConfigureServices()` method in `Startup.cs` (line 9 below):

{% highlight c# linenos=table %}
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

Next up, in your `Configure(..)` method add the following code (from line 7 below). Crucially, this must be added *after* `app.UseIdentity()` is called:

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

In that same method, ensure you replace anything that looks like this:

{% highlight c# %}
app.UseMvc(routes =>
{
    routes.MapRoute(
        name: "default",
        template: "{controller=Home}/{action=Index}/{id?}");
});
{% endhighlight %}

With this:

{% highlight c# %}
app.UseMvcWithDefaultRoute();
{% endhighlight %}

Alright, that's your authentication server configured for `OpenIddict`.

We haven't configured any clients in the server yet, we will come back to do that once our client is set up and we know the details we'll need.

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

And add the following to each of the below methods, ensuring you configure line `34` and `39` with your respective application URLs:

{% highlight c# linenos=table %}
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
        var idServerUrl = YOUR_AUTHENTICATION_SERVER_URL;
        // Note: these settings must match the application details
        // inserted in the database at the server level.
        options.ClientId = YOUR_CLIENT_APP_ID;
        options.ClientSecret = YOUR_CLIENT_APP_SECRET;
        options.PostLogoutRedirectUri = YOUR_CLIENT_APP_URL;

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

For the above code to compile, you'll need to fill in the following variables:
- YOUR_CLIENT_APP_ID
  - This is just the ID of your client. For now you can just use something like "My Client", nice and professinal, like
- YOUR_CLIENT_APP_SECRET
  - Again, we're just getting this up and running so choose anything, but for production apps ensure this is kept absolutely private and is impossible to guess. Best to just use a random guid.
- YOUR_CLIENT_APP_URL
  - This is just the URL your client app is running at, for now it's likely some localhost URL
- YOUR_AUTHENTICATION_SERVER_URL
  - This is the URL of your authrozation server we configured earlier, again likely to be some localhost URL

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
    <a class="btn btn-lg btn-success" href="/home/signin">Sign in</a>
}
{% endhighlight %}

Now add the following usings and methods to the controller for the above cshtml file:

{% highlight c# %}
using Microsoft.AspNet.Authentication.Cookies;
using Microsoft.AspNet.Authentication.OpenIdConnect;
using Microsoft.AspNet.Http.Authentication;

...

public IActionResult SignIn()
{
    return new ChallengeResult(OpenIdConnectDefaults.AuthenticationScheme, new AuthenticationProperties
    {
        RedirectUri = "/"
    });
}

[HttpGet("~/signout"), HttpPost("~/signout")]
public async Task SignOut()
{
    // Instruct the cookies middleware to delete the local cookie created when the user agent
    // is redirected from the identity provider after a successful authorization flow.
    await HttpContext.Authentication.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);

    // Instruct the OpenID Connect middleware to redirect the user agent to the identity provider to sign out.
    await HttpContext.Authentication.SignOutAsync(OpenIdConnectDefaults.AuthenticationScheme);
}
{% endhighlight %}

The `SignOut()` method above can be in any controller, because we've mapped a specific route, which is the relative route `~/signout` to always go to this action.

Now, hit `Ctrl+F5` and you should see a nice big green `Sign in` button:

![2015-12-20 15_00_26-Home Page - OpenIddictClient ‎- Microsoft Edge.png]({{site.baseurl}}/media/2015-12-20 15_00_26-Home Page - OpenIddictClient ‎- Microsoft Edge.png)

Before we click it...

# Back to the authorisation server project
...we need to dip back into our authorisation server to configure our new clientn app.

The first thing we need to do is make our data context inherit from `OpenIddictContext`. So open up your `ApplicationDbContext.cs` and add the following using:

{% highlight c# %}
using OpenIddict;
{% endhighlight %}

Then change the class definition to this:

{% highlight c# %}
public class ApplicationDbContext : OpenIddictContext<ApplicationUser>
{% endhighlight %}

Then add the following method:

{% highlight c# %}
private static bool _databaseChecked;
// The following code creates the database and schema if they don't exist.
// This is a temporary workaround since deploying database through EF migrations is
// not yet supported in this release.
// Please see this http://go.microsoft.com/fwlink/?LinkID=615859 for more information on how to do deploy the database
// when publishing your application.
public void EnsureDatabaseCreated()
{
    if (!_databaseChecked)
    {
        _databaseChecked = true;
        this.Database.EnsureCreated();
    }
}
{% endhighlight %}

Your final class should look something like this:

{% highlight c# %}
public class ApplicationDbContext : OpenIddictContext<ApplicationUser>
{
    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        // Customize the ASP.NET Identity model and override the defaults if needed.
        // For example, you can rename the ASP.NET Identity table names and more.
        // Add your customizations after calling base.OnModelCreating(builder);
    }

    private static bool _databaseChecked;
    // The following code creates the database and schema if they don't exist.
    // This is a temporary workaround since deploying database through EF migrations is
    // not yet supported in this release.
    // Please see this http://go.microsoft.com/fwlink/?LinkID=615859 for more information on how to do deploy the database
    // when publishing your application.
    public void EnsureDatabaseCreated()
    {
        if (!_databaseChecked)
        {
            _databaseChecked = true;
            this.Database.EnsureCreated();
        }
    }
}
{% endhighlight %}

Now open up your `AccountController.cs`.

This next part is a bit of a hack for now, as we need to ensure that the database is created and ready for `OpenIddict` to use when account stuff happens.

Firstly, we need to pass in our data context into our `AccountController` now because we're going to be looking up clients etc. so add `ApplicationDbContext` to your constructor and save a reference to it in a private field named `_applicationDbContext`. Your new constructor should look something like the following:

{% highlight c# %}
// Create this private field
private readonly ApplicationDbContext _applicationDbContext;

public AccountController(
    UserManager<ApplicationUser> userManager,
    SignInManager<ApplicationUser> signInManager,
    IEmailSender emailSender,
    ISmsSender smsSender,
    ILoggerFactory loggerFactory,
    // Add this in
    ApplicationDbContext applicationDbContext)
{
    _userManager = userManager;
    _signInManager = signInManager;
    _emailSender = emailSender;
    _smsSender = smsSender;
    _logger = loggerFactory.CreateLogger<AccountController>();
    // Save a reference to it
    _applicationDbContext = applicationDbContext;
}
{% endhighlight %}

Now add the following line to the top of each of the below three methods:

{% highlight c# %}
//
// POST: /Account/Register
[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Register(RegisterViewModel model)
{
    _applicationDbContext.EnsureDatabaseCreated();
    ...
}

//
// POST: /Account/ExternalLogin
[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public IActionResult ExternalLogin(string provider, string returnUrl = null)
{
    _applicationDbContext.EnsureDatabaseCreated();
    ...
}

//
// POST: /Account/Login
[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null)
{
    _applicationDbContext.EnsureDatabaseCreated();
    ...
}
{% endhighlight %}

Now, open up your authorisation server's `Startup.cs` again and add the following code *beneath* `app.UseOpenIddict()`:

{% highlight c# %}
using (var context = app.ApplicationServices.GetRequiredService<ApplicationDbContext>())
{
    context.Database.EnsureCreated();

    // Add Mvc.Client to the known applications.
    if (!context.Applications.Any())
    {
        // Note: when using the introspection middleware, your resource server
        // MUST be registered as an OAuth2 client and have valid credentials.
        // 
        // context.Applications.Add(new Application {
        //     Id = "resource_server",
        //     DisplayName = "Main resource server",
        //     Secret = "875sqd4s5d748z78z7ds1ff8zz8814ff88ed8ea4z4zzd"
        // });

        context.Applications.Add(new Application
        {
            Id = YOUR_CLIENT_APP_ID,
            DisplayName = "My client application",
            RedirectUri = YOUR_AUTHORISATION_APP_URL + "/signin-oidc",
            LogoutRedirectUri = YOUR_CLIENT_APP_URL,
            Secret = Crypto.HashPassword(YOUR_CLIENT_APP_SECRET),
            Type = OpenIddictConstants.ApplicationTypes.Confidential
        });

        context.SaveChanges();
    }
}
{% endhighlight %}

Again, you'll need to fill in the variables for `YOUR_AUTHORISATION_APP_URL` and `YOUR_CLIENT_APP_URL`, `YOUR_CLIENT_APP_ID` and `YOUR_CLIENT_APP_SECRET` exactly as before.

For now, we're hardcoding seeding our client app into our authorisation server's database. Later you might want to refactor this out.

Finally, we'll need to run migrations on our database to get the `OpenIddict` tables in there.

Open up a command prompt at your authorisation server's project folder and run the following command:

`dnx ef migrations add OpenIddict`

You will see this error, we'll deal with this in a moment:

![2015-12-20 16_04_58-C__Windows_System32_cmd.exe.png]({{site.baseurl}}/media/2015-12-20 16_04_58-C__Windows_System32_cmd.exe.png)

You should now see a new file in your solution explorer:

![2015-12-20 16_05_32-Photos.png]({{site.baseurl}}/media/2015-12-20 16_05_32-Photos.png)

We need to fix up the auto-generated migration a little.

There will be lots of entries like this:

{% highlight c# %}
migrationBuilder.DropForeignKey(
    name: "FK_AspNetUserRoles_AspNetUsers_UserId",
    table: "AspNetUserRoles");
{% endhighlight %}

Delete from this migration's `Up(..)` and `Down(..)` methods any entries that reference any `AspNetXXX` tables.

You should end up with a migration class that looks more or less like this, but it might vary as `OpenIddict` changes:

{% highlight c# %}
public partial class OpenIddict : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Application",
            columns: table => new
            {
                Id = table.Column<string>(nullable: false),
                DisplayName = table.Column<string>(nullable: true),
                LogoutRedirectUri = table.Column<string>(nullable: true),
                RedirectUri = table.Column<string>(nullable: true),
                Secret = table.Column<string>(nullable: true),
                Type = table.Column<string>(nullable: true)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Application", x => x.Id);
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(
            name: "Application");
    }
}
{% endhighlight %}

Now *be sure to recompile your whole project before the next step*, because we're going to run the migrations, but the migration running code doesn't compile your project for you, so if you forget you'll be using the old migrations.

Now, go back to command prompt at your authorisation server's project folder and run the following command:

`dnx ef database update`

You should see something like this:

![2015-12-20 16_13_20-C__Windows_System32_cmd.exe.png]({{site.baseurl}}/media/2015-12-20 16_13_20-C__Windows_System32_cmd.exe.png)

Now go back to Visual Studio, hit `Ctrl+F5` and your authorisation server should run just like before:

![2015-12-20 16_14_36-Home Page - AuthorisationServer ‎- Microsoft Edge.png]({{site.baseurl}}/media/2015-12-20 16_14_36-Home Page - AuthorisationServer ‎- Microsoft Edge.png)
