---
layout: post
title: "Master’s Degree is Finished (...and so is my Arduino Project...)"
date: 2011-04-19
modified:
categories: blog
excerpt:
tags: [arduino, graduate-school, real-time-systems]
image:
  feature:
share: true
comments: false
---
While I have no illusions of grandeur with respect to how many people are (or are not) following this blog, I must apologize for the severe lack of updates to the few of you who do track its posts.

Over the past month, I have been very busy working on my last project before I earned my Master’s Degree in Software Engineering. I am happy to say that I presented it to my professor today and, as yesterday at 4:30PM, I am officially finished! (Disclaimer: Final grades are not yet posted.) While I did enjoy learning a wide variety of new topics while earning my degree, I am most definitely happy to have a great deal of my personal time back.

So, before I pack this project away, I wanted to share a bit about my it, and just summarize the key points. First, here is my Arduino microcontroller all wired up.

{:.center}
![Finished Arduino]({{ site.url }}/images/arduino-finished.jpg)

If you recall, the purpose of the project was to simulate a closed-loop injection system for patients who are afflicted with diabetes mellitus, more commonly known as diabetes. Diabetes is a disease the afflicts an estimated 25.8 million people (with only about 19 million of those individuals diagnosed). Within diabetes, there are two types of diabetes labeled, conveniently, Type 1 and Type 2. (Note: There are other types of diabetes, but for the purposes of the project, I focused specifically on these two types.) Type 1 is the rarer form of the disease, and exists when an individual’s body does not produce enough or any insulin. This type of diabetes is also known as childhood diabetes. Type 2 diabetes, on the other hand, afflicts approximately 90% of the individuals diagnosed with diabetes. In the case of Type 2, an individual’s body fails to respond to the insulin that is produced. Essentially, the body becomes immune to its own insulin.

Diabetes is a serious disease that, if not properly monitored and managed can lead to additional complications such as blindness, kidney and nervous system disease, decreased cardiovascular health, amputations, and, in extreme cases, diabetic coma and death. Despite the many advances in diabetes management, there still fails to exist a system which will not only monitor blood-glucose levels, but will also deliver insulin to that individual without any additional intervention. If the development of such a closed-loop system were to be developed, it could provide a safer, more consistent means of managing diabetes.

The purpose of this particular project was not to necessarily solve the problem of the closed-loop system. Instead, our professor knew that developing such a real-time system would present a number of challenges that we would have to solve. Since completing the project, I have a much greater level of respect for those individuals and companies who design such systems.

In order to actually use and test my closed-loop injection system, I required a patient. In this case, my “patient” was represented by a C# application that I developed. This application would calculate the current blood-sugar level per “TICK” of time (as defined by the Arduino). These calculations included things like eating (raises blood-sugar), taking insulin (automatic or manual; decreases blood-sugar), and the passing of time (slowly decreases blood-sugar). Because each diabetic individual has different needs, I knew that I had to focus on a single individual. So, I modeled blood-sugar changes for someone of my body type. This was all tracked in a bar graph that displayed the blood-sugar levels at each particular time step.

From the point of view of the closed-loop injection system, I had two main tasks – monitor the blood-sugar level, and deliver the insulin. Instead of describing how I met each requirement (if anybody is interested, I can link to the SRS), let me show something more interesting instead – source code for my Arduino microcontroller.

{% highlight java %}
/*
 * The main loop.
 *
 * The purpose of this background loop is to take
 * regular readings from the user (blood-insulin readings)
 * and store that data for use when the WDT requires it.
 * In addition, the background loop will check to make
 * sure that insulin levels are not running low. An
 * LED is turned on if this happens to remind the patient
 * to get their insulin refilled. */
void loop()
{
  if(Serial.available() > 0) {
    // Read into the buffer until the semicolon character
    // is read. When this happens, process the data.
    if(readUntil(Serial.read(), 59, buffer)) {
      // Immediate flush serial so that the blood value is
      // the most recent and there is no lag.
      Serial.flush();

      // Read in the blood level and clear the buffer.
      bs = strToFloat(buffer);
      buffer = "";

      // Check to make sure blood-sugar levels are not dangerously low.
      if(bs < THRESHOLD_LOW) {
        pPrintln("Please eat. Blood-sugar levels are dangerously low!");
      }
    }
  }

  // Status check - provide warning if the amount
  // of insulin in the pump is getting low.
  if(statusFlag == true && (basalAmount <= 5 || bolusAmount <= 5)) {
    statusFlag = false;
    pPrintln("Insulin well levels are running low.");
    pPrintln("Please refill as soon as possible.");
    digitalWrite(TIMER_PIN, HIGH);
  } else if(basalAmount > 5 && bolusAmount > 5) {
    digitalWrite(TIMER_PIN, LOW);
    statusFlag = true;
  }
}
{% endhighlight %}

