---
layout: blog
category: blog
splash: ""
tags: null
published: false
title: HTTP Error 500.19 with ASP.NET 5 after moving a project to a new folder
---

If you move your `ASP.NET 5` project to a new folder and run it, you might suddenly find yourself with a 500.19 error whilst IIS attempts to read your `web.config`. What I noticed was that the folder that IIS was looking in for my `web.config` was the old folder:

![2015-12-21 11_42_42-IIS 10.0 Detailed Error - 500.19 - Internal Server Error ‎- Microsoft Edge.png]({{site.baseurl}}/media/2015-12-21 11_42_42-IIS 10.0 Detailed Error - 500.19 - Internal Server Error ‎- Microsoft Edge.png)

To resolve this, I did two things.

# Kill IIS Express

![2015-12-21 11_44_11-Run.png]({{site.baseurl}}/media/2015-12-21 11_44_11-Run.png)

# Restart Visual Studio

After which, my project ran fine again and IIS found the new `web.config`. I guess it is a teething problem with the latest version of IIS Express.