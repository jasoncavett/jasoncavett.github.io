---
layout: post
title: "WebGL Support in Google Chrome in Linux"
date: 2012-08-28
modified:
categories: blog
excerpt: "Because Windows Shouldn't Have All the Fun"
tags: [chrome, google, linux, webgl]
image:
  feature:
share: true
comments: false
---
As any good computer nerd knows, the browser of choice these days is Google Chrome. It’s fast. Chrome supports a ton of the latest not-yet-standard standards (AKA HTML5 and CSS3). It has an [excellent extension environment](https://chrome.google.com/webstore/category/home). And, it has a great set of built-in tools to support development. And, last of all, it supports WebGL…with a caveat or two.

That brings me to the reason for my post.

I have been recently doing work with GPU rendering in the web browser. Most of the work has been happening on my Windows 7 box (soon to be Windows 8!). But, because my Linux machine is much more powerful, I wanted to move the development to there. So, I did.

And nothing worked.

I quickly learned about [chrome://gpu](chrome://gpu/), and after visiting there, I saw this screen:

{:.center}
![GPU Settings in Chrome]({{ site.url }}/images/gpu-ss.png)

Hmmm…

So, I did a bit of poking around in Chrome’s options, and a little bit of searching around on the web, and I found the exact two commands I was looking for to bypass this problem.

**Note: This solution will only work if your graphics card is well supported and you have your drivers properly setup.** If you have questions on how to setup OpenGL or proprietary drivers for your card, I would start with a Google search for your graphics card (and, perhaps, distribution).

In order to use WebGL in Linux, run your web browser with this command:

```
x-www-browser --enable-webgl --ignore-gpu-blacklist
```

Note that you may need to replace x-www-browser with the path to Chrome. (Make sure that you shut down Chrome completely before running that command so you start a new browser session.) Visit [chrome://gpu](chrome://gpu/) to make sure everything worked for you.

No, go enjoy [WebGL goodness](http://www.chromeexperiments.com/webgl)!

**Follow-up**
I did want to write a follow up based on the discussion that happened after I [shared blog post](https://plus.google.com/103414471305322507429/posts/fScQJP84nvf) to my Google+ profile. [Ryan Sleevi](https://plus.google.com/105761279104103278252/posts), a Google employee and Chrome developer provided this warning:

> Normally the blacklist exists because of known (functional or security) bugs in drivers
> that make in (unsafe or unstable) to use WebGL. While it may work, I might advise and
> advocate caution before using that latter flag too widely.
> While a lot of work has gone in to sandboxing, securing, and sanitizing the GPU process,
> one buffer underrun can be your undoing.

You now have been warned. Please keep this in mind.
