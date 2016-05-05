---
layout: blog
category: blog
splash: ""
tags: ""
published: false
title: Fixing nested file names with the node_modules folder in Visual Studio
---
If you get this:

`The specified path, file name, or both are too long. The fully qualified file name must be less than 260 characters, and the directory name must be less than 248 characters.`

Then I recommend following these steps:

## Ensure you have the latest node.js

<https://nodejs.org/en/download/>

## Ensure you have the 

 Tools -> Options -> Project and Solutions -> External Web Tools -> move $(PATH) up on level so it is over $(DevEnvDir)
 
 ![2016-05-05 14_45_54-Options.png]({{site.baseurl}}/media/2016-05-05 14_45_54-Options.png)

Run this:

npm install -g npm