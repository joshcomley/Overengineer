---
layout: blog
category: blog
splash: ""
tags: 
  - "null"
published: true
title: "Prevent form validation on ng-click in AngularJS"
---


With the following HTML, validation on your form will be triggered every time you click on either button:

{% highlight html %}
<form>
    <input type="text" ng-model="video.Title" required />        
    <button ng-click="updateTitle()">Update</button>
    <button ng-click="alert('hi!')">Say hi</button>
</form>
{% endhighlight %}

But we don't necessarily want that to happen for our "Say hi" button. So how do we stop it?

Well, according to the [W3 specification on `button` elements](https://www.w3.org/TR/html401/interact/forms.html#h-17.5), if you don't specify a `type="[something]"` for your button, then it will default to `type="submit"`:

<img src="{{site.baseurl}}/media/2016-02-09 16_52_30-Forms in HTML documents.png" width="300" />
![2016-02-09 16_52_30-Forms in HTML documents.png]({{site.baseurl}}/media/2016-02-09 16_52_30-Forms in HTML documents.png)

In such a case, any click to this button, no matter what `ng-click` says, will submit the form. And submitting the form will trigger validation. The solution? Make sure it's type isn't `submit`:

{% highlight html %}
<form>
    <input type="text" ng-model="video.Title" required />        
    <button ng-click="updateTitle()">Update</button>
    <button type="button" ng-click="alert('hi!')">Say hi</button>
</form>
{% endhighlight %}

The form is no longer submitted on click, and we merrily say "hi!" to our users without a validation care in the world.

Happy coding!
