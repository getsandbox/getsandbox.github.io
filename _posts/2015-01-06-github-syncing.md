---
layout: post
heading: Automatic syncing of your Sandbox definitions to Github
description: Support for automatic syncing of Sandbox route definitions in and out of Github, now you can store your stubs with your code and tests.
author: nick
---

This month's platform update brings support for automatic two-way syncing your Sandbox definitions (main.js etc) with a Github repository of your choice and some API Blueprint importer bug fixes.

### Two-way Github Syncing

<img class="img-middle" style="width:720px; border: 0;" src="/lib/images/2015_01_06_github.png" />

Defining and having your stateful Sandbox stubs always running and testable in the cloud is great, but in a perfect world the stub definition should be in Git versioned alongside your code and most importantly, your tests!

The new two-way syncing feature solves this problem for you. You can configure at a Sandbox level which remote repository and what subpath inside that repository you want the sync with. Once configured any change in the remote repository (under the chosen subpath), or change in the Sandbox platform will trigger a sync.

The external repository syncing is available in the Sandbox Settings -> External Repository page. It requires that you grant OAuth permissions to your Github account, we use this to list your repositories, branches and to commit changes into your selected repository. 

Importantly the syncing process only checks out the files and folders under the given subpath, ```/test/stubs``` for example. So none of your other code, configuration etc will be ever copied to our servers.

At the moment this syncing feature only supports Github, but Bitbucket support is planned soon.

[Try it out now](https://getsandbox.com), and [hit us up](https://twitter.com/_getsandbox) with any feedback.