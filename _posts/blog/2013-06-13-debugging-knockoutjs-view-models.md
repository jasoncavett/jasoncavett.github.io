---
layout: post
title: "Debugging Knockout View Models"
date: 2013-06-13
modified:
categories: blog
excerpt:
tags: [javascript, knockoutjs]
image:
  feature:
share: true
comments: true
---
As I have been doing a ton of Knockoutjs work lately, I figured I would share a useful tip that I learned for debugging your Knockoutjs view models. This will help you to see what data Knockout actually thinks its getting without you having to do a ton of work to browse through the structure via a debugger.

Simply add the following line to your HTML, load the page up, and…that’s it! Enjoy. More Knockoutjs goodness coming in the future.

{% highlight html %}
<pre data-bind="text: ko.toJSON($data, null, 2)"></pre>
{% endhighlight %}
