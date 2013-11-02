---
layout: post
published: true
title: The Great Satan - Rails Concerns
description: Why ActiveSupport::Concern is a really bad thing.
---

Corey Haines has already [masterfully expounded](http://blog.coreyhaines.com/2012/12/why-i-dont-use-activesupportconcern.html) on why ```ActiveSupport::Concern``` will lead to you having a Real Bad Time - and you really should read it, but I wanted to write a quick cautionary tale to those thinking about using Concerns and highlight a couple of the more practical pains explicitly.

## The Headless Horseman

```ActiveSupport::Concern``` may not have been intended to be used this way, but I see it used this way in a lot of Rails projects I've worked on since the introduction of Concerns. I'm talking about discorporated methods:

{% highlight ruby %}
require 'active_support/concern'

module MonsterConcern
  extend ActiveSupport::Concern

  included do
    def can_fangs_be_seen_by_children?
      nearby_children.can_see?(:fangs)
    end

    def shaggy_monster
      make_coat_type(:shaggy)
    end

    def engage_rage
      ...
    end

    def chew_bones
      ...
    end
  end
end
{% endhighlight %}

We have four disembodied methods here, stuck in a file. We get no indication of _whom_ includes this file - if we're lucky the name might reflect that information, but it's still not exact. Where should we go to see where the ```nearby_children``` method is defined?

## The Goose Chase

Let's say we want to look at the ```make_coat_type``` method. Where is it? It could be on the ```Monster``` model, but it's not. If we're lucky, maybe we spy a file called ```monster_coat_type.rb```, but chances are it's bundled into something bigger. Eventually we find the appropriate method, but it's in another concern file that we didn't expect.

## Hunt and Peck

New developers might look through a couple of files before they hit the thing they are looking for, because Concerns give no indication of what they contain at the file name level. Better developers will ```grep``` immediately, but even then they might waste time optimistically looking in a couple of file that seem likely first.

## Conclusion

```ActiveSupport::Concern``` actually makes your code base worse. They make it:

- Harder to reason about the code you are currently reading. You must satisfy several mental dependencies before you can make any kind of informed change in the code you are looking at. This increased mental barrier also makes it more likely that a developer will make a mistake in the code they are changing inside a concern.

- Harder to navigate the codebase. If I want to look at the ```make_coat_type``` method, I have to already know that it:

  - Doesn't live on the main ```Monster``` model.
  - Is actually inside another file which contains a different ```Concern```

- Encourage a use of Modules that is just as bad as 'The Kitchen Drawer' approach to using the ```lib``` directory that we know is bad, but this time it's at a much more fine grained line-of-code level.

This is all symptomatic of the Rails world only now just coming to terms with some of the Domain Driven Design concepts of Services, and at the same time having the project leader stand behind a really badly thought out approach to handling complex large software projects.

Your Models don't deserve this fate - tearing methods away from them and placing them in the Phantom Zone that is ActiveSupport::Concerns. Give them and anyone who might read the code other than you a chance for a better life, and if you need further guidance, there's this [excellent CodeClimate post](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/) which shows SEVEN ways not to use Concerns.