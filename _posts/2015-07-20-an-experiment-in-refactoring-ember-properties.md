---
title: An experiment in refactoring Ember Properties
author: John McDowall
layout: post
permalink: /an-experiment-in-refactoring-ember-properties/
draft: false
dsq_thread_id:
categories:
  - ember-core
---

You might have run across the situation where you have lots of properties in a Controller or Component, and they have associated Computed Properties that calculate some presence value. They end up feeling like repetitive boilerplate. 

How might you go about cleaning them up? Perhaps auto-generating them?

## Typical Ember Example

Specifically, what kind of code am I talking about? Something like this:

{%highlight javascript linenos%}
import Ember from 'ember';

let LoginComponent = Ember.Component.extend({
  userName: '', 
  password: '', 

  hasUsername: Ember.computed.notEmpty('userName'),
  hasPassword: Ember.computed.notEmpty('password'),

  ... etc ...
});

export default LoginComponent;
{% endhighlight %}

Typically these properties are the results of requirements from the UI to display different elements on particular states. The template will have some conditionals that lean on these properties to achieve this. Depending on how complex the UI is, you could end up having a few of these state properties. 

Let's look at how we could save ourselves from manually having to define lots of boilerplate properties.

## Higher Order Functions 
Any function that takes another function as a parameter, or returns a function as its result is a Higher Order Function. Given that functions are first class citizens in Javascript, it's pretty easy, even casual to use Higher Order Functions to clean up functions that differ only by name and the property referenced. Can we use Higher Order Functions to create the `notEmpty` Computed Properties for us? Let's see. 

## Take 1: Using Ember's `defineProperty` Method

Ember provides a method which allows you to perform a basic level of meta-programming by defining properties at runtime. The parameters for `defineProperty` are (basically):

- The object you want to define the new property on
- The name of the new property
- A 'Descriptor' which for our purposes will be a Computed Property to be attached to the new property

Which looks like this:

`Ember.defineProperty(this, myCpPropName, Ember.computed('propName', function(){...});`

We're now going to use a Higher Order Function to pass back to `defineProperty` so that the Computed Property has a dynamic value.

_Note that this is a trivial example for the purposes of showing how Higher Order Functions can be used, and the example is chosen to illustrate the technique._

And with that in mind lets look at the solution:

{%highlight javascript linenos%}
import Ember from 'ember';

const { computed, on } = Ember;

// This is our Higher Order Function, that we are passing in the field name
// to so that it can be captured by the Computed Property and returned as the
// function to be called. 
//
// We define it here because if we defined it on the Component we would expose
// a method on the public API which is not usable outside the component, and 
// only returns a descriptor. 
const has = (field) => { return computed.notEmpty(field); };

let LoginComponent = Ember.Component.extend({

  UI_FIELDS: ['userName', 'password'],

  // Builds a field property name from a field name.
  _fieldPropName(field) {
    // Utilize ES6 string interpolation. 
    return `has${field.capitalize()}`;
  },

  // Run once on component init, sets the default empty values for the field
  // and generated the hasPropertyName properties for each field.
  defineFieldProperties: on('init', function() {
    this.UI_FIELDS.forEach((field) => {
      this.set(field, '');
      let propName = this._fieldPropName(field);
      Ember.defineProperty(this, propName, has(field));
    });
  })
});

export default LoginComponent;

{% endhighlight %}

Unfortunately there's a bit of a hidden trap here. As it turns out `defineProperty` is a relatively slow function, and we've just set it up so that the Component is going to execute it every time its initialized. While the technique above would be fine in something long lived like a `Service, it's no good in a component displayed 100 times. So what do we do? 

## Take 2: Pre-creating the attributes at Object creation

The best way to go about this is to pre-generate an Object with the properties and Computed Properties we need, and then re-extend our component with the pre-generated items before we `export` it. By taking this approach, Ember will only have to do its `defineProperty` magic once and only once.

{%highlight javascript linenos%}
import Ember from 'ember';

const { computed, on } = Ember;

// Our Higher Order Function, as before
const has = (field) => { return computed.notEmpty(field); }

// The Component UI fields as before. 
const UI_FIELDS = ['userName', 'password'];

// Pre-generate an object with the properties and computed
// properties that we need. 
let processedAttrs = {};
UI_FIELDS.forEach( (field) => {
  processedAttrs[field] = '';
  processedAttrs[`has${field.capitalize()}`] = has(field);
});

// Create a Component just as you normally would. 
let LoginComponent = Ember.Component.extend({
  // Your usual Ember component stuff in here as needed
  debugOutput: on('init', function(){
    console.log('hasPassword=', this.get('hasPassword'));
    console.log('hasUserName=', this.get('hasUserName'));
  })
});

// Export the component with an extra extension of our pre
// generated attributes and computed properties.
export default LoginComponent.extend(processedAttrs);

{%endhighlight%}

So obviously this got a little messy looking, as changes to optimize for performance usually make things. That said, the technique above could be abstracted out quite easily to wrap the Ember Component class and handle the necessary heavy lifting without all of the ceremony.  

## Summary

At the end of our exploration, we can see how it might be possbile to clean up and auto generate boilerplate Computed Properties. In the end things looked a litlle messy, and obviously you wouldn't use this technique to clean up two properties. 

That said, I feel like there's the bones of a simple validation system here. You could build a simple form validation system on this technique with some pre-canned validators that are higher order functions along the lines of the `has` function, for example `minLength`, `maxLength` and so on. Then the `UI_FIELDS` value would be expanded to an object, and each field name would have associated keys describing the validation to use, defaults, etc. 

It was an interesting diversion for me, and hopefuly you learned something new. 

_Eternal thanks to the ever patient and helpful Alex Matchneer, Todd Smith-Salter and Kelly Sutton for their input on this article!_
