---
layout: post
title: "Ninject, WebApi, and OWIN"
date: 2015-03-16
modified:
categories: blog
excerpt:
tags: [ninject, webapi, asp.net, owin]
image:
  feature:
share: true
comments: true
---
With all the updates [pouring out of Microsoft these days](http://www.asp.net/vnext/overview/aspnet-vnext/aspnet-5-overview) within .NET land, I have been checking out their latest offerings within some of my own projects.

So far, it has been a breeze to setup a [self-hosted application](https://msdn.microsoft.com/en-us/magazine/dn745865.aspx), and the fact that MVC no longer uses System.Web is a wonderful thing. Recently, I wanted to try out a new (to me) dependency injector. It is a sad thing to say, but, many projects I have worked have not used DI (which has led to a lot of workaround, hackarounds, etc.), so, while I understand the concepts, I haven't had the pleasure of using DI fully across a project. Now was my chance.

After doing a bit of researching, I settled on [Ninject](http://www.ninject.org/). I like the fact that it is lightweight, is very modular in structure, good documentation, and, of course, ninjas are always cool. (*Pro Tip*: If you ever play [Superfight](http://www.superfightgame.com/) with me, the Ninja will always win the fight. Hands down.)

As I said, it has excellent documentation. Because I am using OWIN, I followed [this set of instructions]https://github.com/ninject/Ninject.Web.Common/wiki/Setting-up-a-OWIN-WebApi-application. Unfortunately, I ran into a little issue.

It turns out, that, because I am using the latest version of [Microsoft.Owin](https://www.nuget.org/packages/Microsoft.Owin/), which is 3.0.1 at the time of this writing. However, the latest stable version of [Ninject OWIN host for WebApi 2](https://www.nuget.org/packages/Ninject.Web.WebApi.OwinHost/3.2.4) has a set of dependencies that will depend on Microsoft.Owin 2.0.0.

Now, I realize the instructions on the Ninject website that I linked up above say to specifically install **Ninject.Web.Common.OwinHost** and **Ninject.Web.WebApi.OwinHost**. For some reason, I decided to look at the second, saw that Ninject.Web.Common.OwinHost was a dependency, and just decided to install that and allow everything else to fall out. In fact, when I ran into this problem, I mapped out the dependency tree. Here it is:

```
Ninject.Web.WebApi.OwinHost (3.2.4)
  Ninject.Web.WebApi (≥ 3.2.0.0 && < 3.3.0.0)
    Ninject (≥ 3.2.0.0 && < 3.3.0.0)
    Ninject.Web.Common (≥ 3.2.0.0 && < 3.3.0.0)
      Ninject (≥ 3.2.0.0 && < 3.3.0.0)
    Microsoft.AspNet.WebApi.Core (≥ 5.0 && < 6.0)
      Microsoft.AspNet.WebApi.Client (≥ 5.2.3)
        .NETFramework 4.5
        Newtonsoft.Json (≥ 6.0.4)
  Ninject.Web.Common (≥ 3.0 && < 4.0)
    Ninject (≥ 3.2.0.0 && < 3.3.0.0)
  Ninject.Web.Common.OwinHost (≥ 3.0 && < 4.0)
    Ninject.Web.Common (≥ 3.2.0.0 && < 3.3.0.0)
      Ninject (≥ 3.2.0.0 && < 3.3.0.0)
    Microsoft.Owin (≥ 2.0 && < 4.0)
      Owin (≥ 1.0)
  Ninject.Extensions.ContextPreservation (≥ 3.2 && < 4.0)
    Ninject (≥ 3.2.0.0 && < 3.3.0.0)
  Ninject.Extensions.NamedScope (≥ 3.2 && < 4.0)
    Ninject (≥ 3.2.0.0 && < 3.3.0.0)
  Microsoft.AspNet.WebApi.Owin (≥ 5.0 && < 6.0)
    Microsoft.AspNet.WebApi.Core (≥ 5.2.3 && < 5.3.0)
      Microsoft.AspNet.WebApi.Client (≥ 5.2.3)
        .NETFramework 4.5
        Newtonsoft.Json (≥ 6.0.4)
    Microsoft.Owin (≥ 2.0.2)
      Owin (≥ 1.0)
    Owin (≥ 1.0)

Ninject.Web.Common.OwinHost (3.2.3)
  Ninject.Web.Common (≥ 3.2.0.0 && < 3.3.0.0)
    Ninject (≥ 3.2.0.0 && < 3.3.0.0)
  Microsoft.Owin (≥ 2.0 && < 4.0)
    Owin (≥ 1.0)
```

As you can see, everything looks good for any reference to Microsoft.Owin across the board. In fact, I am a bit stumped as to how I ran into this issue in the first place. (If I get Ninject.Web.WebApi.OwinHost version 3.2.3, a dependency exists that requires a version < 3 of Microsoft.Owin, but, that doesn't make sense based on the above dependency tree.) If anybody has any insight, would really appreciate further clarification in the comments below.

Maybe the lesson to learn here is, next time, I should listen to the instructions and install each package separately and in the right order.
