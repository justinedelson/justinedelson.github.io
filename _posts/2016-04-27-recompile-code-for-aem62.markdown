---
layout: post
title:  Do I need to recompile my AEM code for 6.2?
date:   2016-04-27
image: /assets/article_images/2016-04-27-recompile/logo.jpeg
---

With the release last week of [Adobe Experience Manager 6.2](https://docs.adobe.com/docs/en/aem/6-2/release-notes.html){:target="_blank"} many developers are going to start
testing their applications on this new release. In the past, most AEM releases have been, at least at an API level, almost entirely backwards compatible, but with this release, the product management and engineering teams have taken the opportunity to remove some long-deprecated APIs. As a result, many applications will need to be either refactored or recompiled to work with AEM 6.2.

> Now might be a good time to review the [complete list of deprecated and removed features](https://docs.adobe.com/docs/en/aem/6-2/release-notes/deprecated-removed-features.html){:target="_blank"}

You will know recompilation is necessary if, when you install your application into an AEM 6.2 instance, your bundles fail to start and the log messages complain about various unresolved dependencies. Something like:

    Unable to resolve com.myco.myco-bundle [472](R 472.0):
	missing requirement [com.myco.myco-bundle [472](R 472.0)]
	osgi.wiring.package;
	(&(osgi.wiring.package=com.day.cq.commons)(version>=5.7.0)(!(version>=6.0.0)))

> The actual log message is more verbose, but this is the key part.

What this means is that the bundle `com.myco.myco-bundle`, which is bundle ID 472 has a dependency on a package named `com.day.cq.commons`, meaning that a bundle must be export this package. This dependency has a constraint on it which requires that the exported package's version must be greater than or equal to 5.7.0 and less than 6.0.

This constraint got that way because, in general, OSGi tooling (such as the `maven-bundle-plugin` and `bnd`) will automatically generate package import constraits based on the version of the exported package on the classpath at build time and assuming that [Semantic Versioning](https://www.osgi.org/wp-content/uploads/SemanticVersioning.pdf){:target="_blank"} is being followed. In this case, the package `com.day.cq.commons` was exported in AEM 6.1 with the version 5.7.0.

But before doing some trial and error, it is actually possible to predict whether or not these types of errors will occur by looking at the changes in package versions. If your application depends upon *any* package which had a major version change between AEM 6.1 and 6.2, it will at minimum need to be recompiled. While every application will be different, what I have observed is that the key packages are these:

* `com.day.cq.commons`
* `com.day.cq.commons.jcr`
* `com.day.cq.replication`
* `com.day.cq.security`
* `com.day.cq.workflow`
* `com.day.cq.workflow.exec`

[Click here](/assets/aem62/major-package-changes.txt){:target="_blank"} for a list of the packages which have had major version changes between AEM 6.1 and 6.2.

In addition, there are some packages which have been entirely removed. These are long-deprecated features and, in some cases, packages which were never intended to be used. These have, at least in my observations, been much less problematic for early adopters, but one to call out specifically is `org.apache.sling.event`. [Click here](/assets/aem62/removed-packages.txt){:target="_blank"} for the full list. If your application depends upon any of these, it will need to be refactored. To see an example of such a refactoring, you can take a look at the [first set of changes made for 6.2 support in ACS AEM Commons](https://github.com/Adobe-Consulting-Services/acs-aem-commons/commit/d633dd4db10dfc20cd19a89ec9aaa4b7819242c9){:target="_blank"}.

## Updating the UberJar

Assuming you need to recompile, the first thing you need to do is update your project to reference the AEM 6.2 UberJar. As documented on [https://docs.adobe.com/docs/en/aem/6-2/develop/dev-tools/ht-projects-maven.html](https://docs.adobe.com/docs/en/aem/6-2/develop/dev-tools/ht-projects-maven.html){:target="_blank"}, the UberJar is available in two different versions:

* An obfuscated version which is freely available on [repo.adobe.com](http://repo.adobe.com){:target="_blank"}.
* An unobfuscated version which can only be obtained by licensed customers and partners through Adobe Support (Daycare).

There are a variety of cases when you *need* the unobfuscated version -- certain types of unit tests, certain Declarative Services use cases, etc. These are laid out in the documentation. It is worth noting that these use cases **haven't changed between AEM 6.1 and AEM 6.2**. In other words, if you were using the obfuscated version before, you can continue to use the obfuscated version. And if you needed the unobfuscated version, you still need it.

To use the 6.2 UberJar, update the dependency in your project's _pom.xml_ file to:

{% highlight xml %}
<dependency>
    <groupId>com.adobe.aem</groupId>
    <artifactId>uber-jar</artifactId>
    <version>6.2.0</version>
    <classifier>obfuscated-apis</classifier>
    <scope>provided</scope>
</dependency>
{% endhighlight %}

Or, if you are using the unobfuscated version:

{% highlight xml %}
<dependency>
    <groupId>com.adobe.aem</groupId>
    <artifactId>uber-jar</artifactId>
    <version>6.2.0</version>
    <classifier>apis</classifier>
    <scope>provided</scope>
</dependency>
{% endhighlight %}

Once this is done, simply build your application. If you have compiler errors, those are most likely due to removed packages or classes and the referencing code needs to be refactored.

