---
layout: blog
category: blog
splash: ""
tags: 
  - "null"
published: true
title: "Gotcha: Unable to load application or execute command 'YourProjectName'"
---


I was recently stuck with a problem whereby neither IIS Express nor Kestrel would load my project.

IIS Express would just hang on a spinning loading icon in Chrome, whereas `kestrel` would at least deliver me the following:

    Error: Unable to load application or execute command 'AuthorisationServer2'. Available commands: kestrel, web, ef.

# Short answer
Your project name in ASP.NET vNext is the *folder* name that contains your project files, *not* the project name of your Visual Studio project.

# Long answer
[Recent changes](https://github.com/aspnet/Announcements/issues/131) to ASP.NET 5 RC2 mean that you must specify the entry point assembly (the assembly that contains your static main method) in your `project.json`:

    "commands": {
      "web": "YourProjectName"
    },
    
The gotcha lies in the fact that unlike traditional project names, in ASP.NET vNext, the project name is abstracted from Visual Studio, and so it is *not* the name of the project itself as you see it in visual studio:

![2016-01-15 16_25_57-AuthorisationServer - Microsoft Visual Studio (Administrator).png]({{site.baseurl}}/media/2016-01-15 16_25_57-AuthorisationServer - Microsoft Visual Studio (Administrator).png)

It's this:

![2016-01-15 16_29_45-src.png]({{site.baseurl}}/media/2016-01-15 16_29_45-src.png)

Yep, it's the folder name. Make the `commands` > `web` entry in your `project.json` match the folder name and you'll be good to go.
