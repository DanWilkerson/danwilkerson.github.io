---
layout: post
---

Oft overlooked, error tracking is one of the most valuable ways to use Google Analytics. This essential guide covers how to use Google Analytics to track:

- [404 Errors](#404-errors)
- [Javascript Errors](#javascript-errors)
- [Modal Errors](#modal-errors)
- [Server-side Errors](#server-side-errors-universal-analytics-only)

Using Google Tag Manager, Universal Analytics, or Classic Analytics. Let's get to work!

## 404 Errors

404 errors occur when a client makes a request to your server for a resource that either no longer exists or never existed. Generally, we're more concerned about the former than the latter, although both can be a problem. By tracking when a user is served a 404 page, you can:

- Find malformed internal links and fix them
- Find valuable external links that point to old content and redirect them
- Improve the user experience of your site by directing users to relevant replacements

We'll use [Events](https://developers.google.com/analytics/devguides/collection/analyticsjs/events) to track 404 errors in our example. Events are the correct hit type for this kind of tracking, as they allow us to easily store all the data we need to locate and correct the issue, as well as prevent ourselves from polluting our page-based reports. 

First, we'll need to figure out a way of detecting when they actually happen. If you're developing the site yourself, this parts easy - you can just edit the contents of your standard 404 error page. If not, load a page on your own site that you know doesn't exist (be sure that there aren't any query parameters that could accidentally let things resolve). If you're not sure how to locate one, try http://www.example.com/hamster-suitcase, with your own domain instead of example.com. Now, check a few things on the page - does the text "404" or "File not found" appear anywhere on the page? How about in the <code>title</code> tag? We want to find something on the page that will almost always be there and clearly indicates that the page we've reached is a 404 page.

For those of you who are less technical, here are some things that would do the trick:
- Right-click the page and select "View Source" - find the <title> tag in the HTML and see if it has a phrase like "404" or "Page not found".
- Right-click on prominent elements on the page and see if they have either a class or id that reliably identifies them as uniquely part of this page (e.g. class="404-error-code").
- Check the URL of the page and see if it ends in something like "404.html".

Below we can see examples of both a title and on-page element that do the trick.

<img alt="example 404 page" src="{{ site.url}}/images/example-404.png" />

<div class="tabs">
  <div class="tab">
    <label for="gtm-1">
      Google Tag Manager
    </label>
    <input id="gtm-1" type="radio" checked="true" name="tabs-radio-1" data-category="gtm" />
    <div class="content">
      <p>Once we've located how we'll differentiate our 404 page, we'll create a Variable and Trigger combination that will fire when our indicator appears. If you're using the title, check that the Page Title Variable is enabled in the "Built-in Variables" section of the Variables screen. If you're using an on-page element, you'll need to create a DOM Element Varable, like this:</p>
      <img src="{{ site.url }}/images/404-on-page-variable.png" alt="example variable to determine if the page is a 404 page" />
      <p>If you're able, the best way to notify GTM that a 404 page is being viewed is to push that information into the data layer.</p>
<pre><code>  var dataLayer = window.dataLayer = window.dataLayer || [];
  dataLayer.push({
    'event': '404 Error'
  });
</code></pre>
      <p>Next, you'll need to make a Trigger that fires when you've detected a 404 page. It should look something like this:</p>
      <img src="{{ site.url }}/images/404-trigger.png" alt="example 404 trigger" />
      <p>Finally, you'll need to add a Tag to your container that fires your Google Analytics Event. If you're more of an advanced GTM user, instead of just passing the Referrer, pass the Referrer or, if it is undefined, 'No referrer'.</p>
      <img src="{{ site.url }}/images/404-error-event.png" alt="example 404 tag" />
    </div>
  </div>
  <div class="tab" data-content="ua">
    <label for="ua-1">
      Universal Analytics
    </label>
    <input id="ua-1" type="radio" name="tabs-radio-1" data-category="universal"/>
    <div class="content">
      <p>Once you've determined how to detect your 404 page, you'll need to trigger an Event. Fortunately the code to do so is pretty straightforward!</p>
<pre><code>  ga('send', 'event', {
    eventCategory: '404 Errors',
    eventAction: document.location.href,
    eventLabel: document.referrer || 'No referrer',
    nonInteraction: true
  });
</code></pre>
    </div>
  </div>
  <div class="tab">
    <label for="cl-1">
      Classic Analytics
    </label>
    <input id="cl-1" type="radio" name="tabs-radio-1" data-category="classic"/>
    <div class="content">
      <p>Once you've determined how to detect your 404 page, you'll need to trigger an Event. Fortunately the code to do so is pretty straightforward!</p>
<pre><code>  _gaq.push([
    '_trackEvent',
    '404 Pages',               // Event Category 
    document.location.href,    // Event Action
    referrer || 'No referrer', // Event Label
    0,                         // Event Value
    true                       // Could this session bounce
  ]);
</code></pre>
    </div>
  </div>
</div>

By setting the Action and Label to the Page URL and Referrer Variables, you'll end up with a simple drilldown report in the Behavior -> Events -> Top Events report. 

![example 404 event report]({{ site.url }}/images/404-events-report.png)

## JavaScript Errors

I like tracking JavaScript errors with Google Analytics for a few reasons:

- You can determine the browser and device that the error appeared on
- When a bug report comes in, you can get a better idea of the scope and impact by seeing how often users have run into it in the wild (remember that time executive X derailed your entire sprint to fix that one little issue?)
- You can find issues you didn't know existed on browsers that might be outside the scope of your usual tests

Just like with our 404 error tracking, we'll use events for this one.

<div class="tabs">
  <div class="tab">
    <label for="gtm-2">
      Google Tag Manager
    </label>
    <input id="gtm-2" type="radio" name="tabs-radio-2" checked="true" data-category="gtm"/>
    <div class="content">
      <p>
        Fortunately, GTM makes this one easy. JavaScript error listening is built right in. To enable it, first visit the Variables screen, click 'Configure', and enable the following Variables:
      </p>
      <ul>
        <li>
          Error Message
        </li>
        <li>
          Error URL
        </li>
        <li>
          Error Line
        </li>
      </ul>
      <p>
        Next, you'll need to create a JavaScript Error Trigger. This, too, is easy - just select it from the Trigger types, and leave it set to 'All JavaScript Errors'.
      </p>
      <img src="{{ site.url }}/images/js-error-trigger.png" />
      <p>
        Finally, you'll need to create a Google Analytics Event Tag. Here's my go-to setup:
      </p>
      <img src="{{ site.url }}/images/js-error-tag.png" />
    </div>
  </div>
  <div class="tab">
    <label for="ua-2">
      Universal Analytics
    </label>
    <input id="ua-2" type="radio" name="tabs-radio-2" data-category="universal" />
    <div class="content">
      <p>
        First, we'll need to start listening for JavaScript errors that occur. Once we've caught the error, we'll need to then trigger a Google Analytics Event. There's some wackiness between browsers in terms of what is (and is not) available, but here's a snippet to get you started:
      </p>
<pre><code>  // Fair warning - I've only tested this on Chrome, but it *should* cooperate on most browsers
  window.onerror = (function(original) {

    return function(message, file, line, column, error) {

      if (original) original.apply(this, arguments);

      var msg = message + ' | Line: ' + line;
      
      // Hedging against a loop here
      try { 

        // If the browser supports passing the error and stack, use that instead
        if (error && error.stack) msg = error.stack;

        ga('send', 'event', {
          eventCategory: 'JS Errors',
          eventAction: file,
          eventLabel: msg,
          nonInteraction: true
        });

      } catch(e) {}

    };

  })(window.onerror);
</code></pre>
    </div>
  </div>
  <div class="tab">
    <label for="cl-2">
      Classic Analytics
    </label>
    <input id="cl-2" type="radio" name="tabs-radio-2" data-category="classic" />
    <div class="content">
      <p>
        First, we'll need to start listening for JavaScript errors that occur. Once we've caught the error, we'll need to then trigger a Google Analytics Event. There's some wackiness between browsers in terms of what is (and is not) available, but here's a snippet to get you started:
      </p>
<pre><code>  // Fair warning - I've only tested this on Chrome, but it *should* cooperate on most browsers
  window.onerror = (function(original) {

    return function(message, file, line, column, error) {

      if (original) original.apply(this, arguments);

      var msg = message + ' | Line: ' + line;
      
      // Hedging against a loop here
      try { 

        // If the browser supports passing the error and stack, use that instead
        if (error && error.stack) msg = error.stack;

        _gaq.push([
          '_trackEvent',
          'JS Errors',               // Event Category 
          file,                      // Event Action
          msg,                       // Event Label
          0,                         // Event Value
          true                       // Could this session bounce
        ]);

      } catch(e) {}

    };

  })(window.onerror);
</code></pre>
    </div>
  </div>
</div>

Just like with our 404 Error Event, the structure and hit type we'll use will provide us with a simple, drill-downable report where we can see our errors. If you'd like to see the pages they occured on or the browser being used, make use of the [Secondary Dimension feature in GA](https://support.google.com/analytics/answer/6175970?hl=en).

## Modal Errors

Often, if "something went wrong", an application will display a modal window that tells the user more about what happened. Tracking these messages in Google Analytics is a great way to keep a record of how often users get hit with these kinds of problems. It's more helpful than you might think; I've caught several bugs with this method, including one instance where a slowly worsening bug led to users being unable to make purchases (!).

To track these, we'll need to add code to our site. If you're using GTM, you might be able to deliver the code from within your container (but be sure to coordinate with your development team, if so). For Classic or Universal Analytics, you'll have to bake the code directly into the site. Unfortunately, I can't advise you on how to "listen" for these - it will boil down to how you display your modals, which can be complicated and very custom. However, I can give you the syntax for how to handle these events when they occur.
<div class="tabs">
  <div class="tab">
    <label for="gtm-3">
      Google Tag Manager
    </label>
    <input id="gtm-3" type="radio" name="tabs-radio-3" checked="true" data-category="gtm"/>
    <div class="content">
    <p>Here's what your handler code might look like:</p>
<pre><code>  function gtmModalHandler(message) {

    var dataLayer = window.dataLayer = window.dataLayer || [];
    dataLayer.push({
      event: 'modalError',
      message: message
    });

  }
</code></pre>
    <p>With this in place, you'll need to create a (you guessed it) Trigger, Variable, and Tag in Google Tag Manager. These are pretty self-explanatory, so I'll just include some screenshots.</p>
    <p>The Trigger:</p>
    <img alt="modal error trigger" src="{{ site.url }}/images/modal-trigger.png" />
    <p>The Variable:</p>
    <img alt="modal error variable" src="{{ site.url }}/images/modal-variable.png" />
    <p>Finally, the Tag:</p>
    <img alt="modal error tag" src="{{ site.url }}/images/modal-tag.png" />
    </div>
  </div>
  <div class="tab">
    <label for="ua-3">
      Universal Analytics
    </label>
    <input id="ua-3" type="radio" name="tabs-radio-3" data-category="universal" />
    <div class="content">
    <p>Here's what your handler code might look like:</p>
<pre><code>  function gaModalHandler(message) {

    ga('send', 'event', {
      eventCategory: 'Modal Errors',
      eventAction: message,
      nonInteraction: true
    });

  }
</code></pre>
    </div>
  </div>
  <div class="tab">
    <label for="cl-3">
      Classic Analytics
    </label>
    <input id="cl-3" type="radio" name="tabs-radio-3" data-category="classic" />
    <div class="content">
    <p>Here's what your handler code might look like:</p>
<pre><code>  function gaModalHandler(message) {

    _gaq.push([
      '_trackEvent',
      'Modal Errors',               // Event Category 
      message,                      // Event Action
      '',                           // Event Label
      0,                            // Event Value
      true                          // Could this session bounce
    ]);

  }
</code></pre>
    </div>
  </div>
</div>

As with our other tracking, once in place, we'll have a nice, easy-to-use report in the Analytics interface for exploring the frequency and volume of modal errors on our site.

## Server-side Errors (Universal Analytics Only)

You can even track exceptions from your backend using Google Analytics. Although technically possible to achieve this with Classic Analytics, I'm only going to cover using Universal Analytics for this, as it is designed to be friendly to this kind of use case.

Universal Analytics implements a standard protocol for measurement that you can interact with easily from just about anywhere. If you're interested in learning more, [check out the official documentation](https://developers.google.com/analytics/devguides/collection/protocol/v1/parameters).

As a result, sending data to Google Analytics becomes trivial - simply cURL (or otherwise transmit) a payload like this (GET or POST):

    https://www.google-analytics.com/collect?
      v=1&  // The protocol version (currently only 1)
      t=event&  // Hit type
      tid=YOUR-UA-NUMBER&  // Property UA number
      cid=SOME-MADE-UP-NUMBER& // This is the User-level ID in GA
      ec=Serverside%20Errors&  // Event Category
      ea=<% error_message %>&  // Event Action
      el=<% addition_data_for_debugging %>&  // Event Label
      ni=1  // Could this session bounce

This functionality isn't just useful for errors: status codes, API calls, and more can all be tracked with Google Analytics in this manner.
Note: you'll almost certainly want to create a brand new Property to send this data to, as it won't play nicely with your user data that you collect client-side. Don't send this to your standard reporting property.

*Have any error-collecting techniques of your own that I missed? Tweet them to me at [@notdanwilkerson](https://twitter.com/notdanwilkerson).*
