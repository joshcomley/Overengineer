---
layout: blog
category: blog
splash: ""
tags: null
published: false
title: Preventing AngularJS from caching your templates and other files
---

In development it can be very useful to be able to ensure that your server files aren't cached.

Here is how to achieve this with `AngularJS` by using a request [interceptor](https://docs.angularjs.org/api/ng/service/$http#interceptors) to intercept requests to the server and inject a no-cache request parameter where required.

First of all, let's create our `AngularJS` interceptor:

{% highlight javascript %}
"use strict";

/**
 * Intercepts HTTP requests from Angular and applies a no-cache request parameter
 * to ensure the request will be unique
 */
angular.module("hazceptionApp")
    .factory("noCacheInterceptor", [
        "$log", "$injector", function ($log, $injector) {
            var noCacheFolders = ["views", "scripts", "lib"];

            var shouldApplyNoCache = function (config) {
                // The logic in here can be anything you like, filtering on
                // the HTTP method, the URL, even the request body etc.
                for (var i = 0; i < noCacheFolders.length; i++) {
                    var folder = noCacheFolders[i];
                    if (config.url.indexOf(folder + "/") === 0) {
                        return true;
                    }
                }
                return false;
            };

            var applyNoCache = function (config) {
                if (config.url.indexOf("?") !== -1) {
                    config.url += "&";
                } else {
                    config.url += "?";
                }
                config.url += "nocache=" + new Date().getTime();
            };

            var interceptor = {
                request: function (config) {
                    if (shouldApplyNoCache(config)) {
                        applyNoCache(config);
                    }
                    return config;
                }
            };

            return interceptor;
        }]);
{% endhighlight %}

This interceptor is very simple and can be easily adapted for your needs:

1 - Use `shouldApplyNoCache()` to check if this particular request should have caching eliminated
2 - If it does, call `applyNoCache()`
3 - ....that's it. But three steps is always nicer, right?

I suppose the third step is to add the interceptor to our app configuration:

{% highlight javascript %}
angular
    .module("yourApp")
    .config(function (..., $httpProvider) {
        // Make sure we only apply this during development
        // You can adjust this check as is relevant to your development environment
        if (window.location.host.indexOf("localhost") !== -1) {
            $httpProvider.interceptors.push("noCacheTemplateInterceptor");
        }
    });
{% endhighlight %}

**We really don't want to apply this interceptor in real-world usage, because it will dramatically increase the traffic on your server. Caching is there for a reason, we only want to override this on our dev boxes!**

Enjoy, folks!