---
layout: post
title:  ACS AEM Commons 2.0.0 Released
date:   2015-08-26
tags: aem opensource
image: /assets/article_images/2015-08-26-acs-aem-commons-2-released/logo.png
bodyClass: black-title
---

I'm pleased to announce the release of ACS AEM Commons 2.0.0

This is a significant release in that we are **removing** support for AEM 5.6.1. For this release (and any future releases in the 2.x line), AEM 6.0 is considered the minimum platform.

New Features in ACS AEM Commons 2.0.0 include:

* [Font Awesome Icon Picker](http://adobe-consulting-services.github.io/acs-aem-commons/features/icon-picker.html) widget (for both Classic and Touch UI)
* [Authorizable Packager](http://adobe-consulting-services.github.io/acs-aem-commons/features/authorizable-packager.html)
* [Version Comparison Tool](http://adobe-consulting-services.github.io/acs-aem-commons/features/version-comparison.html)
* [Synthetic Workflow engine](http://adobe-consulting-services.github.io/acs-aem-commons/features/synthetic-workflow.html#synthetic-workflow-aem-workflow-model-support-v200) supports Workflow Models

We also ported all of the custom CoralUI 1.x interfaces to use CoralUI 2.x. As part of this work, a handful of AngularJS directives were created. Although these are primarily intended to be used within the project, they may be of use outside the project. More information can be found here.

Related to the removal of support for AEM 5.6.1 is that the [Dynamic Tag Management](http://adobe-consulting-services.github.io/acs-aem-commons/features/cloud-services.html#adobe-dynamic-tag-manager-dtm) cloud service is now deprecated. Anyone using it should be migrating to the out of the box cloud service.

Thanks to the following individuals for their contributions, complaints, ideas, and support:

* David Gonzalez (Adobe)
* Sreekanth Nalabotu (Adobe)
* Steffen Roesinger (Adobe)
* Simo Tripodi (Adobe)
* Brian Johnson (3Share)
* Koorosh Davoodian (Stanford University School of Medicine)

As always, while this projects are primarily produced and distributed within Adobe Consulting Services, it is available for usage by other Adobe groups, customers and partners. Contributions and feature suggestions from those groups are also most welcome. More information about this project, including detailed information about the included features, can be found [here](http://adobe-consulting-services.github.io/acs-aem-commons/).