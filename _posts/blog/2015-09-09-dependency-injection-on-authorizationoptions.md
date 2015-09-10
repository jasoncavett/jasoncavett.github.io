---
layout: post
title: "Dependency Injection on AuthorizationOptions"
date: 2015-09-09
modified:
categories: blog
excerpt:
tags: [.net, dependency injection, asp.net 5, authorization]
image:
  feature:
share: true
comments: true
---
This particular blog comes out of the work that I have been doing as of late with [ASP.NET 5](), [MVC 6](), and [all the other goodness]() that Microsoft has been releasing as of late in the development arena.

Before I get into the particulars, I do want to say that it is fun (yes, fun!) programming and running .NET code in my native OSX environment. And, while I would love to say that I miss using Visual Studio...well...I'm definitely more of a fan of the minimalist approach when it comes to my editors.

Anyway, over the past week, I have been writing code in the latest and greatest .NET has to offer to replace a Classic ASP system. It's been a lot of fun, and the development moves along very quickly.

One aspect of the system I have been reworking is the authentication system. With the introduction of the [Identity](http://www.asp.net/identity) (code can be found [here](https://github.com/aspnet/Identity)) subsystem, it has become ridiculously easy to add authentication and to customize user data. I won't get into all the specifics, but you can follow the links to learn more.

One aspect of the Identity system that I found particularly attractive was [code configured policy support](https://github.com/aspnet/Announcements/issues/22). As I quickly learned, this type of support makes it extremely easy to write security policies, in code, that can check for just about anything your heart desires, enforcing those policies in the same places you can apply `[Authorize]`.

In my particular case, I had to create a rule that determined whether or not a logged in user had permissions to edit profile information for another user (think a management structure). This logic could now be encapsulated as a policy rather than having to be scattered throughout the code, or placed in some helper method. Beautiful!

I ran into one issue. My policy required access to by data repository object in order to grab the appropriate data to determine the ability to do this. The policy requirement's constructor was simple:

{% highlight csharp %}
public MyRequirement(MyRepository repository)
{
    Repository = repository;
}
{% endhighlight %}

Inside `Startup.cs`, I added the requirement to my policy, like this (originally).

{% highlight csharp %}
services.Configure<AuthorizationOptions>(options =>
    {
        options.AddPolicy("AllowProfileManagement", policy => policy.Requirements.Add(
            new AllowProfileManagementRequirement(new MyRepository(new MyContext()))));
    });
{% endhighlight %}

Of course, this is a problem in the long run because I had to tightly couple my context and repository with my policy requirement. In addition, when I actually went to run the code, MyContext did not know the connection string to get to my database.

Really, I wanted to use the Dependency Injection for my policy requirements just like I had with dependencies throughout my code. Ultimately, it ended up being fairly simple.

{% highlight csharp %}
services.Configure<AuthorizationOptions>(options =>
    {
        options.AddPolicy("AllowProfileManagement", policy => policy.Requirements.Add(
            services.BuildServiceProvider().GetRequiredService< AllowProfileManagementRequirement>()));
    });
{% endhighlight %}

`services.BuildServiceProvider()` returns the default DI container. The rest is simply wiring up the dependencies.

*This post originally generated from my question at StackOverflow: [Dependency Injection on AuthorizationOptions](http://stackoverflow.com/questions/32470047/dependency-injection-on-authorizationoptions)*
