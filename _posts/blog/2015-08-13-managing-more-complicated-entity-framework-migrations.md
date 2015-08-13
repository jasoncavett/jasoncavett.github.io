---
layout: post
title: "Managing More Complicated Entity Framework Migrations"
date: 2015-08-13
modified:
categories: blog
excerpt:
tags: [entity framework, database, migrations, .net]
image:
  feature:
share: true
comments: true
---
While there is a lot of debate about the goodness of ORMs, I will be the first to admit - I love them. The less "touching" I have to do of databases directly, the happier I am - particularly because much of my focus is on usability and the front-end. (Not to say I can't, of course, but, it's not what gets me really excited when it comes to technology.)

Doing much of my work in .NET land, the defacto ORM is [Entity Framework](http://www.asp.net/entity-framework). One of the aspects of EF that I appreciate the most is its ability to perform migrations - particularly [Code First Migrations](https://msdn.microsoft.com/en-us/data/jj591621.aspx). Having this capability allows me to stay "in the code" while quickly being able to stand up a new datasource and evolve it over time. In addition, the migrations are easily committed to source control so any other developer will also be in line with the same backend. This incremental approach to creating the database very much appeals to me.

An example of what a simple migration may look like can be seen here:

{% highlight c# linenos %}
namespace Cavett.MyProject.Data.Migrations
{
    using System;
    using System.Data.Entity.Migrations;

    public partial class AddFirstName : DbMigration
    {
        public override void Up()
        {
            AddColumn("public.People", "FirstName", c => c.String(nullable: false));
        }

        public override void Down()
        {
            DropColumn("public.People", "FirstName");
        }
    }
}
{% endhighlight %}

Very simple, you can see that a migration was created that will update my database to add the column FirstName to the People table. Even better, I can migrate my database back the other way and the column will be removed. This allows me to seamlessly move between versions of my application if needed.

Unfortunately, as with all technology, real life sometimes creates edge cases that don't fit into the typical scenario. I recently ran into one of these cases where I needed to perform a migration on an existing database that was rather complicated in nature. The change in structure was part of a much larger refactoring process that was going on in the code and the team wanted to bring the database up-to-date to align better with the higher level code changes.

The problem that the team quickly ran into is that the migrations they had developed caused the internal data to be mangled. This primarily was occurring because entire tables were being normalized and a lot was changing within the migration. Since the project was already using EF, we wanted to stick with that, but it was obvious that something was going to need to step in to handle the data as the migrations were running.

After some investigation, I discovered that Entity Framework makes room for the use of a [SqlResource](https://msdn.microsoft.com/en-us/library/system.data.entity.migrations.dbmigration.sqlresource(v=vs.113).aspx) directly within its methods to make direct SQL calls to the database.

To use it, first, create your migration as you normally would - but do not run it against your database. This will place the migrations files into your project - typically they look something like timestamp_MigrationName.cs. Now, create an Embedded Resource SQL file that is named similarly - timestamp_MigrationName_Down.sql and timestamp_MigrationName_Up.sql. The reason for the embedded resource is so your SQL scripts that you write will travel with your migrations upon deploy. (My experience with this process is that, if you don't, EF has difficulty finding/running the SQL.)

At this point, you should go ahead and write your SQL to handle the migration of the data. I'll just let you go do that...

Now that you've finished that up, head into your migration and add the following line in the appropriate location. Generally speaking, a good rule of thumb would be - after your "Add" columns/tables, but before you drop the old ones. ;)

{% highlight c# %}
SqlResource(this.ScriptPath("Up"));
{% endhighlight %}

That's all that is required. It's really quite a simple solution. Now, go ahead and migrate to your heart's content!
