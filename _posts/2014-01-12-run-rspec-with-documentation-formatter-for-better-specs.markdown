---
layout: post
published: true
title: Run RSpec with Documentation Formatter for Better Specs
description: The progress formatter hides some really nasty business.
---

The default RSpec report formatter is `progress`: it's the one that, hopefully, outputs litte green dots as RSpec executes each Spec. This can lead to serious problems with the structure of your Specs.

## Everyone wants to be happy

`progress` is the formatter that makes you feel better than you probably should. It's the formatter that says 'Hey, everything's OK. So much green.'

```
» rspec spec/requests/tps_2014_spec.rb
...................
```

But the `progress` formatter hides times when what you've said you're Speccing is complete nonsense. Often, if you haven't thought about the Spec assertion in any depth, your test might well be nonsense too.

## Documentation formatter to the rescue

RSpec allows you to use different style report formatters (which you can list by typing `rspec -h`), and one of the other built in options is `documentation`. You can choose it easily by inserting `-fd` into your rspec options:

Let's look at an example:

```rspec
» rspec -fd spec/requests/tps_2014_spec.rb

CreateAccount
  paid account
    success
      sets annual plan
      sets the correct values
      moves the needle
      sets the pricing_version
  free account
    success
      sends out a welcome email
      associates the models correctly
      creates a user and free org
        should have content "Welcome to TPS Report 2014"
        should have content "Welcome to TPS Report 2014"
        should have content "Welcome to TPS Report 2014"
    started free then switched to paid
      failure
        doesn't let me check out
      success
        creates paid org
    failure
      incomplete form
        browser behavior
          puts me back on the form
        does not create squat
          should not change #count
          should not change #count
          should not change #count
      email already taken
        should report the error
        doesn't work
          should not change #count
          should not change #count
          should not change #count
      org already taken
        should report the error
        doesn't work
          should not change #count
          should not change #count
          should not change #count
```

The problem is immediately apparent. These Spec descriptions don't actually make any sense. We're seeing multiple repetitions of the same message, and that means either the Spec description is wrong, the test is a lie, or at the very least could be written to be more informative.

## Examining the issue

Let's look at the Spec `CreateAccount > free account > failure > email already taken > doesn't work > should not change #count`. The documentation formatter is telling us three times that something 'should not change #count'. Here are the problems:

- What #count shouldn't change? Cats? Doges?
- Are we asserting three times that the same #count shouldn't change?
- Why are we asserting three times?

Let's look at the Spec:

```
describe "doesn't work" do
  subject { -> { click_button 'Create TPS Report' } }

  it { should_not change(User, :count) }
  it { should_not change(Comapny, :count) }
  it { should_not change(TPSReport, :count) }
end
```

We can see the condition "doesn't work" actually has three distinct criteria that must be satisfied, which is completely absent from the documentation output. This is easy enough to fix:

```
describe "doesn't work" do
  subject { -> { click_button 'Create TPS Report' } }

  it { should_not change(User, :count), 'A User is not created.' }
  it { should_not change(Comapny, :count), 'A User is not created.' }
  it { should_not change(TPSReport, :count), 'A User is not created.' }
end
```

You could argue that I've increased the number of lines required to express the same idea, and for not much gain while reading the Spec. I'd argue I've made the Spec more explicitly intentional about