---
layout: post
title: "Dynamically Set Text Color On Different Colored Backgrounds"
date: 2015-09-10
modified:
categories: blog
excerpt:
tags: [javascript, accessibility, dynamic, colors, usability, ux]
image:
  feature:
share: true
comments: true
---
This code is not my own, rather, I discovered it while I was looking to figure out a way to make text dynamically adjust based on the background color they were sitting on.

This tip actually comes from [W3 Checkpoint 2.2 Ensure that foreground and background color combinations provide sufficient contrast when viewed by someone having color deficits or when viewed on a black and white screen](http://www.w3.org/TR/AERT#color-contrast).

In my particular case, I wanted to be able to dynamically adjust the text inside a Bootstrap navbar. Within the site I was working on, it is possible for users to choose a preference for the color of their navigation bar. I had to make sure that the text was still visible.

To do is fairly simple with a couple caveats. Here is the code to do it.

{% highlight javascript linenos %}
var rgb = $('.navbar').css('background-color').replace('rgb(', '').replace(')','' ).split(',').map(Number);
var o = Math.round(((rgb[0] * 299) + (rgb[1] * 587) + (rgb[2] * 114)) /1000);
if(o > 125) {
  $('.navbar .nav > li > a').css('color', 'black');
  $('.navbar-default .navbar-toggle .icon-bar').css('color', 'black');
} else {
  $('.navbar .nav > li > a').css('color', 'white');
  $('.navbar-default .navbar-toggle .icon-bar').css('color', 'white');
}
{% endhighlight %}

* **Line 1** - Simply, I am getting the current background color of the navbar. In order to use this particular technique, I need to convert the values I'm getting back to an RGB array with the 0, 1, and 2 positions corresponding to R, G, and B.
* **Line 2** - This is the formula that determines the color brightness of the color. This value is the perceived brightness of a color. The reason to get this number is that the ultimate choice of whether the text is white or black is determined by the difference between the text and the background color are greater than a set range.
* **Line 3 - 9** - Now that I have determined the brightness of the background, I now choose black or white for the text. Because of the Bootstrap navbar, I had to do a little bit more work to get the correct text change.

That's all there is to it! This is a nifty and easy way to make a site more usabile and accessible for your users.
