---
layout: blog
category: blog
splash: ""
tags: null
published: false
title: vNext cheat sheet
---

(this blog post will be added to over time, and is initially for my own use as a cheat sheet to remember all the new vNext details, because, well, the only thing that hasn't changed is the colour of Visual Studio)

# DNVM
## Dot Net Version Manager
<https://github.com/aspnet/Home/wiki/Version-Manager>
### Commands
    // Upgrades your installed .NET runtime version to the latest STABLE
    dnvm upgrade
    
    // Upgrades your installed .NET runtime version to the latest UNSTABLE
    dnvm upgrade -u
    
    // Upgrades your installed .NET core runtime version to the latest STABLE
    dnvm upgrade -r coreclr
    
    // Upgrades your installed .NET core runtime version to the latest UNSTABLE
    dnvm upgrade -r coreclr -u
