---
title: Handling environment data with Ember.js
author: John McDowall
layout: post
permalink: /handling-environment-data-with-ember-js/
dsq_thread_id:
  - 2906984383
categories:
  - Ember Infrastructure
---
It&#8217;s likely that your Ember app is going to require some configuration or environment data as it grows: deployment environment name, feature flags, locale strings and so on.

What&#8217;s the best way to handle this requirement? There are a couple of ways you could handle this, but here&#8217;s a technique I&#8217;ve seen in the wild and that works for me.

<!--more-->

## Ember Application Initializers to the rescue

Ember provides an Initializer mechanism that allows you to inject things into the container, or access the instantiated Application object before the application fully runs. This provides a quick and easy easy way for us to use the Application object as a bucket for environment configuration. A simple Initializer takes the form:

{%highlight javascript linenos%}
Ember.Application.initializer({
  name: 'initializer-name-goes-here',

  initialize: function( container, application ) {
  }
});
{% endhighlight %}

As you can see from the snippet above, we are provided with access to the container, and the instantiated Application instance. Perfect.

## Storing the Environment Data

The quickest and easiest way to store the Environment data is to write `meta` tags to your HTML document that our Ember application will look for and read from the Initializer. This is cleaner than using script tags and polluting the global Javascript namespace.

We can use a convention of naming the `meta` tag with something like `env-` at the beginning to signify that it represents environment data:

{%highlight javascript linenos%}
<meta name="env-name" value="production">
<meta name="env-locale" value="en">
{%endhighlight%}


## The configuration Initializer

It might look a little complex, but I&#8217;ve broken each step down with comments:

{%highlight javascript linenos%}
Ember.Application.initializer({
  name: 'config',

  initialize: function( container, application ) {

    // 0. Check to see that we have a bucket for the env data
    if(application.get('env') === undefined) {
      throw new Ember.Error("The Application object must contain an 'env' variable which is assigned as Em.Object.create().");
    }

    var EnvReader = function EnvReader() {

      this.readEnvKeys = function() {

        // 1. Grab all the meta tags from the DOM.
        var metaTags = $("meta");
        var envVars  = {};

        // 2. Process each of the discovered meta tags.
        for( var i=0; i < metaTags.length; i++ ) {

          // 3. Get the meta tag name and value.
          var envKey   = $(metaTags[i]).attr("name");
          var envValue = $(metaTags[i]).attr("value");

          // 4. Does the meta tag start with 'env-'?
          if (/^env\-/.test(envKey)) {
            // 5. Produce a camelized version of the env variable name, ignoring the initial 'env-'.
            var propertyName = Em.String.camelize(envKey.substring(4));

            // 6. Map the string values to actual types.
            envVars[propertyName] = this._mapType(envValue);
          }
        }

        return envVars;
      };

      this._mapType = function(val) {
        return "" === val ? null : "true" === val ? true : "false" === val ? false : (-1 !== val.indexOf(",") && (val = val.split(",")), val);
      };

    };

    var envReader = new EnvReader();
    application.get("env").setProperties(envReader.readEnvKeys());
  }
});
{% endhighlight %}

At a high level, we&#8217;re just pulling out `meta` tags from the DOM, testing if their names start with our chosen convention for environment configuration, generating a Camelized name and mapping the string values to actual Javascript types. The private `_mapType` function even handles converting comma separated values into `Arrays` &#8211; useful for feature flags.

## Performance

This technique relies on Ember&#8217;s Application Initializer facility. Initializers, as the name might suggest, are used only during application startup to allow developers to do things which set the application up &#8211; things like we have just done above. We&#8217;ve used an Initializer to query the DOM *once*, set an object on the global Application object to hold all our configuration and environment data, and then direct subsequent queries to those cached values. The only other time the DOM will be queried is if you reset your Application object, and you&#8217;ll be rebuilding the entire world at that point anyway.

## Summary

By using Ember Application Initializers, we can write a re-usable mechanism for reading environment configuration data from the DOM, and place it in a central bucket that we create on the Ember Application object.

You can access the environment data very easily by referencing the path of the key, like so:

    App.get('env.locale');


See it in action in this JSBIN:

<a class="jsbin-embed" href="http://emberjs.jsbin.com/dowepipo/2/embed?html,js,output">Ember Starter Kit</a><script src="http://static.jsbin.com/js/embed.js"></script>

### This post and I need your help!

Please share this post if you found it useful. It&#8217;ll take a few seconds, but the encouragement will keep me motivated to write new posts! Thanks!
