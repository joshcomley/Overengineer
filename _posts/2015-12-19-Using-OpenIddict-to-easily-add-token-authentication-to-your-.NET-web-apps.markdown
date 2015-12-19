---
title: Using OpenIddict to easily add token authentication to your .NET web apps
date: 2015-12-19T00:00:00.000Z
published: true
---



You'll find this post in your `_posts` directory - edit this post and re-build (or run with the `-w` switch) to see your changes!
To add new posts, simply add a file in the `_posts` directory that follows the convention: YYYY-MM-DD-name-of-post.ext.

Jekyll also offers powerful support for code snippets

    {% highlight ruby %}
    // POST api/Account/RegisterExternalToken
    [OverrideAuthentication]
    [AllowAnonymous]
    [Route("RegisterExternalToken")]
    public async Task<IHttpActionResult> RegisterExternalToken(RegisterExternalTokenBindingModel model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
    
        var externalLogin = await Brandless.OAuth.Tokens.ExternalLoginData.FromToken(
            model.Provider, 
            model.Token);
    
        if (externalLogin == null)
        {
            return InternalServerError();
        }
    
        if (externalLogin.LoginProvider != model.Provider)
        {
            Authentication.SignOut(DefaultAuthenticationTypes.ExternalCookie);
            return InternalServerError();
        }
    
        var user = await UserManager.FindAsync(new UserLoginInfo(externalLogin.LoginProvider,
            externalLogin.ProviderKey));
    
        var hasRegistered = user != null;
        ClaimsIdentity identity;
    
        if (hasRegistered)
        {
            identity = await UserManager.CreateIdentityAsync(user, OAuthDefaults.AuthenticationType);
            IEnumerable<Claim> claims = externalLogin.GetClaims();
            identity.AddClaims(claims);
            Authentication.SignIn(identity);
        }
        else
        {
            user = new ApplicationUser
            {
                Id = Guid.NewGuid().ToString(),
                UserName = model.Email,
                Email = model.Email
            };
    
            var result = await UserManager.CreateAsync(user);
            if (!result.Succeeded)
            {
                return GetErrorResult(result);
            }
    
            var info = new ExternalLoginInfo
            {
                DefaultUserName = model.Email,
                Login = new UserLoginInfo(model.Provider, externalLogin.ProviderKey)
            };
    
            result = await UserManager.AddLoginAsync(user.Id, info.Login);
            if (!result.Succeeded)
            {
                return GetErrorResult(result);
            }
    
            identity = await UserManager.CreateIdentityAsync(user, OAuthDefaults.AuthenticationType);
            IEnumerable<Claim> claims = externalLogin.GetClaims();
            identity.AddClaims(claims);
            Authentication.SignIn(identity);
        }
    
        var ticket = new AuthenticationTicket(identity, new AuthenticationProperties());
        var currentUtc = new SystemClock().UtcNow;
        ticket.Properties.IssuedUtc = currentUtc;
        ticket.Properties.ExpiresUtc = currentUtc.Add(TimeSpan.FromDays(365));
        var accessToken = Startup.OAuthOptions.AccessTokenFormat.Protect(ticket);
        Request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
    
        // Create the response building a JSON object that mimics exactly the one issued by the default /Token endpoint
        var token = new JObject(
            new JProperty("userName", user.UserName),
            new JProperty("id", user.Id),
            new JProperty("access_token", accessToken),
            new JProperty("token_type", "bearer"),
            new JProperty("expires_in", TimeSpan.FromDays(365).TotalSeconds.ToString()),
            new JProperty(".issued", currentUtc.ToString("ddd, dd MMM yyyy HH':'mm':'ss 'GMT'")),
            new JProperty(".expires", currentUtc.Add(TimeSpan.FromDays(365)).ToString("ddd, dd MMM yyyy HH:mm:ss 'GMT'"))
        );
        return Ok(token);
    }
    {% endhighlight %}

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
