---
layout: post
title: "Converting to Attribute Routing in WebAPI Applications"
date: 2015-03-24
modified:
categories: blog
excerpt:
tags: [webapi, asp.net, routing]
image:
  feature:
share: true
comments: true
---
As I am working on my .NET application (sorry! no project page for this one), I find a great deal of freedom in being able to specify everything how I want it to be. Of course, these days, modern technologies such as WebAPI and ASP.NET MVC leverage the principle of [convention over configuration](http://en.wikipedia.org/wiki/Convention_over_configuration). While I generally agree with this idea precisely because it has made my life as a developer easier, I have never really liked the convention-based routing mechanism. For example, when I first set up my application pipeline, I defined my `HttpConfiguration` as such:

{% highlight c# %}
var webApiConfiguration = new HttpConfiguration();
webApiConfiguration.Routes.MapHttpRoute(
    name: "DefaultApi",
    routeTemplate: "{controller}/{id}",
    defaults: new { id = RouteParameter.Optional });
{% endhighlight %}

And that is it for the basic routing. At this point, the names of my classes will determine the routes. So if I create `PersonController` with an action of `public Person GetPerson(int id)` then there will be a route that corresponds to that point in my code like: http://www.example.com/Person/5.

This, of course, is easy, but it is limited in some of its capabilities. I want my routes to sit right alongside my controller actions. I want the ability to version my API. I also want the ability to create some really funky looking routes which make sense to the users of my API, but are not so clean when trying to generically define a route.

Attribute routing is the way to do this. Attribute routing uses attributes on the controller actions to define the specific route. A controller using attribute routing may look something like this:

{% highlight c# %}
[Route("person/{id}")]
public Person GetPerson(int id) { ... }
{% endhighlight %}

Now, I can hit http://www.example.com/person/5 and it would hit that action and return a Person to me. There is [a lot that can be done with attribute routing](http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2), and I'll let you follow the link if you wish to read more. However, I specifically wanted to discuss what I had to do to convert my actions that were returning `CreatedAtRoute` from my actions where new objects were being created. Originally, my action looked something like this.

{% highlight c# %}
[HttpPost]
[ResponseType(typeof(Person))]
public IHttpActionResult PostPerson(Person person)
{
    // ... do stuff to create and save new Person ...

    return CreatedAtRoute("DefaultApi", new { id = person.Id }, person);
}
{% endhighlight %}

But, wanting to use attribute routing, I switched my WebPipeline to the following (snipping portions that don't matter):

{% highlight c# %}
var config = new HttpConfiguration();
config.MapHttpAttributeRoutes();
{% endhighlight %}

Now I don't have a "DefaultApi" to create my route! I had to make the following updates to my PostPerson and GetPerson actions (as well as any other actions). Please note the extra attributes.

{% highlight c# linenos %}
[HttpPost]
[Route("api/person")]
[ResponseType(typeof(Person))]
public IHttpActionResult PostPerson(Person person)
{
    // ... do stuff to create and save new Person ...

    return CreatedAtRoute("GetSinglePerson", new { id = person.Id }, person);
}

[HttpGet]
[Route("api/person/{id}", Name = "GetSinglePerson")]
[ResponseType(typeof(Person))]
public IHttpActionResult GetPerson(int id)
{
    Person person = repository.GetById(id);
    if (person == null)
    {
        return NotFound();
    }

    return Ok(person);
}
{% endhighlight %}

Notice how, in my PostPerson action, I reference the name that is being used in the GetPerson Route. That's all there is to it! So, with a quick switch, all my routes now are attribute-based, give me a lot more power over my routes, and are much clearer.
