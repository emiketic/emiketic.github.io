---
layout: post
title:  "Hands On with the New Meteor 1.3"
author: mehdi
date:   2016-04-02 12:09:00
categories: meteor
tags: meteor npm es2015 es6 mochajs
image: /assets/article_images/2016-04-01-hands-on-with-meteor-1-3-release/emiketic-hands-on-meteor-1-3.jpg
---

[Meteor 1.3] is here and it's awesome! Much hype has been going on around it with good reason: 1.3 is a real leap
in the Meteor development track with impressive improvements in the way the platform integrates itself in the current JavaScript
eco-system.

We first heard about 1.3 during a [keynote] by [Matt Debergalis] from MDG (the Meteor Development Group) at their february SF dev workshop.
We couldn't wait to try it and had jumped on right away with testing the 8th beta.

# ES2015 Modules

Beside the support for modern ES6 syntax since 1.2, in Meteor 1.3, you can now write your own ES2015 modules without having to setup emulation tools such as RequireJS or CommonJS.
Managing modules export, import or considerations of circular dependencies or collisions are now matters handled by the platform natively.

Even better, you can package your own modules similarly to NPM modules for usage inside your app.

[The Meteor Chef] has written an excellent [post] on the matter.


# Native Support for NPM Modules and Package Management

One of the pains of former versions of Meteor, is that you often needed to rely on third-party Meteor packages such as [meteorhacks:npm]
in order to integrate NPM packages that you sometimes needed just for client functionality.
From a syntactical point of view, this approach isn't really easy and clean to maintain. But for the 1.3 release, handling
NPM in your Meteor app is as straightforward as managing any other regular NPM based setup. With your `package.json`
placed at the root of the project, you can simply run things like:
{% highlight bash %}
    npm install --react
{% endhighlight%}

As Matt Debergalis brought it up, enabling developers with the flexibility to chose between Meteor or NPM packages is particularly handy if you're integrating
technologies that are under rapid development, such us React. In our case, this proved to be quite helpful.
We use React back-to-back with Meteor quite a lot, and while trying to integrate the official React Velocity wrapper, we realised some of the
signatures and deps found in React 0.14.8 and needed for Velocity React, weren't met in the 0.14.3 based Meteor package.
Since both projects are neither maintained by the same team, nor released at the same pace, it's often wiser to go for the
official NPM source.

It is also possible now to test your modules in 1.3 as you would for NPM packages. This is discussed in the next section.

*And hey! Follow us on the Facebook channel [React Tunisia] for more updates about React :)*

# Testings and BDD

Following the announcement of [Xolv.io] to discontinue future developments of the [Velocity] testing engine, MDG has
now officially announced support for bundled testing utilities in Meteor thanks to a new meteor command dedicated to tests.
This is really exciting news because there will be no need to look for fragmented projects here and there to gear up your workflow with testing blocks.

Using [Mocha] or the testing driver of your choice, you can test both client and server side with reporting capabilities in the
browser:

{% highlight javascript %}
    meteor test --driver-package avital:mocha --port 3100
{% endhighlight %}

From now on, Meteor will support:

* Unit testings, typically for modules and small pieces of code
* Integration tests for cross-modules and cross-components. This makes it simpler to implement BDD practices within a continuous integration context
* Acceptance tests, or "end-to-end" test where you can leverage tools such as [React Test Utilities] to manage UI tests on the client side
* Load testing for performance and behaviour under stress loads

More resources can be found at the official [Meteor guide] for tests.

# Build Time Improvements

1.3 features a more efficient build toolchain for building Meteor apps faster, but there are still challenges
to face along the road. Improving build time and efficiency is one of the strategical key-points of the MDG
roadmap for the future releases.

# Mobile

While not being fans of non-native approaches to mobile, it is worth mentioning this latest release brings enhanced support
for Phonegap and Crodova drivers for trans-coded Meteor mobile web applications. Along with improved debugging utilities (both client/server side)
and faster build times.

# Conclusion
At [EMIKETIC], we opted for Meteor almost a year ago as the platform of choice for real-time and reasonably scalable web-applications.
We've been taking a close look at each new update the platform had to offer, and had the chance to suffer several of its limitations along the way.
Most of these were due to the lack of support of Meteor to some indispensable tools and libraries found in the JS World.

Luckily, 1.3 came to tackle this.


[Meteor 1.3]: http://info.meteor.com/blog/announcing-meteor-1.3
[EMIKETIC]: http://www.emiketic.com
[keynote]: https://youtu.be/7d0xTR-eYh0
[Matt Debergalis]: https://twitter.com/debergalis
[meteorhacks:npm]: https://github.com/meteorhacks/npm
[React Tunisia]: https://www.facebook.com/ReactJSTun/?fref=ts
[The Meteor Chef]: https://themeteorchef.com
[post]: https://themeteorchef.com/blog/meteor-1-3-from-a-20-000-foot-view/
[Xolv.io]: https://forums.meteor.com/t/velocity-update-xolv-io-are-handing-meteor-testing-back-to-the-mdg/13117
[Velocity]: https://github.com/meteor-velocity/velocity
[Mocha]: https://mochajs.org/
[Meteor guide]: http://guide.meteor.com/testing.html
[React Test Utilities]: https://facebook.github.io/react/docs/test-utils.html
