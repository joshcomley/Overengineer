---
layout: blog
category: blog
splash: ""
tags: 
  - "null"
published: true
title: "Fixing \"Unable to connect to web server 'IIS Express'.\""
---

Delete your web application's `.vs\applicationhost.config` and try again.

Then, in your project settings, change the `App URL` port to something new:

![2016-02-07 12_38_46-AuthorisationServer - Microsoft Visual Studio (Administrator).png]({{site.baseurl}}/media/2016-02-07 12_38_46-AuthorisationServer - Microsoft Visual Studio (Administrator).png)

Launch your app. You can change the `App URL` back again afterwards, this just ensures VS will regenerate your `applicationhost.config`.

Enjoy!