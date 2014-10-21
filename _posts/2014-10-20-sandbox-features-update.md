---
layout: post
heading: Importing Apiary and Swagger
description: Improved code editing experience, generate sandboxes from Apiary and Swagger 
author: ando
---

This month's platform update brings support for generating mock services from Apiary and Swagger documentation, and an improved code editing experience.

### Generate Sandboxes from Apiary and Swagger

If you've invested the time and effort to design your web API with the excellent tools available from Apiary or Swagger, you can now import your API design spec to generate a mock API implementation with Sandbox - getting your development teams up and running even quicker. [Try it out now](https://getsandbox.com)

We love designing APIs with Apiary however their mock server only provides simple canned responses. You can now add custom behaviour, dynamic responses and more when you import your Apiary API spec into Sandbox.

We've added support for Swagger too, with support for more formats coming soon. [Hit us up](https://twitter.com/_getsandbox) with the format you're after and we'll prioritise it.

### Improved code editing experience

You've probably noticed that the dashboard and coding experience have undergone an overhaul. Everything you need to code, inspect and debug your sandboxes and API integration problems is now available on a single page. 

Sandboxes now support splitting your code into multiple modules. If you're familiar with NodeJS ```require``` to import modules then you'll know how to work with Sandbox imports right away. See the [docs](https://getsandbox.com/docs) for more details.

### Forking Sandboxes

Creating sandbox clones is a great way to ensure tests can run in parallel with completely isolated independent data and now you can tweak the behaviour of a sandbox without affecting other clones. Forking a sandbox provides you with your own copy complete with a Git repository to push changes too. Note that your forked sandbox will no longer receive updates from the original. Any sandbox you have access too can be forked from a sandbox's project page.

We always welcome feedback from our users, reach us [at twitter](https://twitter.com/_getsandbox).