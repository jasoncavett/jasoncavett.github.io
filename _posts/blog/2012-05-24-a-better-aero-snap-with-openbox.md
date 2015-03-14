---
layout: post
title: "A Better Aero Snap with Openbox"
date: 2012-05-24
modified:
categories: blog
excerpt: "It's a snap!"
tags: [linux, openbox]
image:
  feature:
share: true
comments: false
---
While I love running a lightweight system, there are definitely some better UI “stuff” that Windows 7 has that my beloved Openbox setup does not – at least not out of the box. I was specifically looking for information on how to emulate Windows 7 [Aero Snap feature](http://windows.microsoft.com/en-us/windows7/products/features/snap) with the keyboard that automatically docks windows to the right and the left of my screen as I use this quite frequently when I’m developing. (Note: I didn’t worry about using the mouse to drag and drop snap windows as I really dislike lifting my hands up off the keyboard unless I absolutely have to. This also explains my love affair with VIM.)

If you visit [Openbox’s wiki page on the subject](https://wiki.archlinux.org/index.php/Openbox#Aero_snap_behaviour), they eventually send you to [this forum post](http://ubuntuforums.org/showthread.php?t=1796793). While the technique does work, it does not feel nearly as smooth as the Windows 7 version does. So, I decided to think about what was I actually trying to accomplish and found a different way to get the snap feature to work and it feels a lot smoother to me.

Open up `~/.config/openbox/rc.xml` in your favorite editor.

Add this keybinding to the keybind section:

{% highlight xml %}
<!-- Window Tiling: Emulates Windows 7 Snap feature -->
<keybind key="W-Left">
  <action name="UnmaximizeFull"/>
  <action name="MaximizeVert"/>
  <action name="MoveResizeTo">
    <width>50%</width>
  </action>
  <action name="MoveToEdgeWest"/>
</keybind>
<keybind key="W-Right">
  <action name="UnmaximizeFull"/>
  <action name="MaximizeVert"/>
  <action name="MoveResizeTo">
    <width>50%</width>
  </action>
  <action name="MoveToEdgeEast"/>
</keybind>
{% endhighlight %}

Save the file and reload Openbox.

Now, this will allow windows to be thrown to the left and the right using Window Key + Left/Right, respectively, and will maximize them and then resize them to 50% of your screen width. In addition, if you have multiple monitors, this will allow you to cross boundaries to the next monitor. There are, however, two, minor flaws with this approach. Here are the flaws and their fixes. (Please make sure you don’t bind the keys twice. Keybindings should be unique.)

First, there is no way to go back to the “regular” size of the window if it undocks from the edges. In Windows 7, if you are docked to the left and hit the “Windows + Right” key, you will return to the original window size in between docking it to the left. In the case of my solution, this will never happen. However, a default setting in rc.xml should already take care of this problem. If, at any time, you hit “Windows Key + Down,” your window should undock and return to it’s normal window size before it was maximized and resized to the right or left. I haven’t looked into whether keybindings allow for conditional statements (“if maximized, then just unmaximize”), but it doesn’t bother me so I’ll leave that as an exercise to you.

Note, the following XML is what you should already have in your rc.xml to configure your unmaximize a window.

{% highlight xml %}
<keybind key="W-Down">
    <action name="Unmaximize" />
</keybind>
{% endhighlight %}

The second issue that you may run into is that if you have other window open in between the window that is currently being thrown to the right or the left of your screen, it will first dock itself against those other windows (meaning you have to hit right or left more times to get it to the edge of your screen). This, by default, is a feature that is turned on in Openbox which is snap-to-windows. I will admit to not having figured out a fix for this, but, in all honesty, it’s never really bothered me.

That’s about it. Let me know if you have any suggestions for improving upon what I’ve done and I will happily update this post to reflect those changes.
