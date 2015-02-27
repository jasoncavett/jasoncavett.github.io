---
layout: post
title: "Changing Images During an Android Animation"
date: 2011-05-23
modified:
categories: blog
excerpt:
tags: [android, animation, java]
image:
  feature:
share: true
comments: true
---
I was recently working on a portion of my application where I wanted to animate a die rolling across the screen. As part of this animation, there were two aspects. First, I had to build the animation so that the dice would move across the screen. The second part of the animation required that the die flip and tumble.

Now, before I get into this farther, I am not going to show that exact animation for this tutorial. However, from this tutorial, it is fairly easy to extrapolate to get the full animation effect. Instead, I am going to focus on how to make these two aspects of the animation to work together.

For this tutorial, I grabbed the ‘shake’ animation from [Android API demos](http://developer.android.com/resources/samples/ApiDemos/src/com/example/android/apis/view/index.html). Here is what the shake animation looks like.

`/res/anim/shake.xml`
{% highlight xml %}
<set android:interpolator="@anim/cycle" android:shareinterpolator="true" xmlns:android="http://schemas.android.com/apk/res/android">
 <translate android:duration="2000" android:fromxdelta="-5" android:toxdelta="5">
</translate></set>
{% endhighlight %}

I next created my content view layout which looks like the following:

`/res/layout/dice.xml`
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
 android:layout_width="fill_parent" android:layout_height="fill_parent"
 android:gravity="top|right" android:hapticFeedbackEnabled="true">
 <ImageView android:id="@+id/background"
  android:layout_height="fill_parent" android:layout_width="fill_parent"
  android:scaleType="fitXY" />
 <ImageButton android:id="@+id/dice" android:clickable="true"
  android:onClick="rollDie" android:src="@drawable/dice_sides"
  android:layout_alignParentRight="true" android:layout_width="wrap_content"
  android:layout_height="wrap_content" android:background="@null" />
</RelativeLayout>
{% endhighlight %}

Three things to point out.

1. The `ImageButton` has `android:background` set to `@null`. This ensures that no border is drawn around the button. (Basically, it won’t look like a button.)
2. The second thing you should notice is that the source for the `ImageButton` is called `dice_sides`. I will get to that in a moment.
3. My `onClick` action for the `ImageButton` is `rollDie`. Let’s take a look at that method inside the `DiceActivity` first.

{% highlight java %}
/**
  * Handle the roll of the dice.
  *
  * @param view
  *            The view that is being acted upon.
  */
 public void rollDie(View view) {
  // Setup the animation.
  Animation shake = AnimationUtils.loadAnimation(view.getContext(),
    R.anim.shake);
  View dice = findViewById(R.id.dice);
  dice.setAnimation(shake);

  shake.start();
  handler.sendMessageDelayed(Message.obtain(handler, 0, 7), 200);
 }
{% endhighlight %}

If you have taken any look at the [animation tutorials](http://developer.android.com/guide/topics/resources/animation-resource.html) on the Android developer site, this should all be pretty straightforward. I load the animation, I set the animation on the view that I care about, and I start it. The only thing that should not look entirely familiar is line 15. This line is what allows us to pair animations so that the die looks like it is rolling.

In order for that part of the animation to work, we need two things. First, is dice_sides.xml which, if you recall, I referenced as the source within the content view layout for the `ImageButton`. `dice_sides.xml` is really a [LevelListDrawable object](http://developer.android.com/reference/android/graphics/drawable/LevelListDrawable.html). Using this will allow us to simulate a movement between the various drawable objects.

`/res/drawable/dice_sides.xml`
{% highlight xml %}
<level-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:maxLevel="0" android:drawable="@drawable/dice_1" />
    <item android:maxLevel="1" android:drawable="@drawable/dice_2" />
    <item android:maxLevel="2" android:drawable="@drawable/dice_3" />
    <item android:maxLevel="3" android:drawable="@drawable/dice_4" />
    <item android:maxLevel="4" android:drawable="@drawable/dice_5" />
    <item android:maxLevel="5" android:drawable="@drawable/dice_6" />
</level-list>
{% endhighlight %}

Of course, you will need all the various drawable images that are referenced within this item list.

The second part that needs to be completed in order to complete the animation of the dice rolling is the use of the handler which I started discussing above in the `rollDie` method.

Let’s take a look at some code. This inner class is within my `DiceActivity` class.

`DiceActivity.java`
{% highlight java %}
// Variables to manage the dice roll.
private Random rnd = new Random();
private DiceRollHandler handler = new DiceRollHandler();

/**
 * Handler to manage the rolling of the die.
 *
 * @author Jason Cavett
 */
class DiceRollHandler extends Handler {
 /**
  * @see android.os.Handler#handleMessage(android.os.Message)
  */
 public void handleMessage(Message msg) {
  int value = rnd.nextInt(6);
  ImageView dice = (ImageView) DiceActivity.this
    .findViewById(R.id.dice);
  dice.setImageLevel(value);

  // If there are still rolls available, roll another time.
  Integer rollsLeft = (Integer) msg.obj;
  if (rollsLeft &gt; 0)
   DiceActivity.this.handler.sendMessageDelayed(Message.obtain(
     DiceActivity.this.handler, 0, --rollsLeft), 200);
 }
}
{% endhighlight %}

I declare two private variables outside the inner class so that I can use them properly within the handler. What you see happening here is fairly straightforward. At the time the animation fires with the onClick action, I fire a message to my DiceHandler with a value of 7 and a delay of 200ms. The above method is called as a result. I then generate the next random value and get the dice image view via the resource ID. I then call the setImageLevel(int) method which will display the image corresponding to the number from the previously shown level-list. (Remember that the source of the dice view is the level-list.) To complete the animation (which I want to continue until the “shake” effect is done), I grab the number of rolls and send another message with the roll count decremented by one. This continues on until there are 0 rolls left. Once that occurs the message stops firing and the animation is finished.

As I said previously, it is fairly straightforward to expand this to even more advanced animations. Because I couldn’t find any information about this on the internet, and I had seen other questions asking about this same type of problem, I wanted to post it here. I hope this helps someone out!
