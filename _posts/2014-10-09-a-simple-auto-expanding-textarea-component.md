---
title: A Simple Auto Expanding Textarea Component
author: John McDowall
layout: post
permalink: /a-simple-auto-expanding-textarea-component/
dsq_thread_id:
  - 3100150910
categories:
  - Uncategorized
---
Greetings! The nights are drawing in pretty quickly here, and it&#8217;s also additionally pretty dark in the morning due to the mountain I live next to and the added darkness is making it pretty difficult to maintain my early morning routines. A short and sweet post this week about how to extend the built-in Ember.js Textarea View so that it auto expands to contain the text inside it, but first a short interlude into a recent poll I ran on Ember.js usage:

<!--more-->

> The results can be seen here: <http://strawpoll.me/2695391/r>. Admittedly, it&#8217;s an extremely small sample size, but I was shocked to see Rails in the minority. It&#8217;s good to see Ember-CLI leading the pack, as it&#8217;s a great project – and the results overall make me feel more comfortable about writing more Ember-CLI related posts in the near future. On to today&#8217;s post&#8230;

When we need a `Textarea` in one of our Templates, we can simply use the `textarea` helper like so:

    {{textarea value=notes}}


And that will render a basic text area bound to the `notes` property. But it will be a static `textarea` that does not accommodate content that is larger than its style will permit without scroll bars. What if we wanted the `textarea` to automatically grow vertically as more text is entered, without resorting to any external jQuery plugins?

First of all, we create a new `Component` that `extends` the Ember `TextArea`:

{%highlight javascript linenos%}
App.AutoExpandingTextAreaComponent = Ember.TextArea.extend({
  ... snip ...
});
{%endhighlight%}

If we just left the new `AutoExpandingTextAreaComponent` as is above, we could use it in any template with `{{auto-expanding-text-area value=notes}}` and its behaviour would be exactly the same as the default `textarea`. So far, so obvious.

The trick here for us, is that we want the `textarea` to be aware of any keypresses, and change its size accordingly. We&#8217;ll use the `didInsertElement` hook to attach an event listener for `keypress`es on the text area:

{%highlight javascript linenos%}
App.AutoExpandingTextAreaComponent = Ember.TextArea.extend({
  didInsertElement: function(){

    Ember.run.next(function() {

      // Focus the text area
      this.$().focus();

      // Listen for keypress events and recalculate the height of the text area.
      this.$().on('keypress', function(e) {
        var textArea = $(this);
        var newHeight = this.scrollHeight + parseFloat(textArea.css("borderTopWidth")) + parseFloat(textArea.css("borderBottomWidth"));

        while(textArea.outerHeight() < newHeight) {
          $(this).height($(this).height()+1);
        }
     });

    }.bind(this));

  } // didInsertElement
});
{% endhighlight %}

We&#8217;re careful to make sure that the jQuery event listener runs inside an `Ember.run.next` in order to make sure it happens after Ember has propagated all property updates and settled it&#8217;s internal book-keeping on the Runloop.

Lastly, we want to make sure we tear down the event listener when the element is destroyed, by using the `willDestroyElement` hook:

{%highlight javascript linenos%}
App.AutoExpandingTextAreaComponent = Ember.TextArea.extend({
  didInsertElement: function(){
    Ember.run.next(function() {
      this.$().focus();
      this.$().on('keypress', function(e) {
        var textArea = $(this);
        var newHeight = this.scrollHeight + parseFloat(textArea.css("borderTopWidth")) + parseFloat(textArea.css("borderBottomWidth"));

        while(textArea.outerHeight() < newHeight) {
          $(this).height($(this).height()+1);
        }
     });
    }.bind(this));
  },

  willDestroyElement: function() {
    this.$().off('keypress');
  }
});
{% endhighlight %}

Here&#8217;s a working example:

<a class="jsbin-embed" href="http://emberjs.jsbin.com/saxoqokaniha/1/embed?js,output">Ember Starter Kit</a><script src="http://static.jsbin.com/js/embed.js"></script>

## Summary

Ember makes it pretty easy to extend the primitives it ships with, using the built in lifecycle hooks. In this example we took the stock `Ember.TextArea` view and extended it as a new component that has new behaviour on its appearance.
