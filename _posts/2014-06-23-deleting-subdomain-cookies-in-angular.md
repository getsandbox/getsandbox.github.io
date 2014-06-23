---
layout: post
heading: Deleting Subdomain Cookies in Angular
description: A solution to a presumably fairly common problem with using the $cookies service to delete cookies on the client side in AngularJS
author: nick
---


Die cookie, die!
----------

We use AngularJS to build our frontend apps here at Sandbox HQ, normally Angular does all the heavy lifting for us but I came across a problem the other day where Angular kind of got in the way.

For our main web app, all backend requests are made against api.getsandbox.com, which means all session cookies are generated against that domain too. When you login/create a session we set the standard Secure/HTTP Only cookie that contains your private session ID, but also a JavaScript accessible cookie that contains your username (and maybe some other non-sensitive stuff in the future). We do this so we can render the page header with a logged in look, before we actually have received any of the context of who you are from the server, as that can take some few hundred milliseconds.

All of this works fine with Angular, the problem came when logging out the web app. When logging out we need to clear the client side username cookie, which can be done from JavaScript, and in theory should be do-able using the $cookies service. What I quickly found was that the $cookies service only seems to work for cookies that are set against the current fully qualfied domain 'getsandbox.com' for example. But since our web app is hitting 'api.getsandbox.com' for all requests the cookie domain was created as '.getsandbox.com' which means it works for all subdomains.


To make it work
---------

So to fix my frustrating problems of the invincible username cookie, I had to change this $cookies only code:

{% highlight javascript %}

delete $cookies['username']

{% endhighlight %}

to this, using both $cookies service, and the standard document.cookie property.

{% highlight javascript %}

var bareDomain = window.location.host
document.cookie = 'username=; Domain=.' + bareDomain + '; Path=/; Expires=Thu, 01 Jan 1970 00:00:01 GMT;';
delete $cookies['username']

{% endhighlight %}

In retrospect it makes sense, only the cookie name and value are exposed through to $cookies, and depending on the browser, a very specific string is required to match the existing cookie and override the value / delete it, and if you (like me) created the cookie in a different way than expected it won't get deleted!