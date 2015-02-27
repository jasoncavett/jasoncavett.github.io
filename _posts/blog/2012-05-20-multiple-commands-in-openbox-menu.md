---
layout: post
title: "Multiple Commands in Openbox Menu"
date: 2012-05-20
modified:
categories: blog
excerpt: "Make Openbox your to command!"
tags: [linux, openbox]
image:
  feature:
share: true
comments: true
---
For those of you who love using a very light window manager (either because you are running on very old hardware, or, like me, you just like things lean and mean), [Openbox](http://openbox.org/) is an excellent choice. It has a ton of configuration options and is extremely lightweight.

Almost all of Openbox’s configuration happens in to XML-based files – rc.xml and menu.xml. menu.xml is for configuring the lightweight, Openbox menu (which you get to by right clicking on the background within Openbox). rc.xml is pretty much there for configuring everything else within Openbox (keyboard commands, number of desktops, etc).

While these files are very easy to configure – just fire up your favorite editor – it is not always intuitive of how to accomplish certain tasks. For example, I wanted to run two commands, one right after the other, with just a single menu selection. In case you are wondering the same thing, here is how you do that:

{% highlight xml %}
<item label="Suspend">
    <action name="Execute">
        <execute>sh -c 'xscreensaver-command --lock; sudo pm-suspend'</execute>
    </action>
</item>
{% endhighlight %}

Please note (as pointed out by reader, Jabril), you have to add the ampersand character after all the preceding commands so that they will run in parallel and allow the subsequent commands to run. Those of you familiar with the command line are accustomed to this idea. (Thanks for the catch, Jabril!)
