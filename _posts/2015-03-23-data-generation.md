---
layout: post
heading: Generate mock data more easily with Faker library in your Sandbox
description: Often when building stubs you want details like addresses, names, emails etc to be realistic but creating them is a pain, use Faker library in Sandbox to make it simple.
author: nick
---

### Data generation helpers

This month's platform update brings support for data generation helpers in the form of making the Faker library accessible to all Sandboxes. Generally when building stubs you want your stub data to represent the values created by the real system, but when it comes to addresses, names, emails, phone numbers etc it is easier to hard code one than generate a new realistic value everytime.

With this month's platform update you can now use the Faker library to generate realistic data for many different properties increasing the value and accuracy of the Sandbox stubs you are creating.

There are quite a number of different properties available some of them include:

* firstName
* lastName
* fullName
* phoneNumber
* email
* city
* streetName
* streetAddress
* streetSuffix
* country
* state
* latitude
* longitude
#####...

Using the Faker library is simple, it is available to any of your JS code as a global variable already, no need to load or require() it in.

{% highlight javascript %}
Sandbox.define('/test', 'get', function(req, res) {
    // send a random state as the response
    res.send(faker.address.state());
});
{% endhighlight %}

The full list of available properties, including more details on functions and usages is available on the [Faker page](http://marak.com/faker.js/). 

As well as data generation helpers, we've added CORS support to the Sandbox Runtime component [available here](https://github.com/getsandbox/sandbox), which should help everyone testing JS apps locally, and a bunch of bug fixes across the platform generally.

[Try it out now](https://getsandbox.com), and [hit us up](https://twitter.com/_getsandbox) with any feedback.