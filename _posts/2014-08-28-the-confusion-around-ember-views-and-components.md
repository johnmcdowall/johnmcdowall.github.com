---
title: The Confusion Around Ember Views and Components
author: John McDowall
layout: post
permalink: /the-confusion-around-ember-views-and-components/
dsq_thread_id:
  - 2967187163
categories:
  - Ember View Layer
---
The island that I live on has a year-round population of around 3,000 folks. It&#8217;s a pretty tight knit community. We have our own special online forum, looking like it&#8217;s been running since the Internet began, where Islanders will post useful information, community updates, garage sales announcements and the like. However, it&#8217;s very much a case of being in the right place at the right time. If you don&#8217;t log on to the island forum for a couple of days, you could easily miss that the local Cafe is holding a free all-day pie tasting. And that&#8217;s a total bummer.

The Ember community is still a bit like this. You have to be in the right place at the right time to receive some new wisdom about how things work. This is especially true for the shift that happened with Views and Components.

<!--more-->

## Views vs Components

When I first started getting involved with Ember on a regular basis, Views were the only option. Components weren&#8217;t even a twinkle in Tomhuda Katzdale&#8217;s eye. So, you wrote lots of Views. The thing about Views is that they have implicit access to the current context, so you can easily reference properties on your Controller. When Components came along, I thought &#8216;great!&#8217; and agreed it was a fabulous way to write re-usable, self contained components in your app, but noted that Components are completely isolated from the current context – you have to pass everything in that you might need. So far, so good.

Somewhere along the way, I noticed that the community had gone [HAM][1] on using Components all the time, in almost every circumstance. To me this seemed crazy! Views have their place, Components have their place. I was curious as to how this shift had come about.

## Digging up the truth

Googling, I found [this StackOverflow][2] question on the difference between Components and Views, and when to use each. One answer [in particular][3] was advocating basically forgetting about Views and that Tomhuda Katzdale had officially advocated this position:

> According to a training video that was recorded on August 2013, Yehuda Kats and Tom Dale (Ember Core Team Members) told the audience to not use views unless you&#8217;re a framework developer. They&#8217;ve made lots of enhancements to Handlebars and introduced Components, so views are no longer necessary. Views are used internally to power things like {{#if}} and {{outlet}}.

And yet the official Guides still had references like:

> Views in Ember.js are typically only created for the following reasons:
>
>   * When you need sophisticated handling of user events
>   * When you want to create a re-usable component

So I decided to do some investigation. The StackOverflow user mentioned &#8216;video training&#8217; – the only video training I was aware of was the official Tilde introductory course for sale at [Gaslight][4]. As it would happen, I had bought this previously and hadn&#8217;t had time to get through it fully. I did some digging.

Eventually I found the section in the Gaslight video training: 10.2 Components Q&A @ 26:15 mark

<img src="{{ "/images/posts/Screen-Shot-2014-08-27-at-9.04.52-PM-1024x912.png" | prepend:site.baseurl }}" alt="Tom Dale" width="300" height="267" class="alignnone size-medium wp-image-78" />

> Tom Dale: There was a general question about Handlebars, and probably some of you have gone through older Ember app examples and probably seen a lot of View classes. Basically don&#8217;t use Views is my answer. Since those examples were written we&#8217;ve added a lot more features to Handlerbars [..] we&#8217;ve added Components, [..] in general I would say Views are not something that we would expect most developers to be writing&#8230; they&#8217;re more of an internal book-keeping object at this point. You should use Components.

Then at 30m:

<img src="{{ "/images/posts/Screen-Shot-2014-08-27-at-9.10.54-PM.png" | prepend:site.baseurl }}" alt="Yehuda Katz" width="300" height="179" class="alignnone size-medium wp-image-79" />

> Yehuda Katz: View is basically an internal implementation detail that is the root class of things like #if, outlets, components&#8230;you could use a View but then you&#8217;re a Framework developer. You should instead use one of the things that we have built for you that uses View and the thing that is closest to View but still has better semantics is Component.

So you see, you had to be in the right place, at the right time. I can&#8217;t find this positioning reflected anywhere in the Guides (please let me know if you can!).

Incidentally, here&#8217;s a handy table that Yehuda drew in the video:

<table class='table'>
  <tr>
    <td>
      &nbsp;
    </td>

    <td>
      controller
    </td>

    <td>
      scope
    </td>
  </tr>

  <tr>
    <td>
      render
    </td>

    <td>
      own
    </td>

    <td>
      isolated
    </td>
  </tr>

  <tr>
    <td>
      partial
    </td>

    <td>
      parent
    </td>

    <td>
      parent
    </td>
  </tr>

  <tr>
    <td>
      component
    </td>

    <td>
      none
    </td>

    <td>
      isolated
    </td>
  </tr>
</table>

## So?

Yehuda and Tom seemed to have given out a very vital piece of information in the training video, that&#8217;s gotten lost. This happens from time to time: I just discovered that the way I&#8217;ve been defining Fixtures in Ember Data is a lie thanks to a Ember Zone reader&#8217;s comment on the previous post, where the real definition is covered in a Github Issue.

I also think a lot of developers are favouring Components because it&#8217;s Ember&#8217;s reflection of the new hot thing &#8211; Web Components – but without fully understanding the tradeoffs involved.

Personally, I think unless the answer to &#8216;Am I going to re-use this elsewhere in my App&#8217; is yes, I&#8217;ll still use a View purely to avoid the unnecessary ceremony involved in setting up everything a Component might need. I also think it&#8217;s still semantically clearer.

## Summary

Here&#8217;s what I say:

  * Views – use &#8216;em when the thing isn&#8217;t going to be re-used elsewhere, and avoid un-necessary setup ceremony.
  * Components &#8211; use &#8216;em when the thing is truly re-useable.

Perhaps one side benefit of &#8216;everything is a Component&#8217; is easier testing as the Component testing story is very good in Ember now, but I haven&#8217;t delved deep enough into this yet to be able to say for sure. That might be one reason to always use Components.

 [1]: http://www.urbandictionary.com/define.php?term=ham&defid=2992211
 [2]: http://stackoverflow.com/questions/18593424/views-vs-components-in-ember-js
 [3]: http://stackoverflow.com/a/23798881
 [4]: https://teamgaslight.com/training/courses/4
