---
layout: blog
category: blog
splash: ""
tags: 
  - "null"
published: true
title: HTTP Error 500.19 with ASP.NET 5 after moving a project to a new folder
---


If you move your `ASP.NET 5` project to a new folder and run it, you might suddenly find yourself with a 500.19 error whilst IIS attempts to read your `web.config`. What I noticed was that the folder that IIS was looking in for my `web.config` was the old folder:

![2015-12-21 11_42_42-IIS 10.0 Detailed Error - 500.19 - Internal Server Error ‎- Microsoft Edge.png]({{site.baseurl}}/media/2015-12-21 11_42_42-IIS 10.0 Detailed Error - 500.19 - Internal Server Error ‎- Microsoft Edge.png)

To resolve this, just restart Visual Studio *even if you've only just opened Visual Studio fresh anyway*. Don't waste time fiddling around with a perfectly fine `web.config`.

I guess it is a teething problem with the latest version of IIS Express.
