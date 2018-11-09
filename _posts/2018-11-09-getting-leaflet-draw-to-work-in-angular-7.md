---
layout: blog
category: blog
published: false
title: Getting leaflet-draw to work in Angular 7
---
## Issue

The code works fine in development builds, but when published (in my case with AOT, I didn't test without), I am getting `l.Control.Draw is not a constructor`.

## Solution

Be prepared. This is uglier than Ted Cruz and Papa John's bastard baby.

# First
Import `leaflet` and `leaflet-draw` into your equivalent of `vendors.ts`:

    import 'leaflet';
    import 'leaflet-draw';
    
# Second
*Wherever* you have used these imports in your code, replace it with:

    declare const L: any;
    
If you encounter build errors, you can use `@ts-ignore`:

	// @ts-ignore
	drawControl: L.Control.Draw;

Et voila.

F***ing stupid.