---
layout: post
published: true
title: A simple Rake task to update Ember.js
description: Writing your Ember.js app to include Internationalization support is easy.
---

I was using a Rake task that I had seen [Ivan Vanderbyl](https://twitter.com/IvanVanderbyl) using to update the Ember source files to whatever was the latest available. It assumed you were using the ```ember-rails``` gem and it would pull down the repos necessary, build the projects, and place the compilation results in the ```vendor``` directory so that the asset pipeline would pick them up. 

Since the Ember team put some work into getting builds served up from S3, the process of getting the latest code has become super simple. All you need is this Rake task in your ```lib/tasks``` folder and away you go:

{% highlight ruby %}
# A simple rake task to update Ember and Ember Data to latest official build from S3.
#
# Assumptions:
# - You're using the `ember-rails` gem
# - You are requiring rember with sprockets with something like `require ember`
#
# Usage:
#
#   $ rake ember:update
#
#  This will place a development build (with comments and debug tags) into:
#   vendor/assets/ember/development/ember.js
#  and a production minified build (no comments or debug tags) into:
#   vendor/assets/ember/production/ember.js
#
# Idea by Ivan Vanderbyl (ivan@crashlog.io) December, 2012
# This implementation by John McDowall (john@mcdowall.info) April, 2013
 
def say(msg, &block)
  print "#{msg}..."
 
  if block_given?
    quietly do
      yield
    end
    puts " Done."
  end
end
 
namespace :ember do
  desc "Update Ember.js from latest Builds on S3."
  task :update => [:core, :data]

  task :core do
    say "Grabbing Core from S3..." do 
      sh "curl -# http://builds.emberjs.com.s3.amazonaws.com/ember-latest.js -o vendor/assets/ember/development/ember.js"
      sh "curl -# http://builds.emberjs.com.s3.amazonaws.com/ember-latest.min.js -o vendor/assets/ember/production/ember.js"
    end
  end

  task :data do
    say "Grabbing Data from S3..." do 
      sh "curl -# http://builds.emberjs.com.s3.amazonaws.com/ember-data-latest.js -o vendor/assets/ember/development/ember-data.js"
      sh "curl -# http://builds.emberjs.com.s3.amazonaws.com/ember-data-latest.min.js -o vendor/assets/ember/production/ember-data.js"
    end
  end
end
{% endhighlight %}

Now you can simply say ```rake ember:update``` and if you're using the ```ember-rails``` gem, you'll get the latest builds in no time at all. 

Gist [here](https://gist.github.com/johnmcdowall/5431985).