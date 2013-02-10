---
layout: post
published: true
title: Gemfile Best Practices &amp; Discourse
description: Your Gemfile is often the starting place for new developers to get up to speed on your code. Here are some simple practices that can help new developers get a sense of the makeup and any peculiarities, and keep order in your project's Gemfile.
---

This weekend I [did some Boy Scouting](https://github.com/discourse/discourse/pull/93) on the newly released [Discourse project](https://github.com/discourse/discourse). The [Boy Scout Rule](http://www.hans-eric.com/2010/07/26/the-boy-scout-rule/) says:

> You should always leave the campground cleaner than you found it.

Discourse is a Rails project, which means it has a nice fat, juicy Gemfile listing all of its dependencies. 

The Gemfile tends to be the first thing I look at in an unfamiliar Rails project in order to get a quick mental overview of what makes up the application and if it's using any custom forks or private libraries. 

Over the years, I've come to form a set of best practices for the general organization of Gemfiles that I think are very useful when on-boarding new developers and keeping order in your Gemfile. 

1. Consistent use of Ruby hash syntax. Use either the old hashrocket or the new Ruby 1.9 syntax, but not both. 
2. Consistent use of a single quoted delimiter. Use either apostrophes or quotation marks, but not both.
3. No commented Gem references. If it's commented out, it shouldn't be there. 
4. Comments relating to a Gem are on the same line as the gem statement, not above. 
5. Group Gems that are sourced from Git repos at the top. Chances are they are referencing pre-versions that will become general release and you can change the reference to be part of the General project group later.
6. Group Gems that are sourced from a project path after Git repo sourced Gems. These are probably gems that you might make public and thus reference in the general project gem group later.
7. Group all of the General project gems together (consider using the `:default` group).
8. Group all of the Asset gems after the General group.
9. Group all of the Test related gems after the Asset gems. 
10. Group all of the Development related gems after the Test gems. 
11. Within all Gem groups, sort the references by Alphabetical order.
12. When adding new Gems, maintain the alphabetical ordering within the groups.

These guidelines produce a Gemfile which is logical and ordered: no little clumps of Gems related to the main project here and there. The alphabetical ordering within the groups increases scanability to see if a Gem has already been included. 

Here's my Discourse Gemfile changes:

<script src="https://gist.github.com/johnmcdowall/4751169.js"></script>

What do you think? Discuss these ideas on [Hacker News](http://news.ycombinator.com/item?id=5198217).
