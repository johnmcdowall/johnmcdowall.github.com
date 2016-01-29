---
title: Ember Works Best When You Tell Instead Of Ask
author: John McDowall
layout: post
permalink: /ember-works-best-when-you-tell-instead-of-ask/
dsq_thread_id:
  - 3053260265
categories:
  - Uncategorized
---
Welcome to another note from the Ember Zone. This week on the Island we&#8217;ve had a couple of days of really torrential rain, and sunsets like the one above. On one hand this is good because it replenishes the reservoir that we all get our drinking water from. On the other hand it shakes and knocks down all of the leaves from the trees that were ready to fall, which admittedly does create a pretty colourful carpet everywhere. The wind, to the rain, to the leaves, to my driveway. Nature&#8217;s binding system at work.

This started me thinking about how Ember really excels when you take advantage of the fact that the optimal state of things when using Ember is to Tell, don&#8217;t Ask.

<!--more-->

&#8216;Tell don&#8217;t Ask&#8217; is a common software heuristic. The distinction is that procedural code ask lots of questions about what it should do next, whereas Object Oriented code relies on telling other objects what to do at the right time. It&#8217;s a subtle, but powerful inversion of obvious thinking.

One of the most powerful aspects of Ember is that it has this concept baked in from the ground up in the form of the Bindings system.

From [the guides][1]:

> A binding creates a link between two properties such that when one changes, the other one is updated to the new value automatically.

By virtue of creating a Binding, we are allowing the bound object to automatically tell interested parties when something has changed, they don&#8217;t have to ask by repeatedly polling, or relying on a custom event system.

Any time you are reaching into another object to query the state of some property, you should re-write it as a Binding and take advantage of &#8216;Tell, don&#8217;t ask&#8217;. For example, instead of:

    // Assuming the controller has `someController` in its `needs`.
    this.get('controllers.someController.someProperty');


Create a Computed Property using the `alias` macro on the object that needs to know the information like so:

    someProperty: Ember.computed.alias('controllers.someController.someProperty')


The first example involves &#8216;asking&#8217;, and you might think that the second example also involves asking, because you&#8217;re setting the binding in the object that needs to know what&#8217;s going on. Turn that thinking around! When `someController.someProperty` changes, our own `someProperty` will change in kind. It&#8217;s implicitly told what to do via the Bindings system.

We can now create a new property based on our own bound property to further enhance the local state:

    criticalCondition: function() {
      return this.get('someProperty') > 80;
    }.property('someProperty')


**Update: This was an Observer before, but @locks pointed out it should be a Property because it doesn&#8217;t contain any behaviour, and rightly so. Thanks!**

Once again, `criticalCondition` will automatically update when the bound property all the way over in `controllers.someController.someProperty` changes, and we won&#8217;t have to do a damn thing.

## Summary

Ember&#8217;s Bindings and Computed Properties system is probably the most powerful thing in the entire framework. It allows for the same experience you might get when editing a spreadsheet – change the value in cell A2, and a bunch of other cells update – in your daily programming.

In short, whenever you feel yourself writing a long property path chain in a `get`, flip it around to use the Bindings system with something like a Computed proeprty.


 [1]: http://emberjs.com/guides/object-model/bindings/