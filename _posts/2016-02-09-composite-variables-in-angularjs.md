---
layout: blog
category: blog
splash: ""
tags: null
published: false
title: Composite variables in AngularJS
---

In my view I have something like this:

    <div ng-show="!isNew && !uploading">...</div>

And then much later on in the same view:

    <a href="..." ng-show="!isNew && !uploading">...</a>

The logic for whether or not to show these two fields is a composite of two scope variables. They are too far apart in the view to wrap in a parent that encapsulates the logic.

My two options I can think of that smell:

- I *could* watch those two variables in my scope using `$scope.$watch(..)` and update a composite scope variable with the result, but... it smells to me, when in the view it is as simple as it is for one
- I *could* change my controller code itself to always update the composite variable, but this smells too, maybe wrongly, but because it introduces multiple points of error if I forget to update the composite in the write place. Plus, I want to keep my controller variables entirely functional, and my UI make decisions on what to do with the functional values it gets

I was hoping I could, in my view, do something like this:

{{shouldShow = !isNew && !uploading}}