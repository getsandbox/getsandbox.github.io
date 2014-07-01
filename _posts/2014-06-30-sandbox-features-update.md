---
layout: post
heading: New Features and Pricing
description: We've updated the dashboard, released a slew of features and finalised our pricing model.
author: ando
---

This month's platform update brings support for organisations, web editor, state viewer, performance improvements and more. Read on to see the changes now available:

### Introducing Organisations

Sandbox now supports Organisations in addition to standard users. Organisations simplify management of group owned mock applications, expand on our permissions system, and help focus your workflow for business and large projects.

If you've ever had to manage multiple accounts, wanted to add read-only collaborators, or needed to give someone else administratrive access over one of your mock applications, then you'll find Organisations solves these problems.

Creating an organisation helps you centralise your organisation's code. All mock applications live under the organisation, and billing will be attributed to the organisation account. Any user can create and administer an Organisation, and organisations are free to use.

<img src="/lib/images/2014_06_30_organisations.png" />

### Edit files in your browser

Edit your services and templates directly in your browser - and deploy instantly! If you don't enjoy using Git then get stuck into our web editor featuring full syntax highlighting, javascript linting, and validation. Saving your changes will perform the Git commit and push for you, and deploy your changes to all running instances of your code. Note that if your code is invalid when you submit, you will get an error and the changes will not be deployed.

<img src="/lib/images/2014_06_30_web_editor.png" />

### State viewer

A much requested feature, being able to view the current ```state``` object, is finally here. Available from any Application Instance page, you can traverse the state tree and view any object you've persisted. There's also search! Quickly locate any value or key in the state tree - have fun.

<img src="/lib/images/2014_06_30_state_viewer.png" />

### Pricing

Pricing has been finalised and made available. However, for a limited time Sandbox will continue to remain free to use while we finailise the implementation of trial plans. The tiered plans cater for small teams of developers through to enterprise plans with options for premium support. There is also a free tier available for individuals - it provides, for free, 1 instance of an application running and responding to requests.

<img src="/lib/images/2014_06_30_pricing.png" />



### Improved error messages and development feedback cycle

We've been working hard at improving the development feedback cycle and our highest priority item here has been to improve the error messages returned when there's a problem with your code. When pushing code changes, if your code fails the initial validation process an error will be returned describing the file responsible, the invalid code and the line number. Similarly, during runtime, should your code throw an exception, an error will be returned in JSON format with the same details as above.

We always welcome feedback from our users, you can tweet us or reach us here.