---
title: Ember Application Initializers
author: John McDowall
layout: post
permalink: /ember-application-initializers/
dsq_thread_id:
  - 2927009763
categories:
  - Ember Infrastructure
---
Ember gives you a powerful mechanism to get you application set up, as well as a way to nicely modularize any tricky things like setting up Websockets.

<!--more-->

## A Basic Application Initializer

In the last post, [Handling Environment Data with Ember.js][1], we used an Initializer to read and cache environment variables that your App needs to know about. The basic form of an Initializer is:

{%highlight javascript linenos%}
Ember.Application.initializer({
  name: "initializerName",

  initialize: function(container, application) {
    ... your code ...
  }
});
{% endhighlight %}

As you can see, you start off by giving your Initializer a name. This name allows you to later specify an order for the execution of your initializers by specifying either or both of `before` or `after` options, like so:

{%highlight javascript linenos%}
Ember.Application.initializer({
  name: "configReader",
  before: "websocketInit",

  initialize: function(container, application) {
    ... your code ...
  }
});
{% endhighlight %}

{%highlight javascript linenos%}
Ember.Application.initializer({
  name: "websocketInit",
  after: "configReader",

  initialize: function(container, application) {
    ... your code ...
  }
});
{% endhighlight %}


This will ensure that the `configReader` Initializer runs before the `websocketInit` Initializer.

## Accessing the container and application instance

As you can see, the Initializer is passed both the container and application instance. This allows you to look up things you will most likely want to use, like the `Adapter` or `Store`:

    adapter = container.lookup('adapter:application')
    store = container.lookup('store:main')


You could also look up or set variables on your Application instance. For example, assuming you had run the environment config reader from the previous post first, in subsequent Initializers you can look up those environment variables:

    userId = application.get('env.userId')


## Asynchronous Initializers

If your Initializer performs a record lookup, then it&#8217;s vital that the Initializer simply doesn&#8217;t proceed allowing other Initializers to execute until that record lookup is fulfilled. You can tell Ember not to boot up the Application and start routing by utilizing the `deferReadiness` and `advanceReadiness` methods on the Application instanced passed to the Initializer.

For example, here&#8217;s an Initializer that looks up the Current user from the store, and injects it into the container, making it easily available in `Routes` and `Controllers`:

{%highlight javascript linenos%}
Ember.Application.initializer({
  name: "currentUserLoader",
  after: "store",

  initialize: function(container, application) {
      // Wait until all of the following promises are resolved
      application.deferReadiness()

      container.lookup('store:main').find('user', 'current').then( function(user) {
        // Register the `user:current` namespace
        container.register('user:current', user, {instantiate: false, singleton: true});

        // Inject the namespace into controllers and routes
        container.injection('route', 'currentUser', 'user:current');
        container.injection('controller', 'currentUser', 'user:current');

        // Continue the Application boot process, allowing other Initializers to run
        application.advanceReadiness();
      });
   }
});
{% endhighlight %}


## Summary

Ember Application Initializers are a great tool for squaring away all of those pesky tasks that need to be performed to get your App ready to run. Here&#8217;s a list of possible use cases:

  * Injecting the current user into all `Routes` and `Controllers`
  * Setting up other Javascript libraries, like Socket.io
  * Reading environment configuration data
  * Setting a CSRF token to be used on all Ajax requests
  * Pre-loading records written to the HTML DOM into the Store for faster initial loading.

Are you already using Ember Application Initializers for something else not in the list above? Let me know in the comments below!

 [1]: http://ember.zone/handling-environment-data-with-ember-js/