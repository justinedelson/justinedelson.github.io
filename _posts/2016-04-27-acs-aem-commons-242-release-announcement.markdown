---
layout: post
title:  ACS AEM Commons 2.4.2 and 3.0.2 Released
date:   2016-04-27
image: /assets/article_images/2015-08-26-acs-aem-commons-2-released/logo.png
bodyClass: black-title
---

I'm pleased to announce the release of ACS AEM Commons 2.4.2 and 3.0.2.

This is a significant release in this project's history as we are now doing dual releases for compatibility between AEM 6.0, 6.1, and 6.2. As described in [this blog post](/2016/04/27/recompile-code-for-aem62.html){:target="_blank"}, there are a variety of non-backwards compatible changes in Java packages between AEM 6.1 and 6.2 which requires that many projects, including ACS AEM Commons be recompiled.

> If you are using 6.0 or 6.1, you will need to use versions in the 2.x line, 2.4.2 in this case. If you are using 6.2, you need to use versions in the 3.x line, 3.0.4 in this case.

We expect to maintain this dual version release process for a while, based on user feedback.

Aside from 6.2 compatibility, there are some new features in this release:

* [Fast Action Manager](http://adobe-consulting-services.github.io/acs-aem-commons/features/fast-action-manager.html){:target="_blank"} - A fluent API for parallization of synthetic workflows, replication and more.
* [Path-Based Render Condition](http://adobe-consulting-services.github.io/acs-aem-commons/features/path-rendercondition.html){:target="_blank"} - An implementation of the Granite UI Render Condition pattern which makes it easy to show and hide UI elements based on content paths.

There's also the usual assortment of bug fixes and minor enhancements.

Thanks to the following individuals for their contributions, complaints, ideas, and support:

* David Gonzalez (Adobe)
* Brendan Robert (Adobe)
* Kaushal Mall (Adobe)
* Chris Pilsworth (Adobe)
* Sreekanth Nalabotu (Adobe)
* Steffen Roesinger (Adobe)
* Johnny Yang (BizTech)
* Dominik Förderreuther (Adobe)
* Joseph Rignanese (Adobe)
* Florian Brunswicker (Adobe)
* Simo Tripodi (Adobe)

As always, while this projects are primarily produced and distributed within Adobe Consulting Services, it is available for usage by other Adobe groups, customers and partners. Contributions and feature suggestions from those groups are also most welcome. More information about this project, including detailed information about the included features, can be found [here](http://adobe-consulting-services.github.io/acs-aem-commons/){:target="_blank"}.