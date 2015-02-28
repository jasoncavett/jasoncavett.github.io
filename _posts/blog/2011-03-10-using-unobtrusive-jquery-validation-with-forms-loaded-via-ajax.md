---
layout: post
title: "Using Unobtrusive jQuery Validation with Forms Loaded via AJAX"
date: 2011-03-10
modified:
categories: blog
excerpt:
tags: [asp.net-mvc-3, jquery]
image:
  feature:
share: true
comments: true
---
I wanted to share this quick, but simple, tip on how to use unobtrusive jQuery validation with ASP.NET MVC 3 and forms loaded via AJAX. It’s easy to do, but it’s not entirely intuitive without looking into jquery.validation.js.

All the steps for setting up the validation are the same. I’ll reiterate them here for consistency.

First, make sure the global web.config file has the following settings configured. (This is assuming, of course, that you want validation enabled globally. If you do not, you can call Html.EnableClientValidation() and Html.EnableUnobtrusiveJavaScript() in the specific code that you care about.)

{% highlight xml %}
<appSettings>
    <add key="ClientValidationEnabled" value="true" />
    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
</appSettings>
{% endhighlight %}

You must also include the correct scripts. Note that, at this time, jquery.validate works only with jQuery 1.4.4. ([Nick Craver](http://twitter.com/Nick_Craver) has pointed me to [a version that works with jQuery 1.5](https://github.com/jzaefferer/jquery-validation) but has not yet been officially released.)

{% highlight html %}
<script type="text/javascript" src="@Url.Content("></script>
<script type="text/javascript" src="@Url.Content("></script>
<script type="text/javascript" src="@Url.Content("></script>
<script type="text/javascript" src="@Url.Content("></script>
{% endhighlight %}

The last bit of setup occurs in the HTML. You need a place to put your dynamic form as well as a wrapper around the entire content so you can parse the form later for validation.

{% highlight html %}
<!-- Some HTML Content .. -->
<div id="validation">
    <div id="ajaxform"></div>
</div>
<!-- Some more HTML Content .. -->
{% endhighlight %}

That’s all the setup that is required. Now, go to the location where you are loading a view (a partial view in my specific case) via AJAX. In my particular case, I was loading a partial view based on what node was selected in [jsTree](http://www.jstree.com/). In either case, I am using jQuery’s [$.ajax function](http://api.jquery.com/jQuery.ajax/). Here is the function call:

{% highlight javascript %}
$.ajax({
    type: 'GET',
    url: '/Home/TreeNode',
    data: {
        id: id
    },
    success: function(data) {
        $('#ajaxform').html(data);
        $.validator.unobtrusive.parse($("#validation"));
    }
});
{% endhighlight %}

Without going into too much detail (please see the above link for more information about the `$.ajax` function), I am basically doing a GET on the URL /Home/TreeNode and passing in the parameter ‘id’ (which is what I use to look up the data in a simple database).

The interesting part of this code occurs in the success function. There are two things happening here. First, I am placing the data I receive (which is my partial view) into #ajaxcontent. That is simple enough. The second part, and focus of this post, is highlighted at line 9 in the code above. This line is called to give jquery.validate the ability to respond to changes in the form that was loaded via AJAX. Note that any form loaded normally (on the initial page load) will work automatically. You have to call the above line only if you are loading a form after the fact.

That’s all there really is to it! Very easy to do although, as I said previously, not entirely obvious at first glance.
