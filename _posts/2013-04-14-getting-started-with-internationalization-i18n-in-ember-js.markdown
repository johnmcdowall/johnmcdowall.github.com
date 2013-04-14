---
layout: post
published: true
title: Getting started with Internationalization (i18n) in Ember.js
description: Writing your Ember.js app to include Internationalization support is easy.
---

Writing your Ember.js app to include Internationalization support is easy. Here's a quick guide on how to implement it. 

1. Grab the CLDR.js library [here](https://github.com/jamesarosen/CLDR.js). CLDR provides a bunch of handy pluralization logic that works across different locales. You want it. 
2. Next, grab the ember-i18n library [here](https://github.com/jamesarosen/ember-i18n). This is the meat and potatoes of handling i18n in Ember applications, and it takes care of all of the gritty details. 
3. Now, before you create your Ember application, make sure to set the default locale for CLDR like so:
  ```CLDR.defaultLocale = 'en';```
4. Lastly, create a file called ```i18n.js``` that you will load after ```CLDR.js``` and ```ember-i18n```. This is where you will store your i18n strings. 
5. Inside the ```i18n.js``` file, simply setup an object like so, with the appropriate keys for your application:
    {% highlight javascript %}
    Em.I18n.translations = {
      'user.edit.title': 'Edit User',
      'user.followers.title.one': 'One Follower',
      'user.followers.title.other': 'All {{count}} Followers',
      'button.add_user.title': 'Add a user',
      'button.add_user.text': 'Add',
      'button.add_user.disabled': 'Saving...'
    };
    {% endhighlight %}

In your Handlebars templates, you can now reference the usual ```t``` object to get the i18n strings:

```<h2>{ { t user.edit.title } }</h2>```


Now, that was pretty easy, eh? 