---
title: Beginning to understand the Ember.js Container
author: John McDowall
layout: post
permalink: /beginning-to-understand-the-ember-js-container/
dsq_thread_id:
  - 3009123168
categories:
  - The Container
tags:
  - core
---
I love Libraries. I&#8217;ve always loved them. Vast storehouses of knowledge, and thanks to the [Dewey Decimal Classification][1], storehouses where things can very easily be found. Here on the Island, we have a tiny library and thanks to the convention used to store the books, I will be able to find the same book here on the Island or in Vancouver using the same lookup system, in the same locations. It is the original convention over configuration, as previously libraries would order their books based on the order of acquisition meaning the same book would be hard to find in different libraries.

When we built web apps back in the Old World, it was easy to not have to think about our objects because the Page request-response cycle meant that mostly everything we relied on – controllers, views etc – was thrown away and rebuilt on the server from scratch with every request.

With client side apps, we&#8217;ve done away with the traditional request-response cycle, so the constructs we need to have a working app, those controllers, views, models and whathave you, need a home because they&#8217;re not going anywhere. They need a home in the client, with an address, where they can be looked up and told &#8216;Hey. It&#8217;s time to do your thing again&#8217; and they&#8217;ll still have whatever state they need to complete their task.

This is the core reason behind the existence of the Ember.js Container.

<!--more-->

Ember will do a lot of auto-generation of things in your App. Where should it place these newly created objects? How are other objects within the system meant to be able to find these new objects? The the first part of the answer is, you guessed it, the Container! The second part of the answer is: a consistent namespacing scheme.

## Commence tenuous Condo Building Analogy

Think about the Container as a Condo building. A very special Condo building. Controllers live in the Penthouse. Other things live in the Condo, but lets focus on Controllers for now.

When a new Controller moves in, the concierge says: &#8220;Oh hey, you&#8217;re a Controller. You live in the Penthouse and your new address is &#8216;controller:new_controller'&#8221;. A bunch of other Controllers move in that day, and each get similar addresses. This addressing scheme is a convention, and is consistent. All Controllers get an address starting &#8216;controller:&#8217;.

Anyone looking for a Controller simply has to know the addressing convention, and the Concierge can tell them where to find it.

Ok, I agree this analogy is a bit of a stretch <img src="http://ember.zone/wp-includes/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />

But essentially, the two things you need to know about the Container are:

  * It&#8217;s a place where long lived objects are placed so they can be looked up later, and the exact same instance is returned every time (unless you configure it otherwise, but we&#8217;ll get to that later)
  * It uses a consistent namespace based naming scheme to allow things in the container to be discoverable.

## Putting things in the Container

In Ember, it is verboten to look up the Application container by hand. You can (currently) easily access the Container by simply using the `__container__` variable on your main Application object. But you shouldn&#8217;t do this.

Instead, you should be able to reference `this.container` on whatever object you&#8217;re currently in ***that has come from the Container*** – typically Controllers, Views and the like. This works because when something is put in the Container, the Container itself adds a variable to the thing being added called `container`. Magic, eh?

There&#8217;s a shortcut method on your Ember Application called `register` that allows you to place something in the container at a given address. For example, given a Controller called `Session` that isn&#8217;t loaded by any particular Route:

    App.register('services:session', App.Session);


Has now placed the Controller in the Container at the address &#8216;services:session&#8217;. This particular case sounds like something that should be done at application boot, which makes it a perfect candidate for an [Application Initializer][2], which is even easier as one of the two parameters passed into an Initializer is the application, making the above code become

    application.register('services:session', App.Session);


## Looking things up in the Container

Ok, so you&#8217;ve made an Initializer, injected your `Session` controller into the container, now what? Well, wherever you need to use the `Session` object you could look it up like so:

    var session = this.container.lookup('services:session');


And that will work. It&#8217;s literally all you need to do. But there&#8217;s a better way.

## Dependency Injection – all the Kids are doing it

Back in your Initializer, just after you register the `Session` object, you have an opportunity to **inject** that object into other objects in the container. You&#8217;d do it, like so:

    container.injection('controller', 'session', 'services:session');


Whoa, now! What&#8217;s all this?

Well, the first parameter is the type of object you want to inject the `Session` instance into. It&#8217;s `controller` because we want to make this `Session` object available in all other Controllers.

The second parameter is the variable name that the Container will create in each of the targeted objects to inject into. So in this case, it&#8217;s going to creates a `session` variable on every Controller in the Container.

The third parameter is the address of the actual thing we want to inject, in this case it&#8217;s `services:session` because that&#8217;s the address we used to register our `Session` object earlier.

And with all of that, the Container will create a `sesssion` variable on every Controller in the Container, that has a reference to the `Sesssion` instance.

Now, in any controller you can say:

    this.get('session');


And get that instance to the Session controller you registered, and operate on it as you would any other container.

If, for some reason, you didn&#8217;t want the same instance of the `Session` object each time you looked it up – otherwise known as a **singleton instance**, when you registered the object with the container, you can pass an optional parameter like so:

    application.register('services:session', App.Session, {singleton: false} );


And this will tell the Container to create a new `Session` instance every time it is looked up.

## Summary

The Container is the keystone of your entire Ember Application. Without it, life would be very difficult indeed, as we as developers would have to manually do the bookkeeping necessary for where any objects we need have to be. Ember uses the Container to do all of this for us, and for itself.

 [1]: http://en.wikipedia.org/wiki/Dewey_Decimal_Classification
 [2]: http://ember.zone/ember-application-initializers/