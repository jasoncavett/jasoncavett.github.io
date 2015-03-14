---
layout: post
title: "Accessing the Watchdog Timer on the Arduino Uno Microcontroller"
date: 2011-03-17
modified:
categories: blog
excerpt:
tags: [arduino, real-time-systems]
image:
  feature:
share: true
comments: false
---
As I discussed in a [previous blog post](http://www.jasoncavett.com/2011/03/arduino-uno-microcontroller-has-arrived.html), I am currently enrolled in a real-time systems course as my final course before I earn my MS in Software Engineering. As part of the course, we are to develop a real-time systems project. In particular, we are to develop the beginnings of an artificial pancreas which will be used to manage insulin levels for individuals with diabetes. What’s interesting about this project is that it is one that is being worked on in industry, although, as it can be imagined, a real version of this device is going to be far more complicated and have much more stringent requirements than the one I am developing for class.

As part of the real-time systems course, we are learning about interrupts. The Arduino Uno has two interrupts that are easy to get to (AKA, the pins are provided directly from the hardware) and, using the [Arduino development environment](http://arduino.cc/en/Main/Software), it becomes trivial to [use those interrupts](http://arduino.cc/en/Reference/AttachInterrupt).

As part of the artificial pancreas project, I wanted to use a timer so I could have the user’s blood-level (which is simulated in a C# application on my computer) checked at regular intervals. While this is obviously possible with a 5x5x5 timer or something similar, my professor had mentioned the presence of a watchdog timer (WDT) which was already part of the board. I decided I wanted to use this timer as a third interrupt.

Now, as Wikipedia points out, a WDT:

> is a [computer](http://en.wikipedia.org/wiki/Computer) hardware or software [timer](http://en.wikipedia.org/wiki/Timer) that triggers a system [reset](http://en.wikipedia.org/wiki/Reset_(computing)) or other > corrective action if the main [program](http://en.wikipedia.org/wiki/Computer_program), due to some fault condition, such as a
> [hang](http://en.wikipedia.org/wiki/Hang_(computing)), neglects to regularly service the watchdog. The intention is to bring
> the system back from the unresponsive state into normal operation.

Because of this, I do realize that it may not necessarily be a good idea to take control of the timer and, if you continue, be aware that you could make it very difficult (and possibly impossible) to use your board. **I am not responsible for anything you may do to your own microcontroller.**

Okay, now that we have the disclaimers out of the way (sorry…), let’s get to the fun.

I first want to point out that I have the original bootloader. I have not yet [purchased or burned](http://www.adafruit.com/index.php?main_page=product_info&products_id=123) Lady Ada’s bootloader. Now, in order to start using the WDT, the first place I went to was the [Arduino Uno (and, ultimately, the ATmega168) documentation](http://www.atmel.com/dyn/resources/prod_documents/doc2545.pdf). While the Arduino language did not provide a way to get to the WDT directly, I had to find out if it was even possible. Fortunately for me, this led me (almost) right to the solution.

If you head to section 10.9.2, you will find the information regarding the Watchdog Timer Control Register, or the WDTCSR. This is where the magic happens.

In order to use the WDT, you have to enable the use of the WDT interrupt (AKA, you basically have to hijack it – that’s the best description I have been able to come up with). Before I continue, let’s take a look at some code (all done in the Arduino language):

{% highlight java linenos %}
#include
#include <avr/interrupt.h>;
#include <avr/wdt.h>;

/*
 * The setup function.
 */
void setup() {
  Serial.begin(9600);
  pinMode(LED_PIN, OUTPUT);
  wdtSetup();
}

/*
 * The main loop.
 */
void loop()
{
  digitalWrite(13, HIGH);   // set the LED on
  delay(1000);              // wait for a second
  digitalWrite(13, LOW);    // set the LED off
  delay(1000);              // wait for a second
  Serial.println("Background");
}

/*
 * Setup of the watchdog timer.
 */
void wdtSetup() {
  cli();
  MCUSR = 0;
  WDTCSR |= B00011000;
  WDTCSR = B01000111;
  sei();
}
{% endhighlight %}

This particular sketch (as Arduino programs are called) began as the basic Blink sketch. I have further modified it. Let’s work top-down.

* **Lines 1-3** - I have included the necessary header files which are needed in order to access the WDT. If you are using the Arduino software environment, the libraries are already available to you without any additional work. Just include them.
* **setup()** – This is your typical setup function. Notice that I am calling wdtSetup here. We’ll get to that in a moment.
* **loop()** - The background loop. I just wanted to make sure the basic program would continue to work even if I was using the WDT.
* **wdtSetup()** - This is the interesting function. I use the cli and sei commands which come from the Interrupt library. They disable and enable all interrupts via a global mask, respectively. I wanted to do this so there would be no problems when I am “taking over” the WDT.The middle three lines set various flags and registers. I will admit to not fully understanding what MCUSR is for. I believe it is cleaned when the Arduino is reset via the reset button so the bootloader is not skipped. I was advised to set this to 0. If anybody has any further knowledge on this, I would appreciate any insight.

The WDTCSR register (which is discussed in section 10.9.2 in the PDF I linked above) is what gives you access to the Arduino WDT. The first statement clears bits 3 and 4. Bit 4 is Watchdog Change Enable. This bit must be cleared in order to change or clear the WDE bit. Bit 3, the WDE bit, must also be cleared in order to set WDP3 through WDP0 (which is the next statement).

The next statement sets the WDP3 through WDP0 bits. Doing this sets the WDT prescale, or the time at which the WDT interrupt fires. I chose two seconds. See page 54 of the PDF for the other time-outs and adjust the bits accordingly.

That’s it. You’re WDT is now ready to be used by you…with one more step. Take a look at the following function:

{% highlight c %}
/*
 * Watchdog Timer Interrupt
 */
ISR(WDT_vect)
{
    Serial.println("Watchdog!");
}
{% endhighlight %}

This function (which I place as the top function in my sketch) is triggered at the interval that you set on the WDP bits. You can place whatever code you want to have happen at regular intervals inside this function. For my project, I am going to use the WDT to track time so I can check insulin levels every X hours (whatever I end up choosing). I will obviously need another memory location to track that length of time since the WDT is ticking every 2 seconds.

Now, I want to caveat all of this by saying that I am still very much learning about the Arduino Uno. There may be reasons why I don’t want to do the above (besides the obvious), but I was rather excited that I figured this all out, and I am happy to have an additional interrupt to use without having to wire up any extra hardware. If anybody has any additional information, or wants to help clear up some things in this blog post (which I will happily update), feel free to comment below.
