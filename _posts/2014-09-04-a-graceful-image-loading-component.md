---
title: A Graceful Image Loading Component
author: John McDowall
layout: post
permalink: /a-graceful-image-loading-component/
dsq_thread_id:
  - 2986562324
categories:
  - Components
  - Uncategorized
---
Fall is definitely on its way out here in the Pacific North West. And that means lots of wood has to be chopped for Winter, so you&#8217;ll understand and bear with me if this week&#8217;s article is a little short gem that I&#8217;ve used on several occasions now.

There are times when you might be loading a large image, that for whatever reason hasn&#8217;t been resized for displaying properly – topographical maps, sonar scans, aerial imagery or what have you, and you want a better experience for your users than having to watch the image slowly load. I&#8217;m going to show you how with a little CSS and DOM magic, you can have a slow loading image fade in when it is fully loaded.

<!--more-->

There&#8217;s an absolutely fantastic article by The Barrel gang called [Taking Control of Image Loading][1] – I highly recommend you read it as it&#8217;s first technique is the basis for what I am about to show you. All credit for the CSS technique goes to them.

Essentially, we wait for the browser to fire off a &#8216;load&#8217; DOM event on the image when it is fully loaded. When this event occurs, we use it to attach a `loaded` CSS class to the Ember Component containing `div`. Then with some simple CSS, we can toggle from the image in question having an opacity of 0 to an opacity of 1, aided with some nice CSS 3 transitions to perform the fading. Perhaps a demo, Sir/Madame? Voila!

<a class="jsbin-embed" href="http://emberjs.jsbin.com/paponuminuso/1/embed?output">Ember Starter Kit</a><script src="http://static.jsbin.com/js/embed.js"></script>

Now, hopefully above you saw a nice little spinning pie, before a lovely image of the Ember Zone mobile Headquarters appeared. If you hover on the JSBin above, and click the &#8216;Run with JS&#8217; button a few times, you should be able to repeat the effect.

So, what would such an Ember Component look like? Here you go:

{%highlight javascript linenos%}
App.ImageLoaderComponent = Ember.Component.extend({
  src: '',
  classNames: ['img_wrapper'],
  classNameBindings: ['loaded'],
  loaded: false,

  handleLoad: function() {
    var view = this;

    // Use jQuery's `one` to ensure the handler is remove afterwards
    this.$().children('img').one('load', function() {
      // Wrap in an Ember.run to stop Ember from guessing what to do:
      // http://emberjs.com/guides/understanding-ember/run-loop/#toc_what-happens-if-i-forget-to-start-a-run-loop-in-an-async-handler
      Ember.run(function(){
        view.set('loaded', true);
      });
    }.bind(this));
  }.on('didInsertElement')
});
{%endhighlight%}


We&#8217;ve been able to accomplish this effect in 12 lines of code. **This** is the Ember Zone. I was able to read the Barrel article, and directly translate it into an Ember Component without having to look anything up in the Ember guides – the concept translated directly into code that I thought should work, and it did.

The most interesting part of the above code snippet is where we use a function provided as a prototype extension of `Function.prototype` by Ember.js called `on` to signify that the `handleLoad` function should be run when the `didInsertElement` event occurs. This is a useful technique to stay clear of writing a giant monolithic `didInsertElement` function and instead have cohesive functions that trigger on the correct event.

One thing that could be a bit confusing is the unfortunate naming collision between jQuery&#8217;s `on` method &#8211; which is used to attach event handlers to event types, and Ember&#8217;s own `on` Function prototype extension which although it has a similar function is not the same, and can only listen for Ember events as far as I can see. Hopefully someone will correct me!

Then we use `bind` to `Function.prototype` that allows you to keep the context of `this` in the method used as callback allowing us to avoid having to do any `var _this = this'` hoopla.

## Summary

You now have a neat little Component that will show images in a nice way, enhancing the default experience provided by the browser to something a little bit better. And hopefully you learned something about Ember&#8217;s `on` extension, and the `bind` extension present in Javascript.

### Update

Thanks to Pat O&#8217;Callaghan for pointing out in the comments that I was still using the old jQuery `bind` method for attaching event handlers, so I&#8217;ve updated the article. Thanks, Pat!

### Update 2

More updates! Thanks to @mixonic for reaching out to point out there&#8217;s some reliability issues with the image `load` event in browsers – stay tuned for a complete overhaul of this component later!

 [1]: http://www.barrelny.com/blog/taking-control-of-imageloading/