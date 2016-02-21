---
layout: blog
category: blog
splash: ""
tags: null
published: false
title: Moving to ASP.NET Core
---

Add:
    "Microsoft.AspNetCore.Hosting": "1.0.0-*",
    
    #Startup.cs
    
         public static void Main(string[] args)
        {
            var application = new WebHostBuilder()
                .UseCaptureStartupErrors(true)
                .UseDefaultConfiguration(args)
                .UseIISPlatformHandlerUrl()
                .UseServer("Microsoft.AspNetCore.Server.Kestrel")
                .UseStartup<Startup>()
                .Build();

            application.Run();
        }

#\_ViewImports.cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers