---
layout: post
title: "Using Swashbuckle in ASP.NET Core 1.0"
date: 2016-05-31
modified:
categories: blog
excerpt:
tags: [asp-net-core, asp-net, swashbuckle, swagger]
image:
  feature:
share: true
comments: true
---
I spend a lot of time writing APIs in .NET. As part of that, I have to document my endpoints. Being the lazy developer I am, I always want this documentation generated for me so I never need to lift a finger after I add a new API.

In the past, [Microsoft's ASP.NET Web API Help Pages](http://www.asp.net/web-api/overview/getting-started-with-aspnet-web-api/creating-api-help-pages). While they were a bit ugly by default, and did not have [built in support for testing the API](https://blogs.msdn.microsoft.com/yaohuang1/2012/12/02/adding-a-simple-test-client-to-asp-net-web-api-help-page/), they worked and did the job that I asked of them.

Recently, I have been working very heavily in [ASP.NET Core 1.0](https://dotnet.github.io/). While it is a fast-moving project, and things are changing almost daily, it is a lot of fun and it feels good to work with. (It may also have something to do with the lightweight and familiar tooling that I am using while developing on OSX. I love that!) Unfortunately, the Web API Help Pages have not been ported to support ASP.NET Core. So, I went in search of another alternative.

Very quickly, I landed on [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle) which adds [Swagger documentation](http://swagger.io/), and a UI to browse said documentation, to Web API projects. It supports the testing of API endpoints, the model is easier to understand, and [it looks great](http://petstore.swagger.io/)! Everything I could want.

Of course, given that I'm working with ASP.NET Core, the documentation to specifically bring Swashbuckle into my project was a little lacking given the rapid changes happening to Core. So, I wanted to document what I found here. Please note that this is the way it is currently performed, and it may change again sometime in the future. Also note that I am doing this from Release Candidate 1 - I have not yet switched over to 2.

First, you must add the package reference to `project.json`. (Don't get me started on `project.json` not being a thing anymore...)

```
"Swashbuckle": "6.0.0-*"
```

Once you have done this, crack open `Startup.cs` and add the following configuration to the `ConfigureServices()` method:

~~~ csharp
// Add Swagger documentation.
services.AddSwaggerGen();
services.ConfigureSwaggerDocument(options => {
    options.SingleApiVersion(new Info {
        Version = "v1",
        Title = "My API",
        Description = "This is my API.",
        TermsOfService = ""
    });
});
services.ConfigureSwaggerSchema(options => {
    options.DescribeAllEnumsAsStrings = true;
});
~~~

Of course, configure schema options and other items as you see fit.

Then, also in `Startup.cs`, add these two lines to `Configure()`.

~~~ csharp
app.UseSwaggerGen();
app.UseSwaggerUi();
~~~

Go ahead and run your project! You should now be able to access your documentation at http://localhost:5000/swagger/ui/index.html. Of course, adjust the URL accordingly.

As an addition, I wanted to hide two of my APIs as they are not properly wired up. Per this Github issue, Swashbuckle is built on Web API's built-in metadata layer. This is beautiful because you can then use the ApiExplorerSettingsAttribute to ignore endpoints, like this:

~~~ csharp
[ApiExplorerSettings(IgnoreApi=true)]
public class MyController : Controller
{
  // ...
}
~~~

Note that you can use these on an entire controller, or on individual endpoints.

That's all there is to it! You now have beautiful documentation for all to use.
