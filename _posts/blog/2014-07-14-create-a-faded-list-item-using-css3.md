---
layout: post
title: "Create a Faded List Item Using CSS3"
date: 2012-07-14
modified:
categories: blog
excerpt:
tags: [ccss, jquery]
image:
  feature:
share: true
comments: false
---
I know there are about 1000 tutorials on various styles that you can give your lists using CSS3 on the web…but I still wanted to add my own voice to the mix.

I wanted to create a CSS list that displayed the name of an item, and an associated value as a set of responses from my users. This is very simple without the styling. Here is my HTML.

{% highlight html %}
<ul id="responseItems">
    <li><a class="answerLink" href="/questions/1">ZIP Code</a><span class="answer">10001</span></li>
    <li><a class="answerLink" href="/questions/3">How Much Time</a><span class="answer">2 Hours</span></li>
    <li><a class="answerLink" href="/questions/4">Maximum Cost</a><span class="answer">$25.00</span></li>
</ul>
{% endhighlight %}

As I said, a very simple, unordered list. For the purposes of this post, I will assume it is sitting on a background with a hex value of `#424242`, but you can place this list wherever you want and on whatever color background you want (that is outside the scope of this post, however). Adjust the colors accordingly.

Now that the list is in place, it’s time to do just a quick styling with CSS3. I’m going to show the code first, and then describe what’s happening.

{% highlight css linenos %}
#responseItems li {
  margin-bottom: 3px;
}

#responseItems li
{
  background-color: #424242;

  /* Gradient for Fading */
  background-image: linear-gradient(left, #FF7F27 0%, #424242 67%);
  background-image: -webkit-linear-gradient(left, #FF7F27 0%, #424242 67%); /* Safari 5.1+, Mobile Safari, Chrome 10+ */
  background-image: -moz-linear-gradient(left, #FF7F27 0%, #424242 67%); /* Firefox 3.6+ */
  background-image: -ms-linear-gradient(left, #FF7F27 0%, #424242 67%); /* IE 10+ */
  background-image: -o-linear-gradient(left, #FF7F27 0%, #424242 67%); /* Opera 11.10+ */

  border-style: none;
  -moz-border-radius: 15px;
  -webkit-border-radius: 15px;
  border-radius: 15px;
  font-size: 16px;
  padding-left: 5px;
}

#responseItems a:link, #responseItems a:visited, #responseItems a:hover, #responseItems a:active
{
  color: black;
  text-decoration: none;
}
{% endhighlight %}

Most of the CSS is fairly straightforward. The part I want to point out starts on line 10. This effect basically takes the box that holds my list item and fades it from one color (`#FF7F27`) to the background color (`#424242`) coming from the left side. Unfortunately, every browser supports a different way of doing this (what a pain), which is why you see five different versions of this gradient.

Other than that, I also give the border a radius for browsers that support it starting at line 17. In the next rule, I also remove the text decoration and set the font color on the link itself. The final, resulting look is:

{:.center}
![Fading List]({{ site.url }}/images/fadinglist.png)

I would recommend playing around with the percentage of the fade (adjust the 67% value after the color). Please note that the rest of the styling is not part of this tutorial.

Also, there is an option to use transparencies within the linear-gradients (at least some of them). I had a little bit of trouble with it, but in case you wish to investigate further, you need to use rgba colors like this:

{% highlight css %}
background-image: -webkit-linear-gradient(left, rgba(255, 255, 255, 1) 0%, rbga(255, 255, 255, 0) 67%);
{% endhighlight %}

Hope this helps you out! Check out ImpressiveWeb’s [CSS3 linear gradients article](http://www.impressivewebs.com/css3-linear-gradient-syntax/) for more information on the topic.
