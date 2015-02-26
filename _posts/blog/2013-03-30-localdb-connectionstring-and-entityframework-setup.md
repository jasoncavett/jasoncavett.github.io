---
layout: post
title: "LocalDB ConnectionString and EntityFramework Setup"
date: 2013-03-30
modified:
categories: blog
excerpt: "Some quick configuration to simplifying database development."
tags: [entity-framework, sql]
image:
  feature:
share: true
comments: true
---
The world of SQL Server can be fairly confusing to someone who is new to it. Heck, it can be confusing to someone who knows it really, really well.

With the release of Microsoft’s 2012 developer products, and particularly with the release of SQL Server Express 2012, Microsoft introduced LocalDB. LocalDB is “a minimal, on-demand, version of SQL Server that is designed for application developers.” Namely, LocalDB has these following benefits:

Small Installation Footprint
Zero Configuration
Run as non-admin
Offers same/similar features as SQL Server Express
In short, LocalDB is really nice for application developers. If you want to see more of the pros (and cons) of LocalDB, head over to [SQLCoffee to see the list](http://www.sqlcoffee.com/SQLServer2012_0004.htm).

So, given the goodness of LocalDB, I wanted to start using it for one of my own applications (particularly an application that is using [Entity Framework](http://msdn.microsoft.com/en-us/data/ef.aspx)). I started searching the web for the configuration and found way too many different suggestions. So, I’m going to add my own into the mix as this is what worked for me. Below is the connection string and the EF connection information. Typically, this goes into your Web.config or App.config file.

{% highlight xml %}
<connectionstrings>
  <add name="DefaultConnection" providerName="System.Data.SqlClient" connectionString="Data Source=(LocalDB)\v11.0;AttachDbFilename='|DataDirectory|DatabaseName.mdf';Integrated Security=True" />
</connectionStrings>
{% endhighlight %}

...and...

{% highlight xml %}
<entityframework>
  <contexts>
    <context type="Com.JasonCavett.Data.DAL.Context, Com.JasonCavett.Data, Version=1.0.0.0, Culture=neutral">
      <databaseinitializer type="MyProject.Migrations.Initializer, MyProject" />
    </context>
  </contexts>
  <defaultconnectionfactory type="System.Data.Entity.Infrastructure.LocalDbConnectionFactory, EntityFramework">
    <parameters>
      <parameter value="v11.0" />
    </parameters>
  </defaultConnectionFactory>
</entityFramework>
{% endhighlight %}
