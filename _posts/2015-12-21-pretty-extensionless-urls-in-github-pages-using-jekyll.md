---
layout: blog
category: blog
splash: ""
tags: 
  - "null"
published: true
title: "Pretty, extensionless URLs in GitHub Pages using Jekyll"
---



If you want to have extensionless URLs in GitHub Pages then you might be inclined to add a permalink to the metadata of your page or post:

    permalink: /my-pretty-url
    
Short answer for TL;DR, you need to put a forward slash at the end of your pretty permalink:

    permalink: /my-pretty-url/
    
If you don't do this, it will *kind of* work and you'll still get the expected URL:

![2015-12-21 15_19_24-the overengineer.png]({{site.baseurl}}/media/2015-12-21 15_19_24-the overengineer.png)

However you may well end up with your browser simply prompting you to download the page rather than actually navigate to it:

![2015-12-21 15_17_32-Cortana.png]({{site.baseurl}}/media/2015-12-21 15_17_32-Cortana.png)

The is simply down to how `Jekyll` generates pages. Right now you will end up with an extensionless file generated for your page:

![2015-12-21 15_22_04-_site.png]({{site.baseurl}}/media/2015-12-21 15_22_04-_site.png)

What you want is a folder with the pretty name containing your page in an `index.html`, like below:

![2015-12-21 15_24_01-my-pretty-url.png]({{site.baseurl}}/media/2015-12-21 15_24_01-my-pretty-url.png)

If you put a forward slash on the end of your `permalink` pretty URL, you force `Jekyll` to do just this:

    permalink: /my-pretty-url/

Enjoy!
