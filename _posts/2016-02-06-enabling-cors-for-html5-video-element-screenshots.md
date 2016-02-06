---
layout: blog
category: blog
splash: ""
tags: 
  - "null"
published: true
title: Enabling CORS for HTML5 video element screenshots
---


Or:

*Preventing **Uncaught SecurityError: Failed to execute 'toDataURL' on 'HTMLCanvasElement': Tainted canvases may not be exported.***

<h1>Scenario</h1>
You are trying to take a screenshot of a HTML5 video element, from a video that is on a different domain, and you are getting a security error similar to the above.

For example:<br/>
Your video playing client is here:<br/>
<http://www.myapp.com/player.html><br/>
Your video is being served from here:<br/>
<http://cdn.myapp.com/video.mp4>

<h1>The solution</h1>
Run through the following check-list to ensure you have absolutely everything configured correctly.

<h1>Chrome</h1>
This seems to be just in Chrome, but loading your HTML5 video will fail *if* you have `crossOrigin` set, but your video and your client are on a different *port* **and** you are **not** on `https`:

**THIS WILL FAIL**<br/>
Client at **http://www.myapp.com/player.html**:

    <video crossOrigin="anonymous" src="http://cdn.myapp.com:81/video.mp4"></video>

**SUCCESS**<br/>
Client at **http://www.myapp.com/player.html**:

    <video crossOrigin="anonymous" src="https://cdn.myapp.com:81/video.mp4"></video>

(the `src` is set to `https` in the successful example)

<h1>Timing</h1>
`getImageData()` and `toDataURL()` will be security blocked *unless* **crossorigin** is set to `anonymous` or `use-credentials` ([as defined here][1]) **before** the video is loaded. You can achieve this either by adding `crossOrigin` directly to the HTML element, or ensuring that javascript sets it **before** the `src` is loaded:

**THIS WILL FAIL**<br/>
{% highlight javascript %}
    vid.src = "...";
    vid.load();
    vid.crossOrigin = "anonymous";
{% endhighlight %}

**THIS WILL SUCCEED**<br/>
{% highlight javascript %}
    vid.src = "...";
    vid.load();
    vid.crossOrigin = "anonymous";
{% endhighlight %}

**THIS WILL SUCCEED**<br/>
{% highlight html %}
    <video crossOrigin="anonymous" controls></video>
{% endhighlight %}
{% highlight javascript %}
    vid.src = "...";
    vid.load();
{% endhighlight %}

<h1>Spelling</h1>
If you are going to set `crossOrigin` in javascript, be sure to use the correct casing for the javascript property: `crossOrigin` (NOT `crossorigin`)
  [1]: https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_settings_attributes

**THIS WILL FAIL**<br/>
{% highlight javascript %}
    vid.crossorigin = "anonymous";
    vid.src = "...";
    vid.load();
{% endhighlight %}

**THIS WILL SUCCEED**<br/>
{% highlight javascript %}
    vid.crossOrigin = "anonymous";
    vid.src = "...";
    vid.load();
{% endhighlight %}

<h1>Serving of data</h1>
You will need your CDN to deliver your video with the `Access-Control-Allow-Origin` set to something useful for you. For more details on this header [see here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS), but to test you can use `*` as a value (highly dangerous, not recommended for production) otherwise you must specify the domain of your client, in the examples case above the header would look like:

`Access-Control-Allow-Origin:http://www.myapp.com`

<h1>Everything should now work</h1>
Providing you have met all of these requirements, you should be good to go. Happy CORSing!
