---
layout: post
title:  Does AEM Apps Require AngularJS?
date:   2015-10-14
tags: aem mobile
image: /assets/article_images/2015-10-14-does-aem-apps-require-angular-js/screenshot.png
---

TL;DR: No.

In talking with customers and partners about AEM Apps, one of the most frequently asked questions I hear is "If I want to build a hybrid mobile app with AEM Apps, do I need to use AngularJS?" Or sometimes this gets phrased as "Why do I need to use AngularJS to build hybrid apps with AEM Apps?" Or even "Why are you guys so in love with AngularJS? We'd rather use &lt;fill in the blank&gt;." So let me start by saying that no, there's no requirement to use AngularJS for hybrid app development with AEM Apps. There was one place where using AngularJS was highly beneficial, but that's actually not the case anymore and I'll come back to this later.

So where did this misconception come from? First and foremost, it comes from the pretty obvious fact that the hybrid sample application we ship with AEM Apps (Geometrixx Outdoors) uses AngularJS. And we do provide some specific utilities for building and exporting AngularJS views into a hybrid application. But the sample app is just that -- a sample and those utilities are specifically there to make AngularJS application management easier. Neither is meant to suggest a **requirement** to use AngularJS. At the end of the day (or, in this case, the beginning of the day), we had to choose something. If we chose ReactJS, odds are we would have this same misconception, just about ReactJS instead of AngularJS.

So no, you don't have to use AngularJS.

Except for one little thing…

With the version of AEM Apps available with the AEM 6.1 release this year, even if the application itself didn't use AngularJS, the splash screen (the screen displayed for the brief window of time when the application is initializing) did use AngularJS. And whereas customer applications could write their own splash screen JavaScript code, that probably wouldn't be a high priority item. However, that has recently changed.

A few weeks ago, Adobe release [AEM 6.1 Apps Feature Pack 2](https://www.adobeaemcloud.com/content/marketplace/marketplaceProxy.html?packagePath=/content/companies/public/adobe/packages/cq610/featurepack2/cq-6.1.0-apps-featurepack). The high profile features in this feature pack are [Deep Links for Push Messaging](http://adobe.ly/1VSQaKl) and [App Templates](http://adobe.ly/1VSQfOk). Less heralded, but interesting for the question posed in this entry's title is a new PhoneGap plugin -- the Content Sync Phonegap plugin. This replaces a combination of JavaScript code and the Chromium Zip Cordova plugin with a single point of entry. The end result is reduced client-side code (thereby making the removal of AngularJS easy) and a significant performance boost, about 10x on Android and 2x on iOS.

If you have already built a hybrid app using AEM Apps, updating it to use the new Content Sync plugin is simply a matter of adding the new plugin to your config.xml file and updating a few client library references. Examples of the changes can be found [here](https://github.com/justinedelson/pgehello/commit/7e04e8a724d9601aed5101aab4064700d109c2cb) and [here](https://github.com/blefebvre/aem-phonegap-kitchen-sink/commit/339039128956aa2fbb2ff89e0405a3d9772874a8). I've also updated my [Anatomy of a Hello World App document](https://github.com/justinedelson/pgehello/blob/master/anatomy.md) (described here) to reflect these changes.

With the release of Feature Pack 2, we should remove any doubt that you **must** use AngularJS with AEM Apps. And if you still have doubts, let me know in the comments and we'll try to clear up any outstanding misconceptions.