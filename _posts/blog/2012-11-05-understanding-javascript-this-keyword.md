---
layout: post
title: 'Understanding JavaScript "this" Keyword'
date: 2013-03-30
modified:
categories: blog
excerpt: "A little bit of that about this."
tags: [javascript]
image:
  feature:
share: true
comments: true
---
I know there are a number of blogs and articles on `this` particular topic (no pun intended) and it only takes a quick Google search to find the information on how JavaScript’s `this` keyword actually works.

However, since `this` blog was primarily started so I could better understand my craft, I’m going to just add another article to the mix because if I can explain a topic, then I understand it. (Yeah…sorry…it’s all about me.)

The reason I even decided to write `this` article in the first place is I had never really run across problems with the `this` keyword. In fact, I will admit that I had mistakenly always thought of `this` to operate how it would as if I were working in Java. Unfortunately (or fortunately, once you understand it), `this` was not the case, and I ran into an issue that was non-obvious to me until I started doing some research. So, I now present to you the findings of my research and my better understanding of JavaScript’s `this` keyword.

Because `this` is a blog about code, let’s start with an example where I created a simple Contact JavaScript object.

{% highlight javascript %}
var Contact = {
  firstName: "Jason",
  lastName: "Cavett",
  email: "example@example.com",
  display: function () { return this.firstName + " " + this.lastName + " email is: " + this.email; }
};

Contact.display();
{% endhighlight %}

Fairly simple, right?

Note: In my particular situation, I was loading the objects using the [jQuery.getJSON()](http://api.jquery.com/jQuery.getJSON/) function and mapping to it with the [jQuery.map()](http://api.jquery.com/jQuery.map/) function. But, it still does not change the fact that I can access the various properties of the object through my display function. I am simplifying this example quite a bit because I did not want to distract from the main point of this blog post.

At this point, I decided that I wanted to create another type of contact which was a little more specific:

{% highlight javascript %}
var BusinessContact = {
  firstName: "John",
  lastName: "Doe",
  email: "johndoe@example.com",
  company: "Acme Corp.",
  phone: "555-555-5555",
  display: Contact.display,
  buisnessInformation: function() {
    return "Company: " + this.company + " - " Phone: " + this.phone;
  }
};

BusinessContact.display();
{% endhighlight %}

Pretty straightforward again, right?

Well, `this` is where things start to get a little tricky with `this` – particularly if you come from a Java background.

In my previous code example, you will get:

```
John Doe email is: johndoe@example.
```

In memory, `BusinessContact.display()` and `Contact.display()` reference the same object. But, the `this` keyword will be whatever object is before the period in the call. So, in the case of the `BuisnessContact`, it is the business contact information, and in the case of the Contact, it is the contact information.

This is why I had a particular problem with my code. I did something like this:

{% highlight javascript %}
var display = BusinessContact.display;
// Some code went here.
display();
{% endhighlight %}

So, there is nothing in front of the period here. What happens?

Well, technically, when there is no object explicitly used, there is the implicit, global object (also known as window – I believe named for the browser `window` in which the JavaScript would run). As a result, I get the following output:

```
undefined undefined is: undefined
```

And that is, of course, because there are no firstName, lastName, or email addresses defined for window. And, in actuality, I had saved a global variable called email which meant I got:

```
undefined undefined is: undefined
```

This is what started my entire confusion in the first place.

So, what can you do to get around this problem (pun very much intended)? Well, the first thing to do is realize that it’s not a problem. The ability for `this` to change allows for some very powerful uses in JavaScript.

This is where the `call` and `apply` become important. In short, you can call your function with either:

```
Contact.display.call(BusinessContact);
```

```
Person.display.apply(BusinessContact);
```

Both are very similar, but slightly different in how you pass in your arguments (if you have a generic function instead of the specific ones I am showing in this example). The different would be:

{% highlight javascript %}
Contact.display.call(BusinessContact, "FirstName", "LastName", "email");
Contact.display.apply(BusinessContact, ["FirstName", "LastName", "email"]);
{% endhighlight %}

This table shows the value of `this` depending on the syntax of the function call. (Thank you to [JavaScript Web Blog](http://javascriptweblog.wordpress.com/2012/04/09/javascript-fat-city/) for the table.)

Type | Syntax of function call | Value of `this`
--- | --- | ---
Method call | `myObject.foo();` |	myObject
Baseless function call | `foo();` | global object (e.g. window) (undefined in strict mode)
Using call | `foo.call(context, myArg);` | context
Using apply | `foo.apply(context, [myArgs]);` | context
Constructor with new | `var newFoo = new Foo();` | the new instance (e.g. newFoo)

So, as you can see, it’s very understandable as to why `this` gets so confusing – especially if you are used to it in another language. Hopefully this post helped you under `this` just a little bit better.
