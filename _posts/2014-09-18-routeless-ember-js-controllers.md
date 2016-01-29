---
title: Routeless Ember.js Controllers
author: John McDowall
layout: post
permalink: /routeless-ember-js-controllers/
cover_image: /images/posts/the-fog-john-houseman-campfire.jpg
dsq_thread_id:
  - 3030391449
categories:
  - Controllers
---
The leaves are falling from the trees around the Island. Fall, or as I still like to call it, Autumn, is here. The temperature is getting a little cooler now, and the nights are drawing in, so pull up close for this tale of the Ghostly Routeless Controller.

<!--more-->

The beginning view of Controllers in Ember.js is that they are instantiated and setup by the corresponding Route. And while this is somewhat true, it&#8217;s not the only way to use Controllers.

Let&#8217;s look at how we can use a `Controller` without a `Route`, and some of the techniques using the `Container` described in the [previous article][1] to model a [Davey Jones&#8217; Locker][2] and display a list of Lost Souls in our App.

We&#8217;re going to need four things to implement this feature:

  * A `Controller` to represent our collection of Lost Souls.
  * An `Initializer` to inject the Controller into the other primitives we wish to have access to the Lost Soul in.
  * A `Template` to render the pathetic contents of the Locker.

## The Controller

Let&#8217;s look at the `Controller` first.

{%highlight javascript linenos%}
App.DaveyJonesLockerController = Ember.ArrayController.extend({
  actions: {
    addSoul: function() {
      var person = {
        name: 'A sad individual'
      };
      // Insert the 'model' at the head of the Array
      this.insertAt(0, person);
    },
    emptyLocker: function() {
      this.clear();
    }
  }
});
{% endhighlight %}

We&#8217;re declaring an `ArrayController` because our `DaveyJonesLockerController` is going to hold a collection of Lost Souls. The core action is `addSoul`, which has the following tasks:

  * Construct a synthetic Model which stores the Lost Souls. I&#8217;m calling it synthetic because it isn&#8217;t backed by Ember Data or any other persistence library.
  * Insert the new model at the head of the ArrayController.

Finally, we declare an action called `emptyLocker` for handling when Davey is tired of these pathetic lost souls whining all the time.

## The Initializer

But at this point, Ember won&#8217;t know about our `DaveyJonesLockerController`, and no `Route` we have defined will set it up. For that, we need an `Initializer`:

{%highlight javascript linenos%}
Ember.Application.initializer({
  name: 'daveyJonesLocker',
  initialize: function(container, application){
    application.inject('route', 'locker', 'controller:davey-jones-locker');
  }
});
{% endhighlight %}

Again, it&#8217;s very, almost childishly, simple. We use the `Initializer` and the convenience method `inject` available on the `application` variable to inject into any routes in the `Container` an instance of our `DaveyJonesLockerController`, and make it available from the `locker` property. As an aside, we could also have injected into any `Controller` like so:

`application.inject('controller', 'locker', 'controller:davey-jones-locker');`

And Controllers would also subsequently have an `locker` property they could access.

## The Template

Next, we build our `Template` that will be used with the `DaveyJonesLockerController`:

{%highlight html linenos%}
{% raw %}
<button {{action 'addSoul'}}>Add soul</button>
{{#if length}}
  <ul class='soul-list'>
    {{#each person in model}}
      <li>{{person.name}}</li>
    {{/each}}
  </ul>
  <button class='flush-locker' {{action 'emptyLocker'}}>Empty Locker</button>
{{/if}}
{% endraw %}
{%endhighlight%}

In this `Template`, we&#8217;re saying if `length` isn&#8217;t a truthy value (in this case greater than zero, i.e. we have Lost Souls), then don&#8217;t display anything. Well, where does `length` come from? In this case `length` is found as a property on `ArrayController` and returns the underlying length of the `Array` the Controller is wrapping. The `Template` is said to be &#8216;proxying&#8217; the property lookup onto the ArrayController.

Next, we simply iterate on the `model` that the `ArrayController` represents and display the appropriate HTML for each Lost Soul.

Lastly, in our main Application Template, we need to use the `render` [helper][3]:

    {{render 'davey-jones-locker'}}


This tells Ember to fire up the `DaveyJonesLockerController` and when rendering the Application `Template`, render the output of the `DaveyJonesLockerController` here.

Here&#8217;s the complete example:

<a class="jsbin-embed" href="http://emberjs.jsbin.com/fubocuniquno/4/embed?output">Ember Starter Kit</a><script src="http://static.jsbin.com/js/embed.js"></script>

## Adding souls from elsewhere

Here&#8217;s how you might add souls to the locker from, say, an action in the Index template. We know from our Initializer that we injected a `locker` property that gives us access to our `DaveyJonesLockerController` onto all `Routes`. First, in the `IndexRoute`, we&#8217;ll capture the `addSoul` action from a button in the template:

{%highlight javascript linenos%}
    App.IndexRoute = Ember.Route.extend({
      actions: {
        addSoul: function() {
          this.get('locker').send('addSoul');
        },
      }
    });
{% endhighlight %}

Next&#8230;well, that&#8217;s it! You&#8217;ve just added a soul from another template into the `DaveyJonesLockerController`.

## Summary

We&#8217;ve created a `Controller` that isn&#8217;t attached to any `Route`, and we&#8217;re using it with a simple pseudo-model that isn&#8217;t used anywhere else in the Application other than to provide global functionality that our Application needs.

In the example above, we have injected the `DaveyJonesLocker` Controller into all Routes &#8211; so from any Route we could add new souls to the locker by accessing the `locker` property and calling `addSoul` on it like so:

`this.get('locker').sendAction('addSoul');`

Since this is just an example, we just raise the action `addSoul` from the Template displayed for `DaveyJonesLockerController` by the call to the `render` helper, but it could also be a method call.

Any time you have data, that still needs behaviour around it, but that doesn&#8217;t fit into the grander Routing structure of your Application, this technique is your friend. You could consider it as &#8220;Global&#8221; data, or as I like to think of it, more along the lines of a super lightweight service.

Happy coding!

 [1]: http://ember.zone/beginning-to-understand-the-ember-js-container/
 [2]: http://en.wikipedia.org/wiki/Davy_Jones'_Locker
 [3]: http://emberjs.com/guides/templates/rendering-with-helpers/#toc_comparison-table
