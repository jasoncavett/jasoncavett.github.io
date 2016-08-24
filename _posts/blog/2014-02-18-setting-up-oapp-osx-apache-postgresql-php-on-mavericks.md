---
layout: post
title: "Setting Up OAPP (OSX, Apache, PostgreSQL, PHP) on Mavericks"
date: 2014-02-18
modified:
categories: blog
excerpt:
tags: [apache, osx, php, postgresql]
image:
  feature:
share: true
comments: false
---
{:.center}
![PHP, Apache, and Postgresql]({{ site.url }}/images/php-apache-postgres.png)

I have long used Linux as my main development platform. And, of course, for anything .NET, I develop on Windows. Fairly recently, however, I began using OSX. A new project (and a foray into PHP) required that I setup Apache, Postgres, and PHP on my Mac.

Unfortunately, as I searched across the internet, there were a number of different ways and methods of installing these components that I decided I would add my own voice for a setup that works for me.

#Installation Setup
Before beginning the rest of the installation, you should first install [Homebrew](http://brew.sh/). The installation is straightforward so I will leave you to head to the Homebrew site and follow those steps there. The primary thing to remember about Homebrew and this point going forward is that Homebrew installs packages to their own directories and then symlinks those into the `/usr/local` directory. For this reason (and something I assume in this post), you should include `/usr/local` in your classpath via your bash_profile.

#Installing PHP
As you may already know, PHP is already available on Mavericks. However this version is 5.4 and [PHP 5.5 is up to its 9th release](http://www.php.net/downloads.php#v5.5.9) with 5.6 in alpha. So, I decided I needed to be able to keep my system upgraded fairly easily.

Because PHP isn’t available from the default Homebrew taps, you’ll have to tap [Jose Gonzalez’s Homebrew PHP tap](https://github.com/josegonzalez/homebrew-php). To do this, you will tap homebrew/dupes which provides formulas that are already part of the OSX installation but may contain upgrades and/or patches. Via the terminal, enter the following commands:

{% highlight bash %}
$ homebrew tap homebrew/dupes
{% endhighlight %}

And also bring in the PHP tap

{% highlight bash %}
$ homebrew tap josegonzalez/homebrew-php
{% endhighlight %}

Now, you can install PHP. This particular tap offers multiple versions, but we’ll use the latest available at the time of this writing – PHP 5.5.

{% highlight bash %}
$ homebrew install php55
{% endhighlight %}

This will use the PHP55 formula to download and install PHP. This may take some time depending on your machines specs as it is building PHP from scratch.

Once the process has completed, and assuming that /usr/local is in your PATH, you should be able to restart your terminal and now type:

{% highlight bash %}
$ php -v
{% endhighlight %}

#Installing Apache
In the case of Mavericks, I will be using the version of Apache that is installed with the operating system due to the fact that this is a development setup, and this (older) version of Apache will not be used in production.

Because Apache is already installed, using it is as simple as running the command:

{% highlight bash %}
$ sudo apachectl start
{% endhighlight %}

You can visit [http://localhost](http://localhost/) in your browser to make sure it says, “It works!” If it does, Apache is running correctly. However, for setting up a PHP environment, you also want to enable PHP on Apache. This is done by updating your LoadModule call. You’ll have to comment out the old version and add a line for the new version. (Note, this is taken from the homebrew-php website.)

{% highlight conf %}
# /etc/apache2/httpd.conf
# $HOMEBREW_PREFIX is normally `/usr/local`
# Note that you choose the correct path for your version.
# LoadModule php5_module    libexec/apache2/libphp5.so
LoadModule php5_module    $HOMEBREW_PREFIX/Cellar/php55/5.5.9/libexec/apache2/libphp5.so
{% endhighlight %}

Once you’ve done this, restart Apache to have the changes take effect.

{% highlight bash %}
sudo apachectl restart
{% endhighlight %}

If you’d like, you can test to make sure that Apache recognizes the correct version. Create a file called `info.php` and place it in your DocumentRoot, which can be found in `/etc/apache2/httpd.conf` (and is most likely `/Library/WebServer/Documents`). The contents of the file should be:

{% highlight php %}
<!--?php <br ?-->phpinfo();
{% endhighlight %}

The page should show the correct version of PHP.

#Installing PostgreSQL
There are number of ways in which you can install Postgres on OSX. However, the easiest way, by far, is to do it via the [Postgres.app](http://postgresapp.com/). It’s as simple to install as downloading the application and moving it into your application folder. I will leave it up to the reader to run and perform the setup of any databases that they wish to use on Postgres.

However, there is one more major step that must be performed after installing Postgres.app – you have to install and enable the PDO_PGSQL driver. You may already have [PEAR installed](http://pear.php.net/manual/en/installation.getting.php). You can test this through the following command:

{% highlight bash %}
$ which pear
{% endhighlight %}

If you get a path back, then you should be good to go to finish the installation. If not, follow the link above on how to install PEAR. Once PEAR is installed, you can then continue by using PECL to download pdo_pgsql. Then, because PECL can’t install the driver directly, you’ll build it and install it.

{% highlight bash %}
$ pecl download pdo_pgsql
$ tar xzf PDO_PGSQL-1.0.2.tgz
$ cd PDO_PGSQL-1.0.2
$ phpize
$ ./configure --with-pdo-pgsql=/path/to/your/PostgreSQL/installation
$ make && sudo make install
{% endhighlight %}

If everything works correctly, pdo_pgsql.so should be placed in a directory like `/usr/lib/php/extensions/no-debug-non-zts-20060613/``. Now that you’ve done that, you have to update `php.ini` to point to the extension.

{% highlight conf %}
# /user/local/etc/php/5.5/php.ini
# Replace the version with your correct version.
extension=pdo_pgsql.so
{% endhighlight %}

#Conclusion
At this point, you have now installed PHP (which can be kept updated through future Homebrew updates), you are using Apache (which is installed with OSX), and you are using PostgreSQL with the appropriate extensions. Hope this helps someone!
