---
layout: post
published: true
title: Ember Application Initializers
description: Use Ember Application Initializers to clean up your application code. 
---

## Ember Application Initializers

Ember Application Initializers allow you to register blocks of code that execute when your Ember application is initializing. With an Initializer, you could access Ember's underlying object container, grab all the controllers and inject other objects into them, register global DOM ```onReady``` event blocks or anything that is useful to setting up your application.

Fundamentally, the Initializer requires a ```name``` attribute and an ```initialize``` method. 

A basic Application Initializer looks like this:

```javascript
Ember.Application.initializer({
  name: "initializerName",
 
  initialize: function(container, application) {
    ... your code ...
  }
});
```

Initializers can also specify dependencies by using a ```after``` attribute which references the name of the Initializer it should run after. 

### Some example use cases
Here's some examples of using Application initializers to clean up repeated code, or give code that would otherwise run outside of Ember a home inside Ember. 

#### Looking up objects in the Container &amp; Injecting Types

You can look up objects in the Container inside the ```initialize``` method like so:

```javascript
store = container.lookup('store:main')
```

Then you could inject a controller into all other loaded controllers in the container:

```javascript
  controller = container.lookup('controller:currentUser').set('content', user)
  container.typeInjection('controller', 'currentUser', 'controller:currentUser')
```

#### Defining global DOM ```onReady``` blocks

If you need code to be executed when the DOM is ready and want a clean place to integrate it into your Ember app, you can use an initializer. This example loads a current-user attribute from a ```META``` tag in the document. 

```javascript
Ember.Application.initializer({
  name: "initializerName",
 
  initialize: function(container, application) {
    $(function(){
      /* Look up an attribute in a meta tag */
      attributes = $('meta[name="current-user"]').attr('content')

      /* Do something with it */
    });
  }
});
```

If you're using Rails, this would be a good way to get the CSRF token and use it in the jQuery AJAX setup. 

#### Pre-loading records

One way to make the initial Ember application experience even faster is to render some records as JSON strings into the HTML document body. When your Ember application starts up, you can load those records from the document and create instances from them, thus saving an initial AJAX call to fetch the records. 

### Further Reading 
I assembled this knowledge from a rag-tag collection of blog posts and gists, namely this Gist by [Ian Vanderbyl](https://gist.github.com/ivanvanderbyl/4560416) and this blog post by [Alexander Zaytsev on using Ember with Rails and Devise](http://say26.com/using-rails-devise-with-ember-js).

This only scratches the surface of what could be possible with Ember Application Initializers. They are poorly documented at the moment, and could change their behavior at any time before the 1.0 release, but until that happens use them wisely.  