---
layout: post
title: "Using the Url.Action Helper with JavaScript Variables"
date: 2012-01-20
modified:
categories: blog
excerpt:
tags: [asp.net-mvc-3, javascript, razor]
image:
  feature:
share: true
comments: true
---
Here’s a quick little tip. If you are using the ASP.NET Url.Action Helper, but trying to pass a JavaScript variable into it, you need to do a little bit of magic in order to get it to work.

Here is what you might try to do originally:

{% highlight javascript %}
function loadPageWithId(idValue) {
  $('#myDiv').load('@Url.Action("Action", "Controller",
                     new { id = idValue })');
}
{% endhighlight %}

Unfortunately, this won’t work because the variable, idValue, won’t evaluate due to the escape sequence. The example below provides the workaround.

{% highlight javascript %}
function loadPageWithId(idValue) {
  var link = '@Url.Action("Action", "Controller", new { id = "replace" })';
  link = link.replace("replace", idValue);
  $('#myDiv').load(link);
}
{% endhighlight %}

In this case, you actually replace the string “replace” with the value in the JavaScript variable which allows it to be evaluated.
