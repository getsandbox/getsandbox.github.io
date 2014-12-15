---
layout: post
heading: Options for testing your iOS and Android apps, APIs and backends with stubs.
description: Deep dive into the unique challenges of testing an API and webservice driven mobile app. We cover of the different options available and their tradeoffs, and what we think is the best approach.
author: nick
---

### A problem

Building an API or web-services backed mobile app poses some unique problems that don't really come up together in other applications. There are a couple of things that make them unique:

- Need an API call to retrieve any resource not shipped in the app itself
- Better user experience if the app bundle is small (faster updates etc.)
- Generally pushing a new app bundle is a multi day/week journey (Apple approvals etc.)
- Its mobile, which means by definition you are testing on a number of different devices, in different places.

What this means in terms of testing your API backed app is that you might encounter three types of issues:

- You want to build out your UI, wire everything together and start validating the design and flow. But your API doesn't exist yet!
- Your API is being built, but isn't publically accessible for your wider set of testers. Can't test everything sitting at your desk, on Wi-Fi, hitting your local server.
- The API is built and deployed! But to test edge cases and rainy day scenarios the testers need more control over how the API behaves to cause errors.

### Options

There are a few different approaches and options to solve these problems; they all focus around simulating or stubbing out the real backend API at some level. Nothing beats the real API, but that isn't always practical, the closer we get to the real API the more valuable (but potentially costly) our testing will be.

#### Fixtures

A fixture is a codified stand-in of a particular object in your application; it is generally used for unit tests where the desire is to test one particular unit in isolation. It is obviously possible to use a fixture to simulate backend API calls, but it can cause problems with later testing.

One of the most important aspects of integration testing, is testing the integration between the app and your backend and to do that properly the request should go over the network. This step is obviously missed when using fixtures, as they are part of the app itself. 

The reason this is important is a large portion of these integration problems occur in the serialization/deserialization and protocol conversions (think Content-Types, Cookies, JSON types, XML!! etc.) and these only take place if you actually make a request!

#### Write a stub

A stub or mock is a small often throw-away application that aims to simulate as much of the target API as required. The simplest and most common implementation of this is a stub that responds to requests with a different 'canned' response, so you get a valid API response but it is the same response every time you make the request. These can obviously get more and more complicated as time goes by, with more and more varied canned responses to simulate different behaviours.

There are *many* different libraries out there to make the job of writing a mock easier, they all generally focus around making it very easy to send a canned response to a given request. But this is only half of the problem, many API operations are not just Retrieves, what about Create, Update and Delete? It is generally very difficult to get a stub to do more than just retrieves well, making testing your app against the stub a limited and unnatural experience.

#### MBaaS!

A relatively recent addition to the landscape, a MBaaS (Mobile Backend as a Service) aims to take away the pain of building a backend entirely, so you can just worry about building the app. Parse and Firebase (sort of) are examples of these.

In the MBaaS approach, because you build the app against their backend from Day 1, you always have a backend available to develop and test against avoiding many of the problems discussed earlier. 

However this introduces new problems, the biggest worries for me are around vendor lock-in, lack of a real API and describing business logic. 

The lock-in occurs because each of the MBaaS providers do it differently and once you have spent the significant amount of time integrating and making everything work, changing can be a rewrite. 

Similarly the API exposed by the provider to manipulate your data is generic, not purpose built for your app or your data. This means instead of your API being a reusable asset, it becomes unsuitable for consumption outside your app.

Finally describing your business logic, a tough enough job as it stands already, is made more difficult by the added abstraction and constraints of the MBaaS service. Each of them support business logic in a different way but obviously none are as simple, maintainable and portable as building it yourself.

#### Sandbox

 We conceived [Sandbox](https://getsandbox.com) to solve these problems, so it is not surprising that we think it does the best job.

 Sandbox is a cloud hosted stubbing platform that allows you (and your collaborators) to create stubs ranging from simple canned responses, to complex fully simulated applications, all in a simple Git versioned JavaScript environment. 

 As well as the simple Sandbox runtime ([open source on github](https://github.com/getsandbox/sandbox)), we built a set of tools on top of this to make developers and testers jobs easier ([more details](https://getsandbox.com/features)). To support the often tricky jobs like debugging requests, modifying responses and applying non-functional behaviour like response delays.

 In Sandbox you are creating routes to simulate your real API. You start by either mocking out your API manually or preferably import your API definition. We support definitions such as Apiary markdown, WSDLs or Swagger to shortcut the process. 

 Most Sandbox implementations start out with simple canned responses, but can be easily extended to support logic and stateful behaviour. Each Sandbox has access to its own zero-effort persistent storage. This means simulating proper CRUD APIs is really simple, [check out the api](https://getsandbox.com/docs/sandbox-api)

 Sandbox solves the challenges of API integration by giving teams an unlimited number of always-on and accessible drop-in replacement environments, a simple scripting runtime and powerful debugging tools. Whether you have an existing API your testers want more control over (failure scenarios are important!), or you are still designing your API and want to import your design and test the UI flow before proceeding, that's why we built it!

 [Try it out for free!](https://getsandbox.com)