---
layout: blog
category: blog
published: true
title: Unable to run Visual Studio installer or updates
---
Both my laptop and my PC suddenly started crashing when opening Visual Studio projects, and the Visual Studio installer would not properly run on either machine.

I was getting this error in my `ActivityLog.xml`:

    System.InvalidOperationException: Controller terminated before accepting connections. Exit code: -1073741819
    
Welp, it turns out it was having the `NODE_OPTIONS` environmnent variable set to `--max-old-space-size=4096`.

I removed this (System) environment variable and suddenly everything worked. Go figure.

More info here:

https://developercommunity.visualstudio.com/content/problem/287499/systeminvalidoperationexception-controller-termina.html

And apparently here:

https://github.com/electron/electron/issues/12695

Somebody shoot me.