The Arduino microcontroller repeatedly runs the main loop (also known as the background loop). While I was developing the project, I did run into a case of starvation. Originally, I had used the background loop to flash an LED to show that the system was working properly. However, by doing this, it prevented the readUntil function from keeping up with the input coming across serial (which was the user’s blood-sugar level). By taking the blink function out, the problem fixed itself and the closed-loop injection system coincided with the simulation.

Here is another interesting function:

{% highlight java %}
/*
 * Perform a manual check of the user's insulin levels.
 * This particular function delivers bolus insulin
 * rather than the basal which is delivered by the
 * automated system.
 *
 * NOTE: Do not use pPrintln within this interrupt as
 * pPrintln reenables interrupts upon finishing its call.
 * Because this interrupt handler already disables interrupts,
 * Serial.println should be used instead.
 */
void manualCheck() {
  cli();
  if(manualBlock == false) {
    Serial.println("Performing manual check...");
    if(bs > THRESHOLD_HIGH && bolusAmount > 0) {
      Serial.println(BOLUS);
      bolusAmount = bolusAmount - 1;
    } else if(bolusAmount == 0) {
      Serial.println("DANGER: Your insulin well is empty. Fill up immediately or seek medical assistance!");
    }

    // Put up blocks so new insulin isn't delivered too soon.
    deliveryTime = DELIVERY_TIME;
    manualBlock = true;
  }
  sei();
}
{% endhighlight %}

This function is what is called when the user presses the button to trigger a manual check of their blood-sugar. One of the issues I ran into here was ensuring that the manual delivery couldn’t happen immediately before or after the automatic delivery in order to prevent an overdose of insulin. In addition, I also had to make sure I debounced the digital input from the button. I did this using the [Bounce library which is available at the Arduino playground](http://www.arduino.cc/playground/Code/Bounce). (Note: I do not show how to use this library in the code above. It is very straightforward to use, however.) The last issue I ran into with the manual interrupt was that I did not want the WDT interrupting its functionality. You’ll notice the calls to cli() and sei(). Both these functions are provided by the [Arduino AVR library](http://www.arduino.cc/playground/Main/AVR). I also had a helper function – pPrintln (protected println) – which also disabled interrupts when it was printing. This would prevent the WDT from interrupting serial printing. However, I quickly learned that if I called pPrintln from within the manualCheck function, the WDT could interrupt my manual interrupt.

The last interesting piece of functionality occurs using the Watchdog Timer (WDT). In my last blog post, I discussed how to access and use the WDT as a timing mechanism (as opposed to using a 5x5x5 timer, or some other similar clock device). After figuring out how to access the WDT, I had no real difficulties in getting it to work.

{% highlight java %}
/*
 * Watchdog Timer Interrupt
 */
ISR(WDT_vect)
{
  // Decrement the timer.
  // Send the message to indicate to the simulation to calculate
  // for the next value of blood sugar.
  deliveryTime--;
  pPrintln(TICK);
  if(deliveryTime == 0) {
    // Check the value against the threshold.
    // If the blood sugar is low, provide the basal
    // insulin.
    if(bs > THRESHOLD_HIGH && basalAmount > 0) {
      pPrintln(BASAL);
      basalAmount--;
    } else if(basalAmount == 0) {
      pPrintln("DANGER: Your insulin well is empty. Fill up immediately or seek medical assistance!");
    }

    deliveryTime = DELIVERY_TIME;
    manualBlock = false;
    Serial.flush();
  }
}
{% endhighlight %}

Of course, I am not showing my entire application – I am primarily sticking to the interesting functions and areas where I either ran into some difficulties or had to spend a little extra time to understand the system.

All-in-all, this was the most interesting project I worked on within my graduate program. I hadn’t had any experience with real-time systems before this, and the project gave me an excellent opportunity to play a little with hardware and debug some of the problems (specifically timing problems) that real-time systems designers run into every day.

Now that I am finished with this project, I plan to return to my personal side-projects. Keep an eye open for posts regarding Android and Silverlight in the upcoming months.
