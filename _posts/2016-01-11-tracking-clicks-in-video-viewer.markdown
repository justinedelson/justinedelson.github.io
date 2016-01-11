---
layout: post
title:  Tracking Clicks in the AEM Interactive Video Viewer
date:   2016-01-11
tags: aem assets
image: /assets/article_images/2016-01-07-tracking-interactivity-in-video-player/screenshot.png
---

AEM Assets includes a great feature called Interactive Video which allows for videos to be easily enhanced with links. While this is primarily intended for commerce use cases, it can be used for education, promotions, or really any case where you want to easily link a video to other content. You can read more about this feature in the [Documentation](https://docs.adobe.com/docs/en/aod/overview/working-with-assets/dynamic-media/interactive-videos.html) and watch a great video walkthrough [here](https://outv.omniture.com?v=s4NHQ2dzqd7hIqWjeG2sIdyNWsTWyupA).

Interactive Video supports two different types of links: quickviews and hyperlinks. Quickviews are designed allow the user to see further information such as product details and even add a product to their cart without navigating away from the current page. Hyperlinks are simple hyperlinks which can either open a new browser tab or navigate in the current tab.

Links are displayed in two separate contexts. While the video is playing, they are displayed in a sidebar. These links are context-sensitive, meaning that the links associated with the current video segment are displayed.[^1] Upon completion of the video, all links are also displayed in a Call to Action screen.

In this post, I'll describe how to add reporting calls so that you can track how many users are clicking on these links as well as in which context (sidebar or Call to Action). For the purpose of this post, I'm using Adobe's [Analytics](http://www.adobe.com/marketing-cloud/web-analytics.html) and Adobe [Dynamic Tag Management](http://www.adobe.com/solutions/digital-marketing/dynamic-tag-management.html) services, but the process would be the same for any analytics/tag management solution.

You can see this code in action on [this demo page](/demos/interactive-video.html).

## Register Event Listeners

The first thing we have to do is register some event listeners. In the code below `videoViewer` is an instance of the JavaScript class `s7viewers.InteractiveVideoViewer` created as part of the Interactive Video embed code. As part of the initialization code for the viewer, `videoViewer`'s `setHandlers()` method is called with a list of event handlers. In the `initComplete` handler, we can get access to two of the components within the viewer and register event listeners for user interactions. These two components `interactiveSwatches` and `interactiveThumbnailGridView` correspond to the sidebar and Call to Action end slates respectively. In this case, we use the same listener for both events

{% highlight javascript %}
"initComplete":function() { 
  var interactiveSwatches = videoViewer.getComponent("interactiveSwatches"),
      callToAction = videoViewer.getComponent("interactiveThumbnailGridView"); 

  interactiveSwatches.addEventListener(s7sdk.event.UserEvent.NOTF_USER_EVENT, onUserEvent, false);
  callToAction.addEventListener(s7sdk.event.UserEvent.NOTF_USER_EVENT, onUserEvent, false);
}
{% endhighlight %}

## User Interaction Event Listener

The event listener (`onUserEvent` in this case) is responsible for setting the proper variables in the data layer and then firing a DTM Direct Call Rule. As a data layer, we are using a global JavaScript object named `digitalData`. The `event` child object has these properties:

* `link` - the link the user clicked on. Will either be a URL in the case of a hyperlink or a query-string style (e.g. sku=12345) collection of key/value pairs in the case of a quickview.
* `linkType` - either `hyperlink` or `quickview`.
* `location` - either `sidebar` or `cta`.

Note that we could have used two separate event listeners, but since the link parsing logic is the same in both cases, it made more sense to reuse the event handler and just have some conditional logic to specify the `location` property based on the event type.

{% highlight javascript %}
function onUserEvent(event) {
    var trackLink = false,
        eventData = {};
    if (event.s7event.data.length > 0 &&
        typeof event.s7event.data[0] === "string") {

        eventData.link = event.s7event.data[0];
        eventData.linkType = "hyperlink";

        if (eventData.link.indexOf("quickview") === 0) {
            eventData.linkType = "quickview";
            eventData.link = eventData.link.substring(10); // strip off 'quickview:'
        }
        if (event.s7event.trackEvent === s7sdk.event.UserEvent.INTERACTIVE_SWATCH) {
            eventData.location = "sidebar";
            trackEvent = true;
        } else if (event.s7event.trackEvent === s7sdk.event.UserEvent.INTERACTIVE_THUMBNAIL_GRID_VIEW_SWATCH) {
            eventData.location = "cta";
            trackEvent = true;
        }
        if (trackEvent && _satellite) {
            digitalData.event = eventData;
            _satellite.track("video-click");            
        }
    }
}
{% endhighlight %}

## DTM Data Elements

In DTM, we have some [Data Elements](http://blogs.adobe.com/digitalmarketing/analytics/getting-started-dtm-data-elements/) set up to extract these variables from the data layer so that they can be passed easily to Analytics. For each of the properties of the global `digitalData.event` object, we have a corresponding Data Element. Since the value of this object will change during the lifespan of this page (i.e. if a user clicks on multiple images), each Data Element uses the Custom Script type.

![DTM Data Element](/assets/article_images/2016-01-07-tracking-interactivity-in-video-player/data-element.png)

The script for each one is simple returning the appropriate value:

{% highlight javascript %}
return digitalData.event.linkType;
{% endhighlight %}

## DTM Direct Call Rule

Finally, we can create a DTM [Direct Call Rule](http://webanalyticsfordevelopers.com/2015/11/03/direct-call-rules-what-are-they-good-for/)[^2] which fires the desired Analytics event when a user clicks on one of these images. We use the Data Elements created above to map to 3 separate eVars 

![DTM Data Element](/assets/article_images/2016-01-07-tracking-interactivity-in-video-player/evars.png)

Once all of the pieces are in place, clicking on any of the images in the sidebar or the end slate causes an Analytics call with the proper values passed as the eVars.

## Conclusion

With a little bit of JavaScript additions, we can add click tracking to AEM's Interactive Video viewers so that we know how many users are clicking on which of the links in our viewers.

[^1]: Depending on the size on the player and the number of links associated with the current segment, links associated with other segments may also be displayed.
[^2]: While a DTM Event Based Rule could be fired based on the user's click, the link data is not easily accessible, so a Direct Call Rule is more effective.