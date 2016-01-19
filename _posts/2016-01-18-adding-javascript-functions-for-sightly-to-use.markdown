---
layout: post
title:  Adding JavaScript Functions for Sightly to Use
date:   2016-01-18
tags: aem sightly
image: /assets/article_images/2016-01-18-adding-javascript-functions-for-sightly-to-use/screenshot.png
---

One of the great aspects of the addition of Sightly to Adobe Experience Manager is that it allows for standard web technologies (HTML, JavaScript) to be used to build AEM components, something which used to require a knowledge of Java Server Pages (JSP) even for the simplest of components. Structurally, an AEM Sightly template (an HTML page) can invoke a corresponding JavaScript script, called a _Use_ object. 

> You can read more about how to use JavaScript Use objects in the [AEM Documentation](https://docs.adobe.com/docs/en/aem/6-1/develop/sightly/use-api-in-javascript.html)

JavaScript Use objects can do just about anything, but they are best suited to doing view-related data manipulation. For anything which is starting to look like complex business logic, you are going to want to refactor that into Java code, either a Java Use object, OSGi services, or a combination of both.

> There's a brief description of the pros and cons of the Java and JavaScript Use APIs [here](http://adobe.ly/1PBYo6n).

But let's say you started with a JavaScript Use object and then "outgrew" it and now need to integrate some backend services. One way of approaching that is to invoke Java methods from inside your Use object. The implementation of Sightly used in AEM uses the Mozilla Rhino project's implementation of JavaScript and Rhino provides a high level of interoperability between [Java and JavaScript](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Rhino/Scripting_Java). But this can get a bit verbose. For example, a Use object which gets a list of AEM Assets tagged with a particular tag would look something like this:

{% highlight javascript %}
use(function () {
    var tagManager = request.resourceResolver.adaptTo(Packages.com.day.cq.tagging.TagManager),
        tags = java.lang.reflect.Array.newInstance(java.lang.String, 1),
        taggedResources,
        taggedAssets = [],
        resource;
        
    tags[0] = "marketing:interest/product";
    taggedResources = tagManager.find("/content/dam", tags, true);

    while (taggedResources.hasNext()) {
        resource = taggedResources.next();
        if (resource.name === 'metadata') {
            taggedAssets.push(resource.parent.parent.adaptTo(com.day.cq.dam.api.Asset));
        }
    }
    
    return {
        assets : taggedAssets
    };
});
{% endhighlight %}

> For the purpose of illustration, I'm hard coding the tag identifier, although in a real case, you'd likely read this property from the current resource's properties.

This isn't awful, but by needing to reference Java class names in our JavaScript code (not to mention the need to create a new instance of an array via reflection) means that we are pretty much back to the state we were with JSPs -- component developers need to know some Java specifics in order to write a component. This code would also be pretty hard to unit test.

A different approach would be to further encapsulate this into an OSGi service and then invoke that service from your Use object with something like:

{% highlight javascript %}
use(function () {
    var taggedAssetFinder = sling.getService(Packages.com.myco.assets.TaggedAssetFinder);
    
    return {
        assets : taggedAssetFinder.find(request.resourceResolver, "marketing:interest/product")
    };
});
{% endhighlight %}

This is a lot better, but what I want to illustrate is a different technique which removes all references to Java classes from our code by injecting a custom object into the scope of the Use object. Specifically we're going to inject a function into the scope so that this code instead becomes:

{% highlight javascript %}
use(function () {
    return {
        assets : findTaggedAssets("marketing:interest/product")
    };
});
{% endhighlight %}

For this, we're going to use a relatively under-used, but very cool[^2] Sling feature -- the `BindingsValuesProvider`. When a script is invoked by the Sling scripting engine as part of request processing[^1] an object is created to store the list of global variables -- the request, the current resource, and so on. This object is called the script bindings and is an instance of `javax.script.Bindings`, a class defined in the [Java Scripting API](https://docs.oracle.com/javase/8/docs/technotes/guides/scripting/prog_guide/api.html). The Sling scripting engine itself only adds a handful of global variables (listed [here](https://sling.apache.org/apidocs/sling6/org/apache/sling/api/scripting/SlingBindings.html)); applications built on top of Sling, including AEM, are able to add additional global variables. This is how, for example, the `currentPage` object is made available to scripts.

There are two ways to use this feature -- an easy way and a less easy way. The easy way is that you can register an OSGi service using the `java.util.Map` interface and having a service property named `javax.script.name`. When Sling executes a script, it gets all of these services and adds their contents to the `Bindings` object which will be passed to the script engine. This way is useful for when are adding an object which doesn't require access to any of the existing bindings. For example, if we wanted to make the Java runtime version available as a global variable named `javaVersion` we could do something like this:

{% highlight java %}
@Component
@Service(Map.class)
@Property(name = "javax.script.name", value = "any")
public class JavaVersionBVP extends HashMap<String, Object> {

    public public JavaVersionBVP() {
        put("javaVersion", System.getProperty("java.runtime.version"));
    }
}
{% endhighlight %}

> The value `any` for the property `javax.script.name` indicates that this additional variable should be applied to any scripting language. If you only wanted to scope this to certain languages, the value would be the script engine name as we'll see below.

Now in a Sightly template, we can simply write:

{% highlight html %}
Currently running Java ${javaVersion}.
{% endhighlight %}

And see the value of that system property.

The less easy way is when we need access to objects from the current bindings. For this, you implement the interface `org.apache.sling.scripting.api.BindingsValuesProvider`. This interface has a single method, `addBindings` which is passed the current `Bindings` object. This allows you to get access to the current request, response, resource, and so on. Sling calls these services in the order of their OSGi service ranking, so you can ensure access to variables created by other `BindingsValuesProvider`s as well.

> A warning about performance - these BindingsValuesProviders get invoked on every script call, so you need to be very careful implementing them to ensure that they are performant. Use some kind of lazy loading or deferred invocation pattern wherever possible.

So now that we know how to add global variables, how do we create a new JavaScript function? As I mentioned above, the current implementation of Sightly uses the Rhino JavaScript implementation. Rhino provides a mechanism for new JavaScript functions to be defined *in Java* using the `org.mozilla.javascript.Function` interface, although in practice you will most likely use the `org.mozilla.javascript.BaseFunction` base class. There's really a single method you need to implement, named `call` (JavaDoc [here](http://bit.ly/rhino-function-javadoc)) which gets invoked when the function is... called. This method gets invoked with four parameters:

* `context` - this is the Rhino scripting context associated with the current thread. It has some utility methods for doing type conversions and working with the current thread.
* `scope` - the current JavaScript scope object.
* `thisObj` - the value of `this` when the function is executed. In Sightly Use objects, `scope` and `thisObj` will always be identical.
* `args` - an argument array.

If your function's arguments are booleans, numbers or strings, they will be passed as part of the `args` array as their corresponding Java type (the wrapper type for primitives, `java.lang.String` for strings). If however, they are JavaScript objects or other Java objects, the `args` array will contain instances of these classes:

* `org.mozilla.javascript.NativeArray` - represents a JavaScript array in Java.
* `org.mozilla.javascript.NativeObject` - represents a JavaScript object in Java.
* `org.mozilla.javascript.NativeJavaObject` - represents a Java object passed via JavaScript. The `unwrap` method can be called to access the original Java object.

Putting this all together, we can write a function to get the tagged assets:

{% highlight java %}
public class FindTaggedAssetsFunction extends BaseFunction {

    private ResourceResolver resourceResolver;

    public FindTaggedAssetsFunction(ResourceResolver resourceResolver) {
        this.resourceResolver = resourceResolver;
    }

    @Override
    public Object call(Context cx, Scriptable scope, Scriptable thisObj, Object[] args) {
        if (args.length == 0) {
            return Undefined.instance;
        }
        String tagId = (String) args[0];
        List<Asset> assets = new ArrayList<Asset>();

        TagManager tagManager = resourceResolver.adaptTo(TagManager.class);
        RangeIterator<Resource> resources = tagManager.find("/content/dam", new String[] { tagId }, true);
        while (resources.hasNext()) {
            Resource r = resources.next();
            if (r.getName().equals("metadata")) {
                Asset asset = r.getParent().getParent().adaptTo(Asset.class);
                assets.add(asset);
            }
        }

        return new NativeArray(assets.toArray());
    }

}
{% endhighlight %}

This code gets the tag identifier from the `args` array and then interacts with the `TagManager` and assets API to obtain the proper list of assets. It then wraps that list of assets into a `NativeArray` so that the JavaScript engine understands it as an array and not just a Java object.

> Strictly speaking, because Sightly is able to iterate over `java.util.List` objects, we don't necessarily need the wrapping of the list into a `NativeArray`, but this would be important if the array was going to be used elsewhere in our JavaScript Use object.

The corresponding `BindingsValuesProvider` implementation would be:

{% highlight java %}
@Component
@Service
@Property(name = "javax.script.name", value = "sightly")
public class FindTaggedAssetsFunctionBVP implements BindingsValuesProvider {

    @Override
    public void addBindings(Bindings bindings) {
        SlingHttpServletRequest request = (SlingHttpServletRequest) bindings.get(SlingBindings.REQUEST);
        if (request != null) {
            bindings.put("findTaggedAssets", new FindTaggedAssetsFunction(request.getResourceResolver());
        }
    }

}
{% endhighlight %}

I hope you have found this post enlightening and adds a new tool to your AEM development arsenal.

The code in this post can be found on [GitHub](https://github.com/justinedelson/sample-sightly-function).

[^1]: Or other contexts, but for the purpose of this post, we are just talking about request processing.
[^2]: I will admit to being biased because I originally added this feature to Sling.
