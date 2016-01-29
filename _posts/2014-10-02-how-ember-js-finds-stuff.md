---
title: How Ember.js Finds Stuff
author: John McDowall
layout: post
permalink: /how-ember-js-finds-stuff/
dsq_thread_id:
  - 3076343748
categories:
  - Ember Infrastructure
---
There are several small satellite Islands that surround the main Island that I live on, and believe it or not seem to be fairly reasonably populated. The one depicted in the photo above reputedly has a retired Doctor living on it, who owns a small cannon which he fires off while dressed in full sea-captain regalia. I&#8217;ve definitely heard the cannon, but never seen the Doctor with my own eyes&#8230;yet. I am curious as to how these folks receive their mail, though. What would their address be? Well, this led me to think about how Ember finds the things it needs.

<!--more-->

Following on from my previous article about [Beginning to understand the Ember.js Container][1], I used a metaphor of a &#8216;concierge&#8217; that was able to look things up when needed. Today I&#8217;m going to look at that Concierge and see how it is that they do their job.

## The Return of the Concierge (metaphor)

In Ember, the Concierge is the person responsible for taking the requested name that we want to look up, in this case let&#8217;s say &#8216;controller:index&#8217;, and translating that an actual object that we can use. In Ember land, this person is called the `DefaultResolver` and its only purpose is to provide that necessary lookup and translation service of string based names to real things.

It&#8217;s worth noting that there are actually two completely different Resolvers kicking around at the moment: one that ships with stock Ember, the `DefaultResolver`, and the other is Stefan Penner&#8217;s [ember-resolver][2] project which is the foundation of how Ember-CLI performs its module based magic. We&#8217;re only going to look at Ember&#8217;s stock Resolver today.

Previously we saw that we could register an object to be found at a particular &#8216;address&#8217; using the following syntax inside an [Initializer][3]:

    App.register('services:session', App.Session);


This creates a link between the string &#8216;services:session&#8217; and our `App.Session` object. When we or Ember needs to perform a lookup on the `Container` for an object, the `Container` is internally going to use the `DefaultResolver` to do the reverse lookup of the `string` name we supply to get the actual object.

## Finding things not in the Container

The `Container` isn&#8217;t used to store everything in Ember. Templates, for example, live in the `Ember.TEMPLATES` variable, which isn&#8217;t stored inside the `Container`. This is where the `DefaultResolver` steps in again to help locate what Ember needs.

The `DefaultResolver` has several methods for looking up `Routes`, `Models`, `Helpers`, `Templates`, `Controllers` and `Views`, and all of them can be overridden. It is through the `DefaultResolver` that the `Container` is able to provide a unified lookup service to the rest of the system.

Let&#8217;s say you wanted to have all of your Modal dialog templates located under a `/modals` subdirectory of `/templates`. If you try to do that without overriding the `DefaultResolver`&#8216;s basic implementation of how Templates are looked up in `Ember.TEMPLATES`, Ember won&#8217;t be able to find the modal templates. Here&#8217;s how you could provide your own implementation that looks to see if a requested template ends in the word &#8216;modal&#8217;, and if it does, add the `/modals` prefix to the lookup path:

{%highlight javascript linenos%}
window.App = Ember.Application.create(
  Resolver: Ember.DefaultResolver.extend({
    resolveTemplate: function(parsedName) {
      var modalSuffix = 'modal';
      var templateName = parsedName.name;

      // If the template ends in 'modal' then we alter the lookup path.
      if( templateName.indexOf(modalSuffix, templateName.length - modalSuffix.length) !== -1 ){
        return Ember.TEMPLATES['modals/'+templateName];
      }

      // Call the default implementation for all the other templates.
      return this._super(parsedName);
  })

)
{%endhighlight%}

In the above example, we&#8217;ve replaced the stock Resolver that ships with Ember without our own, which extends the basic `DefaultResolver` and overrides the `resolveTemplate` hook to implement the Modal template lookup functionality we need. Maybe you&#8217;d also like to override how a Modal&#8217;s `Controller`s are also looked up to match the same folder convention. It&#8217;s possible using this technique.

## Summary

The `DefaultResolver` does a good job and is a main workhorse in Ember. Knowing what it does and how to tweak it as necessary can be very useful for establishing some of your own conventions, but use it with care.

 [1]: http://ember.zone/beginning-to-understand-the-ember-js-container/
 [2]: https://github.com/stefanpenner/ember-resolver
 [3]: http://ember.zone/ember-application-initializers/