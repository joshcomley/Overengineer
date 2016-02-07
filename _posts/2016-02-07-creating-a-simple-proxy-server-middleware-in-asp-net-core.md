---
layout: blog
category: blog
splash: ""
tags: 
  - "null"
published: true
title: Creating a simple proxy server middleware in ASP.NET Core
---


Sometimes it is useful, especially for myself under certain development environment conditions, to use a proxy server.

Below is an implementation you can adapt to your needs.

Suppose we have a file at **http://cdn.myapp.com/movie.mp4**.

Suppose we have a web app at **http://www.myapp.com/**.

Suppose we want to have the **web app** serve a file from the CDN.

In this case, this middleware proxy would work by making a request to:

**http://www.myapp.com/proxy?url=http://cdn.myapp.com/movie.mp4**

In my case this was to bypass security restrictions that were only applicable in development environments. As such, this proxy has not been tested thoroughly for performance etc., as I will never use it in production. Feel free to contribute suggestions in the comments.

This is very open ended, and could leave your server open to easy DDOS attacks by pointing the proxy endpoint to a huge file anywhere on the internet. If you were to use this in production, the first thing I would recommend would be to change the proxy endpoint to work like this:

**http://www.myapp.com/proxy/movie.mp4**

The proxy code itself would then contain the logic to convert the above web app URL to the correct CDN one, rather than just taking absolutely any old URL.

Here is the code.

# project.json

This middleware uses `HttpClient` that is available in `System.Net.Http`, so add this line to your dependencies in your `project.json`:

{% highlight json %}
  "dependencies": {
    "System.Net.Http": "4.0.1-*"
  },
{% endhighlight %}

# Middleware class itself

{% highlight c# %}
using System;
using System.IO;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.AspNet.Http;

namespace YourWebApp.Middleware.Proxy
{
    public class ProxyServerMiddleware
    {
        private readonly RequestDelegate _next;

        public ProxyServerMiddleware(RequestDelegate next)
        {
            _next = next;
        }

        public async Task Invoke(HttpContext context)
        {
            var endRequest = false;
            if (context.Request.Path.Value.Equals("/proxy", StringComparison.OrdinalIgnoreCase))
            {
                const string key = "url";
                if (context.Request.Query.ContainsKey(key))
                {
                    var url = context.Request.Query[key][0];
                    await StreamAsync(context, url);
                    endRequest = true;
                }
            }
            if (!endRequest)
            {
                await _next(context);
            }
        }

        private static async Task StreamAsync(HttpContext context, string url)
        {
            var httpClientHandler = new HttpClientHandler
            {
                AllowAutoRedirect = false
            };
            var webRequest = new HttpClient(httpClientHandler);

            var buffer = new byte[4*1024];
            var localResponse = context.Response;
            try
            {
                using (var remoteStream = await webRequest.GetStreamAsync(url))
                {
                    var bytesRead = remoteStream.Read(buffer, 0, buffer.Length);

                    localResponse.Clear();
                    localResponse.ContentType = "application/octet-stream";
                    var fileName = Path.GetFileName(url);
                    localResponse.Headers.Add("Content-Disposition", "attachment; filename=" + fileName);

                    if (remoteStream.Length != -1)
                        localResponse.ContentLength = remoteStream.Length;

                    while (bytesRead > 0) // && localResponse.IsClientConnected)
                    {
                        await localResponse.Body.WriteAsync(buffer, 0, bytesRead);
                        bytesRead = remoteStream.Read(buffer, 0, buffer.Length);
                    }
                }
            }
            catch (Exception e)
            {
                // Do some logging here
            }
        }
    }
}
{% endhighlight %}

# Extension method for use in `Startup.cs`

{% highlight c# %}
using Microsoft.AspNet.Builder;

namespace AngularOidcClient.Middleware.Proxy
{
    public static class ProxyServerMiddlewareExtension
    {
        public static IApplicationBuilder UseProxyServer(this IApplicationBuilder builder)
        {
            return builder.Use(next => new ProxyServerMiddleware(next).Invoke);
        }
    }
}
{% endhighlight %}

# Startup.cs

{% highlight c# %}
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        // Add this line to the top of your Configure method
        app.UseProxyServer();
        ...
    }
{% endhighlight %}
