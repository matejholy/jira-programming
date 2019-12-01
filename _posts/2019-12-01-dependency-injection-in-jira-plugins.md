---
layout: post
title:  "Dependency injection in Jira plugins"
description: "How to use dependency injection using Atlassian Spring Scanner."
categories: main
tags: jira, server, dependency injection, spring scanner
comments: true
---

Do you need to simplify your code by automatically injecting required instances into your objects? Dependency injection is the answer.

*These instructions have been tested in JIRA 7.13.9 with Atlassian Spring Scanner 1.2.13.*

Jira uses the Atlassian Spring Scanner that makes this quite easy. The best place to learn about it is the [readme file of the Atlassian Spring Scanner Bitbucket repository](https://bitbucket.org/atlassian/atlassian-spring-scanner/src/1.2.x/).

Let's look at an example. If we have an interface named `Helper` and an implementing class `HelperImpl`, all we need to do to make it available for injection, is to use `@Named` annotation on the implementing class:

{% highlight java %}
import javax.inject.Named;
import com.example.jira.plugin.api.Helper;

@Named
public class HelperImpl implements Helper {

  @Override
  public String sayHello() {
    return "Hello.";
  }

}
{% endhighlight %}

To have the `Helper` implementation injected into another class (in this case a REST resource), we'll annotate it's constructor with `@Inject`:

{% highlight java %}
import javax.inject.Inject;
import com.example.jira.plugin.api.Helper;
// ... rest of the imports

/**
 * A resource of message.
 */
@Path("/message")
public class MyRestResource {
    // store Helper in private property
    private Helper helper;
    
    @Inject
    public MyRestResource(Helper helper) {
      this.helper = helper;
    }
    
    @GET
    @AnonymousAllowed
    @Produces({MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML})
    public Response getMessage()
    {
      // return the message from the Helper implementation
      return Response.ok(new MyRestResourceModel(helper.sayHello())).build();
    }
}
{% endhighlight %}

The example above works when injecting an object from the same plugin. If you would like to inject an object from Jira itself (or another plugin), add the `@ComponentImport` annotation to the constructor parameter.

For example, if we want to use the `ApplicationProperties` in our class, we can do it like this:

{% highlight java %}
import javax.inject.Inject;
import com.example.jira.plugin.api.Helper;

import com.atlassian.plugin.spring.scanner.annotation.imports.ComponentImport;
// ... rest of the imports

/**
 * A resource of message.
 */
@Path("/message")
public class MyRestResource {
    // store Helper in private property
    private Helper helper;
    // store ApplicationProperties
    private ApplicationProperties applicationProperties;
    
    @Inject
    public MyRestResource(Helper helper, @ComponentImport ApplicationProperties applicationProperties) {
      this.helper = helper;
      this.applicationProperties = applicationProperties;
    }
    
    // rest of the code
}
{% endhighlight %}

And that's all. Just use `@Named` on your components, `@Inject` on constructors that use your components and `@ComponentImport` when importing stuff outside your plugin.