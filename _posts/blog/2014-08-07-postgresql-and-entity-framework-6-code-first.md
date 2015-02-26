---
layout: post
title: "PostgreSQL and Entity Framework 6 Code First"
date: 2014-08-07
modified:
categories: blog
excerpt:
tags: [.net, code-first, entity-framework, postgresql]
image:
  feature:
share: true
comments: true
---
{:.center}
![Postgresql Logo]({{ site.url }}/images/postgresql-logo.png)

Recently, I made the decision to switch to PostgreSQL from MySQL on a project I was working. There were various reasons for this, the primary being that MySQL’s license terms are dual-licensed under GPL and Commercial licenses. Since neither met my needs (and there are some [technical issues with MySQL](https://plus.google.com/+JasonCavett/posts/VDXDmRxm9LD) that have given me issues), I decided to give Postgres a try.

I had already done the work to setup MySQL with Entity Framework 6 (I may post on this in the future), so I figured it would be fairly straightforward to switch over. It was, but there were a couple extra steps I needed, as well as a caveat from my switch from MySQL, in order to ensure everything worked properly.

#Steps to Success
Of course, to start, make sure you have Postgres installed on your system. Follow the link to download the [EnterpriseDB Windows installer](http://www.postgresql.org/download/windows/). This will give you not only Postgres, but also pgAdmin III which is a reasonable interface for managing Postgres. (I say reasonable only because of my general focus on usability. But, as a whole, it gets the job done.)

There are a couple configuration steps after the database has been installed, but I will discuss these later.

Once you have finished your installation, fire up Visual Studio and the solution/project you will be using.

There are a number of packages you’ll need from Nuget in order to properly interact with Postgres via Entity Framework. At the time of this writing, the Npgsql packages are not a stable release, so you’ll need to enable Nuget to include prerelease packages. These packages are:

* [**Entity Framework**](http://www.nuget.org/packages/EntityFramework/) – Obviously, you’ll need EF in order to use EF. As of this blog post, 6.1.1 is the official release version.
* [**Npgsql**](http://www.nuget.org/packages/Npgsql) – [Francisco Figueiredo Jr](http://fxjr.blogspot.com/), and a number of other software engineers who I am very thankful for, have developed this library which is a .NET data provider for Postgres. Lucky for me (and for you), [Francisco announced in his blog on August 5th](http://fxjr.blogspot.com/2014/08/npgsql-220-release-candidate-1-released.html) that Npgsql 2.2.0-RC1 has been released which provides many updates in supporting Code First. Love that timing!
* [**Npgsql for Entity Framework**](http://www.nuget.org/packages/Npgsql.EntityFramework) - This is a Postgres provider for Entity Framework 6. Like Npgsql, you’ll need the latest release candidate (2.2.0) in order to generate migrations from code and perform the migrations.

Install each of these Nuget packages into your project. Once they are installed, you’ll need to update your App.config (or Web.config if you’re doing that sort of thing) to include the proper information to have EF6 and Npgsql work together. Again, [Francisco has a good article](http://fxjr.blogspot.com/2014/02/using-entity-framework-6-with-npgsql-210.html) on what you need to do. But, I am reposting the config file for archival purposes.

**Note, that some of the values are not 100% identical – particularly the references to Entity Framework. The Nuget installation should take care of a lot of these values for you.**

{% highlight xml %}
<xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework,
 Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
  </configSections>
  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
  </startup>
  <entityFramework>
    <providers>
      <provider invariantName="Npgsql" type="Npgsql.NpgsqlServices, Npgsql.EntityFramework"></provider>
    </providers>
    <defaultConnectionFactory type="Npgsql.NpgsqlConnectionFactory, Npgsql" />
  </entityFramework>
  <system.data>
    <DbProviderFactories>
      <remove invariant="Npgsql" />
      <add name="Npgsql Data Provider" invariant="Npgsql" support="FF" description=".Net Framework Data Provider for Postgresql"
 type="Npgsql.NpgsqlFactory, Npgsql" />
    </DbProviderFactories>
  </system.data>
</configuration>
{% endhighlight %}

At this point, you are almost ready to begin. Just a few code changes…

The biggest in particular is specifying your schema. By default, Entity Framework uses the dbo schema. Postgres, on the other hand, supplies a public schema by default. You actually don’t have to use either (and the public schema may have some security concerns due to its default access permissions), but I ran into considerable issues using the dbo schema.

If you wish to enforce a particular schema (I ended up using public for my own personal project), you’ll need to override <span style="color: red">OnModelCreating</span> of your <span style="color: red">DbContext</span> and set the default schema.

{% highlight c# %}
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    // PostgreSQL uses the public schema by default - not dbo.
    modelBuilder.HasDefaultSchema("public");
    base.OnModelCreating(modelBuilder);
}
{% endhighlight %}

That’s it from the configuration and code standpoint.

Now, the caveat I mentioned earlier. If you recall, I was coming from MySQL. In order for MySQL to work nicely with EF6, I had to do a lot of jumping through hoops and customizations in order for it be clean. As a result, my existing migrations would not work with Postgres. So, instead, I deleted my old migrations and reinitialized them. I realize this may not be the best course of action for everybody, and I’m guessing I could have hand-edited the migrations to be correct. But, I was a bit suspicious of what may have actually been generated and I could afford a fresh start.

So, that’s it. The Npgsql team is doing a great job and it really made this process easy.
