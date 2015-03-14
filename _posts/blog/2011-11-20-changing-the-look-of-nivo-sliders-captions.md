---
layout: post
title: "Changing the Look of Nivo Slider’s Captions"
date: 2011-11-20
modified:
categories: blog
excerpt:
tags: [css, jquery]
image:
  feature:
share: true
comments: false
---
[Nivo Slider](http://nivo.dev7studios.com/) brands itself “The World’s Most Awesome jQuery Image Slider” and, while I haven’t tried **all** the sliders out there, I certainly concede that it is very nice and well done. Following the instructions made it very easy to get it up and running (with a little bit of Googling to solve a stuttering error that I saw inside Google Chrome).

I was using Nivo to move through images at the top of a page I was designing. Each image displayed a product or capability that my client wanted to offer to his customers. When I showed a demo of a site I was working on to the client, he decided that he wanted to see a banner associated with each image. That way, his customers would be able to easily know what the product was that they were being offered (and of course, being able to click on the image to follow to the product was necessary as well – but Nivo already offers that capability).

After first looking to make sure Nivo didn’t offer this type of capability out of the box (it didn’t…at least not directly), I then began looking at how I could use Nivo Slider’s already existing features and modify them a bit. The solution, as it turns out, was quite simple.

I started by downloading the Nivo default theme (this theme is provided with the default download). Follow the instructions here on [how to use a particular theme](http://nivo.dev7studios.com/support/advanced-tutorials/using-themes-with-the-nivo-slider/). Once the default theme was in place, I then began looking at Nivo’s captions as this seemed like a great location to provide a banner without having to actually edit the images or create and use an entirely different plug-in.

Nivo uses the CSS class `nivo-caption` to style its captions. With this in mind, it now becomes as simple as editing the CSS style to your respective look and feel.

If you are editing the default theme, you first need to remove these blocks of CSS:

{% highlight css %}
.theme-default .nivo-caption {
    font-family: Helvetica, Arial, sans-serif;
}
.theme-default .nivo-caption a {
    color:#fff;
    border-bottom:1px dotted #fff;
}
.theme-default .nivo-caption a:hover {
    color:#fff;
}
{% endhighlight %}

Now that you have removed that block, then go ahead and add the following block of CSS. I gleaned some insight from the Pascal theme (which did not directly meet my needs, but did help me solve this problem) that is provided with Nivo. Comments added to explain what you can change to modify the banner for yourself.

{% highlight css %}
.theme-default .nivo-caption {
    bottom:85%; /* The caption's distance up from the bottom of the image.*/
    left: 0px;  /* Setting left to 0px and right to auto pushes the caption to the left. Flip the values to send it back to the right. */
    right:auto;
    width:auto;
    max-width:630px; /* The maximum width of the banner. */
    overflow:hidden;
    background:#333; /* The background color. */
    color:#4c4b4b;
}
.theme-default .nivo-caption p
{
    font-family: Helvetica, Arial, Sans-Serif; /* The font of your caption. */
    padding: 5px 15px;
    color: #fff; /* The color of the font. */
    font-weight: bold;
    font-size: 27px;
    text-transform: uppercase;
}
{% endhighlight %}

Of course, go ahead and tweak this even further to increase the height of the banner (or shrink it), make it stretch across the entire image (it’s fairly straightforward to have it stretch across the top of the page), or anything else that you wish to have it do.
