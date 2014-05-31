---
layout: post
heading: URL Encoding and Camel HTTP Endpoints
author: nick
---


A simple integration...
----------

Here at Sandbox we use Apache Camel for all our routing and mediation, most of the time it makes hard things really easy, but recently we had to integrate with a system over HTTP that used an escaped path parameter as its input. So the request needed to look like this:

```
http://api.somesystem.com/resource/get/RootDir%2FSubDir%2FFilename
```

So a seemingly simple task, we need to pass in a URI style value in the path parameter of a request and all reserved URI characters need to be URL encoded, so they can be presumably decoded on the other side.

We are using the Camel Jetty component for outbound (or consumer in Camel speak) endpoints, so I setup my route like this:

```Java
from("direct:callSystem")
.bean(bean, "doSomething")
.recipientList(
	simple("jetty://http://api.somesystem.com/resource/get/${urlEncodedPathParam}")
)

```

Passing in a URL encoded version of my desired path parameter, 'RootDir%2FSubDir%2FFilename'. But the resulting jetty call resolved to this:

```
http://api.somesystem.com/resource/get/RootDir/SubDir/Filename
```

Where did my encoding go!?
--------

So after much digging around the Camel source code, it appears all endpoints that inherit from the core HttpEndpoint class auto decode values that make up the Endpoint URI. In my desire to make it work and move on I didn't dig into why this is, but the solution to the problem is to just escape the values a few more times.



To make it work
---------

So after a bit of digging around, I eventually was able to get Camel to produce the correct Endpoint URI by **triple encoding the value**. Camel must have some decoding logic that picks up a second encoding, but a triple encoded value seems to get processed the perfect amount!

Hopefully that saves someone from having to step through the source code like I had to.