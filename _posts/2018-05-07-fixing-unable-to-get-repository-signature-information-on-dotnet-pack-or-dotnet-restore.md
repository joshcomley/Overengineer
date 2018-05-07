---
layout: blog
category: blog
published: false
title: >-
  Fixing "Unable to get repository signature information" on "dotnet pack" or
  "dotnet restore"
splash: ''
tags: ''
---
I ran into an odd problem today whereby all of a sudden the `dotnet restore` command (also a problem if you call `dotnet pack`) failed with the following errors:

	C:\Program Files\dotnet\sdk\2.1.300-preview2-008533\NuGet.targets(114,5): error : Unable to get repository signature information for source https://www.myget.org/F/[blahblah]/api/v3/repository-signatures/index.json. [D:\...blah.sln]
	C:\Program Files\dotnet\sdk\2.1.300-preview2-008533\NuGet.targets(114,5): error :   Unable to parse allRepositorySigned information from https://www.myget.org/F/[blahblah]/api/v3/index.json. [D:\...blah.sln]

I tried flushing dns cache, resetting network adapters, clearing `NuGet` cache, uninstalling and reinstalling `NuGet`, everything.

This is what worked:

I was calling `dotnet restore`/`dotnet pack` on the solution file. I ran `dotnet pack` on each individual project in the solution from the command line, now it works on the solution file. Go figure.

You probably don't need to run it on every project, I imagine one will do, but I'm unable to test that as now everything is working. I have confirmed it is working by clearing my `NuGet` cache again, cloning a new version of my repository and running `dotnet pack`, and the restore runs fine once again.

As an aside, pinging `www.myget.org` on the problem machine returns `myget-www-prod-eu-west.cloudapp.net`. Not sure if this has anything to do with it (some cross domain restriction, perhaps?). But then again, now pack works and I can see in the logs the restore took place with `dotnet `