---
layout: post
title: "NuGet, Package Restore, Target Files, and Subversion"
date: 2013-06-03
modified:
categories: blog
excerpt:
tags: [.net, asp.net-mvc-4, nuget]
image:
  feature:
share: true
comments: true
---
I love [NuGet](https://nuget.codeplex.com/). In fact, it wasn’t until NuGet existed that I actually really started to enjoy using Visual Studio. Yeah, in the past, I would use Visual Studio because it was still the best thing out there for my IDE, but NuGet opened up a whole new world of possibilities and made it significantly easier to manage all those pesky libraries that come with any good-sized development project (particularly where there are dependencies). (Don’t worry *nix enthusiasts. I love Gentoo and Mint Linux, so I am well aware that Microsoft was not the first to come up with the package management system. However, what they and the community has developed is done very well.)

Now, with Microsoft’s recent push to make all the parts of Visual Studio and the .NET platform installable via NuGet, it’s created a little bit of a package management mess. For example, in a new ASP.NET MVC 4 project, there are roughly 10 NuGet packages already installed without me doing any additional work. Throw in a bit of OAuth 2 and a couple other items, and I’m up to 15 or 20. While it is a bit of a pain to manage (dependency trees would be nice to see).

As more packages become part of the system, it can sometimes be a bit trickier to manage those packages, and sometimes some interesting issues can crop up. I ran into a bit of an issue last week when I did my most recent upgrade of packages. The package that specifically caused me problems was the [Microsoft.Bcl.Async NuGet package](https://nuget.org/packages/Microsoft.Bcl.Async). This package specifically targets .NET Framework 4 to use the new [async and await](http://msdn.microsoft.com/en-us/library/vstudio/hh191443.aspx) keywords (it is available by default in .NET 4.5 and on). Now, as it turns out, I ultimately didn’t need this package as I was working towards upgraded to .NET 4.5 anyway, but the problem that I got may still be a problem for you, so I wanted to share my solution.

My team uses a Jenkins continuous integration server and a farm with MSBuild in order to build our .NET applications. After I performed this upgrade (everything built fine locally) and pushed to the central repository (DANGER WILL ROBINSON! EMAILS TO THE TEAM THAT I BROKE THE BUILD!), I started investigating my problem. Here is the error I was receiving:

> The imported project `\packages\Microsoft.Bcl.Build.\tools\Microsoft.Bcl.Build.targets`
> was not found. Confirm that the path in the declaration is correct, and that
> the file exists on disk.

Well, the error was pretty straightforward. I just needed to get the Microsoft.Bcl.Build.targets file on my build server. Of course, my build server always performs a clean build, so even if I put the file there, it would get wiped the next time around. Besides, this just felt like an unacceptable solution.

So, I started doing some research. It turns out that the [answer lies here](http://webcache.googleusercontent.com/search?q=cache:wqMaaLGKd3UJ:blogs.msdn.com/b/bclteam/p/asynctargetingpackkb.aspx+&cd=2&hl=en&ct=clnk&gl=us) (linking to a cached version as MSDN Blogs appear to be having problems tonight). It seems that NuGet’s Package Restore system cannot handle injecting MSBuild targets files (such as the Microsoft.Bcl.Build.targets file that was causing me problems). Normally, the Package Restore system is the system that downloads any packages from NuGet if they are not already part of your project. However, when this injection must occur, it can’t do that.

The only solution is to check the targets file into source control so my continuous integration system can find it. That perturbed me because I had already ignored the entire packages folder in my solution because of the fact that NuGet should be able to restore any of those packages. So, I only wanted any *.targets file to be checked in in the future (so I wouldn’t have this problem again), but NOT check in anything else. Unfortunately, Subversion does not have the ability to do exclusive matching (e.g. allow all *.targets and ignore everything else). But, you can fool it. Here’s the svn:ignore pattern I applied to my packages directory (incoming regular expression!):

{% highlight text %}
*[!t][!a][!r][!g][!e][!t][!s]
*.targets?*
{% endhighlight %}

I set that pattern to svn:ignore, checked the targets file in, and everything worked like a charm.
