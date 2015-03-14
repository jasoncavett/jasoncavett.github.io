---
layout: post
title: "Don't. Trust. The Software."
date: 2014-10-22
modified:
categories: blog
excerpt: "As they say in the military, trust but verify."
tags: [software, microsoft]
image:
  feature:
share: true
comments: false
---
I recently made the mistake of trusting software.

No, this isn’t your typical “I accidentally installed a virus because I trusted
a really scammy site” kind of trust.

I trusted Microsoft.

More specifically, I trusted the [Microsoft Web Platform Installer](http://www.microsoft.com/web/downloads/platform.aspx).

Don’t get me wrong. Normally, I really like the Web Platform Installer. It is
the closest I have seen Windows ever get to a central software repository too
(ala Brew, Portage, Yum, etc.). It makes managing all the random software that
may get installed on a development machine, a deployment server, or whatever
else I may need.

However, today, that changed a little bit.

I was recently setting up Web Deploy for a continuous integration build. Just to
make sure I did it right, I pulled up the [official installation instructions
for IIS 8](http://www.iis.net/learn/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later).
You’ll note, if you follow the link, that the instructions specifically say to
install the Recommended Server Configuration for Web Hosting Providers from the
Web Deploy. So, that is what I did.

I then continued to follow the guide to configure Web Deploy so I could do the
deploy from a non-admin account on a remote machine. All was well and good.

Then, I attempted to setup a Publish Profile in Visual Studio 2012. When I
attempt to Validate the connection, I received a few different errors.
Fortunately, the errors are straightforward for the most part and I was able
to debug those (I won’t bore you with the details). I finally made it to this
error:

{% raw %}
2014-10-22 15:53:09 HEAD /msdeploy.axd site=Default%20Web%20Site 8172 - - - 404 7 0 1388
{% endraw %}

After doing a bit of research, I kept coming across that Web Deploy had not been
installed correctly. Of course (and please note my mistake, folks), I assumed
that the Web Platform Installer had done everything correctly. So, I spent the
next hour or so trying everything to get things to work.

Then, through a random (and, at this point, a frustrated) search, I ran across
[this post](http://serverfault.com/questions/557868/unable-to-use-web-deploy-on-windows-server-2012-http-error-404-7).

And I smacked myself in the forehead.

I headed on over to the [Web Deploy 3.5](http://www.iis.net/downloads/microsoft/web-deploy)
download page. I downloaded the actual installer and fired it up. It turns out
that Mr. Louros was 100% correct. When I fired up the installer, I saw this
screen:

{:.center}
![Web Deploy Installer]({{ site.url }}/images/web-deploy-installer.png)

It really was that simple. I added the additional items that had not been
installed. Restarted the service. It worked.

Long story short – don’t make assumptions. Even when the documentation says you
are doing the right thing. Even when the software has worked in the past. Even
when it’s very professional, well-written software. Check it out for yourself.
