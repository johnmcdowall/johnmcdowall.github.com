---
title: What the hell is Ember.K?
author: John McDowall
layout: post
permalink: /what-the-hell-is-ember-k/
dsq_thread_id:
  -
categories:
  - ember-core
---
Sometimes you&#8217;ll be looking through a code base, or even the Ember source code itself, and you&#8217;ll come across a weird looking snippet like this:

    onRegister: Ember.K


What is `Ember.K`, why is it being used and should you use it?

<!--more-->

## Spelunking the Ember.js source

Grab the source if you don&#8217;t have it.

I use [The Silver Searcher][1] for digging around code bases, and highly recommend it. Let&#8217;s run it against the Ember codebase, looking for `Ember.K`:

    ~/code/oss/ember.js master>  ag 'Ember.K'


You&#8217;ll get just shy of 100 lines of output back, showing you that `Ember.K` is used a fair amount throughout the codebase. For example:

    packages/ember-metal/lib/core.js
    196:Ember.K = K;
    201:if ('undefined' === typeof Ember.assert) { Ember.assert = Ember.K; }
    202:if ('undefined' === typeof Ember.warn) { Ember.warn = Ember.K; }
    203:if ('undefined' === typeof Ember.debug) { Ember.debug = Ember.K; }
    204:if ('undefined' === typeof Ember.runInDebug) { Ember.runInDebug = Ember.K; }
    205:if ('undefined' === typeof Ember.deprecate) { Ember.deprecate = Ember.K; }


So we&#8217;ve found it in use, but what is it doing? You might be able to deduce its purpose already, but lets take a look at the definition first.

## Finding the source

Let&#8217;s run The Silver Searcher again, but this time looking for the definition of Ember.K:

    ~/code/oss/ember.js master>  ag 'Ember.K = '
    packages/ember-metal/lib/core.js
    196:Ember.K = K;


It looks like the definition lives in `packages/ember-metal/lib/core.js`. Right enough, the definition is present:

    /**
      Empty function. Useful for some operations. Always returns `this`.

      @method K
      @private
      @return {Object}
    */
    var K = function() { return this; };
    export var K = K;
    Ember.K = K;


The comment is vague: &#8216;Empty function. Useful for some operations. Always returns `this`&#8216;. It&#8217;s a function that returns `this` &#8211; what use is that?

## Dissecting Ember.K and its usage

We know from our explorations that `Ember.K` returns `this`. Why would we want to do that?

You probably noticed in `core.js` definitions like:

    if ('undefined' === typeof Ember.assert) { Ember.assert = Ember.K; }


From this we can see that `Ember.K` is being used as a safe way to neutralize the side effects of any calls to `Ember.assert` when it is undefined. The line of code above essentially makes calls to `Ember.assert` &#8216;safe&#8217;, because in the worst case scenario `this` will be returned, which is guaranteed to be the containing object&#8217;s instance.

If we take a look at `packages/ember-views/lib/views/view.js` and search for `willInsertElement`, we can see that it also has been assigned `Ember.K`. In this instance, `Ember.K` is being used to safely create the hook that views provide to subclasses, again avoiding the caller experiencing an `undefined` value being returned as a result.

In this situation, `Ember.K` is also a truthy value, so that when the function result is used in a conditional, it evaluates as truthy, allowing the default hook to be called and provide a useful value inside the conditional.

You can use this technique when writing views or components, where you have written some template code that raises events, but you aren&#8217;t quite sure what those events should do.

For example, imagine writing the template:

{% highlight html %}
{% raw %}
  <a {{action 'someEvent'}}>Raise Event</a>
{% endraw %}
{% endhighlight %}

But you don&#8217;t know what `someEvent` should do yet. Clicking on the element will result in an error in the Console:

    [Error] Error: Nothing handled the action 'someEvent'. If you did handle the action, this error can be caused by returning true from an action handler in a controller, causing the action to bubble.


By simply defining the action (on the Route, for arguments sake) with `Ember.K`, you can satisfy Ember.js and avoid the error:

    actions: {
      someEvent: Ember.K
    }


You can continue writing your template and deal with the event logic later.

## Summary

But why is the function called K? I don&#8217;t know for sure, but it might be a nod to the [K Combinator][2] which is a function in Combinatorial Logic that simply returns its first argument. Here, we are returning `this` which could be said the be the secret first argument of every Javascript method call.

In short, you can use `Ember.K` for:

  * Making methods safe to call in the event that they might be undefined at the time of calling.
  * Providing hooks for subclasses to implement in a safe way.
  * Creating stand in action event handlers until you are ready to implement the necessary logic, and avoid Ember.js throwing an error.

I hope that&#8217;s been helpful! If you have any corrections, or questions don&#8217;t hesitate to email me.

 [1]: https://github.com/ggreer/the_silver_searcher
 [2]: http://wiki.tcl.tk/1923
