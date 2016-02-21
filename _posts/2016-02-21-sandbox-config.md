---
layout: post
heading: Externalised JavaScript configuration
description: Adding a new external configuration feature, allows new dynamic interactions that were previous much more difficult.
author: nick
---

### Config!

A feature users have been asking for in different ways for a while was a way to make their Sandboxes more configurable, to be able to describe their Sandboxes logic in JavaScript but not have to keep updating and tweaking it to change environment style values. Our new feature ```Sandbox.config``` solves this problem!

The new config feature is already live, and the details of how it works are [here - config properties.](https://getsandbox.com/docs/config-properties) 

It is super simple, all the config values configured via the UI or API are exposed to your JavaScript as a JS map object on the ```Sandbox``` object. A simple example would be:

{% highlight javascript %}
Sandbox.define('/test', 'GET', function(req, res) {
  return res.send(Sandbox.config.responseText)

})
{% endhighlight %}

[Hit us up](https://twitter.com/_getsandbox) with any feedback, or [raise a ticket on GitHub](https://github.com/getsandbox/feedback/issues).