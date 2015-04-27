---
layout: post
heading: Clones and Forks
description: Cloning and forking Sandboxes enables you to support important test and collaborative processes, similar to what you might find in Git.
author: nick
---

### Clones

When you create a Sandbox, manually or via an import, it is created with route definition files (.js and .liquid), a Git repository, a state store and a unique HTTP endpoint. At this point you can add, edit and enhance your Sandbox to fit your test scenarios and that might be enough for your needs, but if you work in a team, or want better coverage you don't have to stop there.

A clone is a new Sandbox that refers to the original. It has a new unique endpoint and a separate state store from the source Sandbox, but refers to the same definition files and thus has no Git repository (only the original). Any change to the original definition files will be reflected on all unique endpoints, the original and any clones that may be created off it.

Clones can be really useful to support a couple of different test scenarios, the first is allowing each tester or client to have their own Sandbox environment. For example if you have a team of a couple of developers and a few testers, each tester could have their own Sandbox environment to test the application against. This not only lets them keep their Console activity separate for easy tracing, but if your Sandbox is stateful the clones let the testers work independently of each from a data point of view, which depending on your process could be a showstopper.

The second important clone use case is around negative testing. Because Sandbox clones share their definition files, it is possible to create a primary Sandbox to test positive test scenarios and then one or more clones to support failure scenarios (like an HTTP timeout). Using clones this way lets you achieve greater test coverage of areas where you typically might not have any coverage at all, these kind of negative tests are often the hardest to cover.

### Forks

A fork, much like in Git parlance, is a new Sandbox copied from the original. For all intents and purposes it is a completely separate and disconnected Sandbox (repository, state and endpoint), but the route definitions are copied across from the original at the time when the fork is completed.

A fork is useful when you have a Sandbox that might fulfill part of your needs functionally, an old version of an API for example. But you need to enhance that function further, or modify its behaviour in some way. While a clone gives you a unique endpoint, you cannot modify any of the definition files, to make modifications you have to edit the original. A fork takes a copy of the definition files at that point in time, letting you then make whatever modifications you need.

This can be useful in a number of scenarios, for example if you have a base Sandbox and want to quickly try out a new change without impacting existing consumers. However because a fork is permanently disconnected from the original Sandbox you should avoid making changes you want to keep in a fork, try and incorporate it into the original.

We will have a future post showing in more details how to use clones to cover negative test scenarios, and how it can help you. In the mean time [give it a try](https://getsandbox.com), and [hit us up](https://twitter.com/_getsandbox) with any feedback, questions or problems.