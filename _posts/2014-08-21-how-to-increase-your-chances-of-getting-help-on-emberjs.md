---
title: 'How to increase your chances of getting help on #emberjs'
author: John McDowall
layout: post
permalink: /how-to-increase-your-chances-of-getting-help-on-emberjs/
dsq_thread_id:
  - 2945605494
categories:
  - Ember
---
I live on an [Island][1]. If I need help with something, it&#8217;s not always easy to get it right away. Sometimes I have to figure something out for myself, like a pleb. But enough of my Mountain Man ramblings.

Here&#8217;s a great way to increase your ability to think for yourself for those times when you can&#8217;t quickly or easily get help, and when someone can help you, you&#8217;ve given them the best possible start for assisting. The Ember.js IRC channels are a great place to ask questions and get help when you&#8217;re stuck on something and the documentation isn&#8217;t helping or Stack Overflow is giving you snake eyes. But there&#8217;s an okay way and an excellent way to get help on #emberjs. Let&#8217;s see what the options are and how one is better than the other.

<!--more-->

## Github Gists

Most folks take this option, from what I can see hanging out in #emberjs. They take sections out of their app and plonk them into a Gist, and post that into the channel while asking for help. This is the obvious and quickest route to getting someone else to understand your predicament, but it lacks several significant things that will make it likely that you&#8217;ll get help quickly.

### Static, Lifeless, Dead.

Gists are static – they don&#8217;t show how the parts in question are actually interacting, which means the reader has to do a lot of mental reconstruction to figure out what&#8217;s happening. If the problem you&#8217;re trying to tackle can actually be expressed in one complete file, then it&#8217;s not so bad (for example, a Component Qunit test).

### Hide Framework bugs

Because Gists are static, they don&#8217;t exercise the full Ember.js stack, so your problem could be a bug in the framework, but it&#8217;s being hidden because the person trying to help you has no way to see the effects.

### They don&#8217;t really force you to think about the problem

It&#8217;s super easy to copy and paste fragments of your app into a Gist. But doing so doesn&#8217;t make you think about your problem, and could be helping to hide some really obvious bug you&#8217;ve missed.

So, what&#8217;s the better alternative? Enter, stage left&#8230;

## JSBin

[JSBin][2] is a great way to share functional code snippets with the people you want to help you. What&#8217;s better, you can get an instantly constructed base Ember project simply by hitting the URL <http://emberjs.jsbin.com>. Try it!

### Deconstruct your problem

First step, is to try and extract the problem into a general high level example of what you&#8217;re trying to achieve that works in JSBin. In doing so, it forces you to think about the problem and chances are you&#8217;ll find your own solution. It&#8217;s like [digital rubber ducking][3].

### I need Ember Data ♨_♨

Well, you can add Ember Data support with a couple of lines.

  1. Add the script tag to the lastest build of Ember Data into the HTML. You can grab this easily from the <http://emberjs.com/builds/#/beta> page.
  2. Setup the fixture adapter: `App.ApplicationAdapter = DS.FixtureAdapter.extend();`
  3. Define a model:

        App.ListItem = DS.Model.extend({
          title: DS.attr('string')
        });


  4. Define some Fixture data:

        App.ListItem.FIXTURES = [
          {
            id: 1,
            "title": "cat"
          },
          {
             id: 2,
             "title": "dog"
          }
        ];


  5. Done!</p>

You&#8217;ve now got a simple data model setup.

By using JSBin, and almost by magic, you&#8217;re increasing the chances that you&#8217;ll find the solution to your problem by yourself, and if you don&#8217;t, that whoever wants to help you will be quickly able to assess what&#8217;s going on – meaning that they are more likely to help in the first place.

## Summary

Gists are ok, but if you really want to boost your chances of getting help, and even maybe solve the problem yourself without even needing to go into the channel, use JSBin. I&#8217;ll frequently fire up a JSBin to try out an idea or explore a concept in Ember. For example, let&#8217;s say you wanted to explore how to side-load object from a page at runtime. Easy, just fire up JSBin and start sketching out a solution &#8211; like this one:

<a class="jsbin-embed" href="http://emberjs.jsbin.com/xulenifapagi/1/embed?html,js,output">Ember Starter Kit</a><script src="http://static.jsbin.com/js/embed.js"></script>

Try to incorporate JSBin into your daily Ember.js work and you&#8217;ll find it&#8217;ll make you a better Ember developer in no time at all.

 [1]: https://www.google.ca/maps/place/Bowen+Island,+BC/@49.3767427,-123.3704728,13z/data=!3m1!4b1!4m2!3m1!1s0x54866b0a15721b2f:0x1150cf2b21435466
 [2]: http://jsbin.com
 [3]: http://en.wikipedia.org/wiki/Rubber_duck_debugging