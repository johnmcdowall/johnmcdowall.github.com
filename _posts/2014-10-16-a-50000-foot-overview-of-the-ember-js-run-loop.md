---
title: A 50,000 foot overview of the Ember.js Run Loop
author: John McDowall
layout: post
permalink: /a-50000-foot-overview-of-the-ember-js-run-loop/
cover_image: /images/posts/typhoon-vongfong-iss.jpg
dsq_thread_id:
  - 3123537952
categories:
  - ember-core
---
The Ember Zone HQ is currently being assailed by the remnants of the Super Typhoon Vongfong. Look at that thing (image courtesy of [Reid Weisman][1], Astronaut on the ISS), with its whirling loop of chaos in the middle, the eye, which is apparently 80KM wide. The eye is calm, surrounded by the most intense and fastest winds, with the speeds decreasing the further you are from the eye. Ember has its own whirling pseudo-loop of controlled chaos. Let&#8217;s look at the strangely named Ember Run Loop.

<!--more-->

At the core of Ember beats the heart of an Accountant. An Accountant who sits at a large desk with six inboxes on it, and the inboxes are always processed in the same order. Items in a given inbox are processed in full before moving onto the next inbox. Sometimes processing a particular inbox will cause items to be added to a previous inbox, and the whole process immediately starts again from the first (highest priority) inbox with unprocessed items in it. This whole routine is performed in order that Ember can take advantage of batches of like things, coalesce them together, and minimize the amount of subsequent work that might otherwise need to be done. This is one of the core features of Ember that makes it fast, even without HTMLBars.

## Ember&#8217;s six inboxes

We can have Ember show us its secret inboxes, or more correctly **queues**, at the developer console:

    > Ember.run.queues
    ["sync", "actions", "routerTransitions", "render", "afterRender", "destroy"]


### The Sync Queue

This is the queue that settles all binding updates, and has the highest priority. You&#8217;re almost never likely to schedule something in here yourself. This is the place of the fast winds that surround the calm eye of Ember&#8217;s hurricane.

### The Actions Queue

This queue contains things that should be processed after all of the binding updates have been synchronized and before any Views are rendered. This is the general workhorse queue in Ember, and is the place where any Promises will get sent to. Any call to `Ember.run` will place the item in this queue.

### The Router Transitions Queue

Anything in your Ember Application that signals a Transition should happen will end up in here. You&#8217;re almost never likely to schedule something in here yourself.

### The Render Queue

This queue contains items that cause changes in the DOM, and is mostly fed from items that were processed in the Actions queue. You&#8217;re almost never likely to schedule something in here yourself.

### The AfterRender Queue

Sometimes you might need to update a DOM element that is created as part of processing an item that would have rendered in the Render queue. You would use the `Ember.run.schedule('afterRender', function() { ... })` call to schedule your function to be run in this queue, after Ember has rendered all of its DOM changes. This is probably the queue you&#8217;ll deal with the most, as you use it to help schedule jQuery plugins attaching to DOM elements, for example.

### The Destroy Queue

The Destroy queue is pretty much as it sounds &#8211; the queue where things that should be destroyed (Views that are no longer shown, etc) are placed, and processed last, and has the lowest priority. You&#8217;re almost never likely to schedule something in here yourself, either. At this level, we&#8217;re at the slow moving winds that dump fat rain drops onto small islands.

## Scheduling things on the Queues

The two main methods you&#8217;ll probably need to use are `Ember.run.next` and `Ember.run.schedule`. The `next` function is used to schedule something to happen in a completely new run loop, that will happen some time after (you don&#8217;t know when) the current run loop has cleared. The `schedule` function is for when you want to schedule something to be processed in the current run loop, on a particular queue. If you&#8217;re certain that the thing you want to schedule should only be scheduled once, then use `scheduleOnce`.

A lot of common advice is to use `Ember.run.schedule` and the `afterRender` queue inside places like `didInsertElement` when handling jQuery plugins and so on, and it is almost always the correct default. That said, I&#8217;ve had instances where scheduling a jQuery plugin hook inside the current run loop&#8217;s `afterRender` queue just wouldn&#8217;t work, and indeed needed to be scheduled in the next run loop using `next`.

When you use `next`, because it schedules the item for the next run loop, which will run an indeterminate amount of time after the current run loop, you may encounter CSS issues with sudden flashes, or document re-flowing. This is usually an indicator you should be using `scheduleOnce` on the `afterRender` queue.

## Day to day use of the Ember Runloop

Chances are you&#8217;ll never need to directly invoke placement of a function on any of the Run Loop&#8217;s queues, but the most common scenario where you might need to is the `afterRender` queue because you want to manipulate a DOM element that Ember has rendered as part of a `View` or `Component`. Usually, you&#8217;ll be doing this because you wish to use some jQuery provided effect or third party component.

## Summary

The Ember Run Loop is a powerful mechanism that allows Ember to minimize the work it needs to do to keep bindings up to date, propagate changes, and reduce the amount of DOM manipulations that need to happen. Hopefully this article has given you the 20% of the information you&#8217;ll need for 80% of the time.

For the other 20% of the time, you can also look at the following excellent resources for more information:

[The Ember Run Loop handbook][2]

[Backburner.js and the Ember Run Loop by Erik Bryn][3]

[Everything You Never Wanted to Know About the Ember Run Loop][4]

 [1]: https://twitter.com/astro_reid
 [2]: https://github.com/eoinkelly/ember-runloop-handbook
 [3]: http://talks.erikbryn.com/backburner.js-and-the-ember-run-loop/
 [4]: http://alexmatchneer.com/blog/2013/01/12/everything-you-never-wanted-to-know-about-the-ember-run-loop/
