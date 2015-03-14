---
layout: post
title: "Retrieving an Android Resource by Name"
date: 2011-05-18
modified:
categories: blog
excerpt:
tags: [android, java]
image:
  feature:
share: true
comments: false
---
Per my last blog post, now that I am finished with graduate school, I can devote time to pet projects that I have been meaning to work on for some time now. The project that has been first and foremost on my plate is an Android application.

As part of the application I am developing, I want to allow user to select a theme (in the application preferences) and have the background of my application change accordingly. Now, assuming that all of my background images are in the drawable resources directory (R.drawable) it would be simple enough to just grab the theme using the getDrawable(int)method. However, this method, while initially appearing to be simple to implement, will quickly become a problem for a number of reasons.

1. It is not very generic. Each value has to be hard-coded and, for each theme, I would have to add multiple paths to multiple background types. (Side note: Each theme in this particular application has different backgrounds depending on the screen the user is on or based on the status of the application.)
2. It is not easy to distinguish between themes. I want to allow the user to be able to select the theme they want to use in a drop down menu box in preferences. Unfortunately, there is no (obvious) way to link that string with the hard-coded references to the drawable objects.
3. If I later want to allow the user to add their own themes to the system, this current solution would not allow for that to happen.

Breaking down those problems, I essentially had these requirements:

1. I have to be able to access the themes by name.
2. I needed to be able to get to the resources by name rather than by their IDs.
3. The solution had to support users creating their own themes in the future.

Well, fortunately for me (and hopefully for you, too), the Android SDK provides a nice way to access resources by name. I’ll get to that in a moment.

First (of course) I had to make sure my drawable resources were in res/drawable (or res/drawable-* if [supporting multiple screen sizes](http://developer.android.com/guide/practices/screens_support.html)). Now, because I wanted to be able to access the resources by name, and because I wanted to do it generically, I named each file according to its type. So, something like the following:

* res/drawable
  * themename1_home
  * themename1_screen1
  * themename2_home
  * themename2_screen1

This naming will be important in a moment.

The second thing I did was create a ThemeManager class which provides the access to my available themes, as well as provides access to my Drawable objects. Here is a bit of the class.

{% highlight java %}
public class ThemeManager {

  // Notice the match between my types and the names in res/drawable.
  public static final String HOME_TYPE = "_home";
  public static final String SCREEN1_TYPE = "_screen1";

  /**
   * The names of the themes.
   */
  public enum Themes {
    ThemeName1,
    ThemeName2;

    /**
     * Override toString in order to return the
     * name as expected by getIdentifier().
     */
    public String toString() {
      return this.name().toLowerCase();
    }

    // ... more

  }
{% endhighlight %}

You’ll notice the importance of naming here. Obviously, if I’m going to retrieve the resources by name, the names have to match. This is why the constant values, as well as the names of the themes are so important.

Now that we’ve done this much, we can now retrieve the `Drawable` resource. This method is inside the `ThemeManager` class.

{% highlight java %}
public static Drawable getThemeBackground(Context context, String themeType) {
  SharedPreferences pref = PreferenceManager.getDefaultSharedPreferences(context);
  String name = pref.getString(MyPreferenceActivity.THEME, DEFAULT_THEME);

  int resId = context.getResources().getIdentifier(
    name + themeType, "drawable", "com.jasoncavett.application");
  return context.getResources().getDrawable(resId);
}
{% endhighlight %}

As you can see here, the secret ingredient is getIdentifier. This method takes three parameters – the name of the resource (**remember not to include extensions**), the type of resources, and the package that the resource resides in (typically this will be the top-level package of your application). This method returns the ID of the resource that you are requesting, and you can then get the resource as you would normally via the resource ID.

You may also be wondering if this solution meets my requirement for allowing users to add their own themes. Absolutely! In the same getThemeBackground method above, it could be modified so that if the Drawable object is null, I can then go look in another path (such as on the sdcard). The only difference is that the user would have to follow a certain folder structure and, of course, they would have to following the naming scheme I put in place.

I am not demonstrating how to setup and use preferences, nor am I demonstrating how to apply the Drawable object to the background. The [Android SDK website](http://developer.android.com/sdk/index.html) has a great deal of information in how to do this if you need help in those areas. (A tip on the preferences – you already have a list of the names via your enumeration in the ThemeManager class…)

I hope this post was useful. I’m going to continue learning the Android SDK and hope to have another post up in the near future!
