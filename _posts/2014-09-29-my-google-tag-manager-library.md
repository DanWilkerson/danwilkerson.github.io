---
layout: post
---

In programming, compiled collections of code that add functionality are typically called 'Libraries'. They're often offered up for free to the community to use, build upon, and improve. I'd like to do the same for the web analytics world; so I've collected a few of my favorite Google Tag Manager tags here.

I've tried to pick tags that are great functional additions to any Google Tag Manager configuration. I'd also like for you to contribute your favorites (or offer suggestions on where I can improve mine); I'll periodically add them to the list, or I might even make a follow-up post collecting your suggestions. To contribute, hit me up on Twitter at [@notdanwilkerson](https://twitter.com/notdanwilkerson) or email me at [notdanwilkerson@gmail.com](mailto:notdanwilkerson@gmail.com).

### 1. Store Cookie
A straightforward tag; this takes values you pass into the Data Layer and creates a cookie on the users browser with them. I've configured mine to fire when {{ "{{event"}}}} equals 'setCookie'.
<pre>
&lt;script&gt;
  var cookieName    = {{ "{{cookieName"}}}};
  var cookieValue   = encodeURI({{ "{{cookieValue"}}}});
  var cookieExpires = parseInt({{ "{{cookieExpires"}}}}) ? new Date(new Date().getTime() + 1000 * 60 * 60 * 24 * {{ "{{cookieExpires"}}}}) : 0;
  var cookieDomain  = {{ "{{cookieDomain"}}}};
  var cookiePath    = {{ "{{cookiePath"}}}};

  document.cookie   = cookieName + '=' + cookieValue + '; Expires=' + cookieExpires + '; Domain=' + cookieDomain + '; Path=' + cookiePath;
&lt;/script&gt;
</pre>
You'll need to configure Data Layer macros for each of the macros above, like below:

![Example dataLayer macro that captures the value of the dataLayer variable 'cookieExpires'](/images/tml-1.png)

I like to set default values for each; here's how I might configure mine for this website:

cookieName: \_\_dfc  // Default Cookie Name
cookieValue: 1  // Can evaluate to True
cookieExpires: 0  // Makes the cookie a session cookie
cookieDomain: danwilkerson.com
cookiePath: /

Now, storing something in a cookie will be as easy as pushing some data into the Data Layer.

<pre>
&lt;script&gt;
var dataLayer = dataLayer || [];
dataLayer.push({
  'event': 'setCookie',
  'cookieName': '__nomnom',
  'cookieValue: 'yum',
  'cookieExpires: 365,
  'cookieDomain': '.danwilkerson.com',
  'cookiePath': '/articles/'
});
&lt;/script&gt;
</pre>

Then you can create a first party cookie macro to access this cookies value.

![Example first party cookie macro that gets the value of our example cookie, __nomnom](/images/tml-2.png)


### 2. Fire A Google Analytics Event
This has been a huge time saver for me; I mentioned it in my first post about [tracking errors using Google Analytics](/google-analytics-error-tracking-guide/). The format is pretty similar to the above; we're just using the built-in Google Analytics Event tag, in this case.

![Screenshot of example Event tag, with each event input filled in with corresponding Data Layer values](/images/tml-3.png)

I use the firing rule {{ "{{event"}}}} equals 'eventFired' to trigger this tag. This makes sending events to Google Analytics simple; just push something like this into the Data Layer:

<pre>
&lt;script&gt;
  dataLayer.push({
  'event': 'eventFired',
    'eventCategory': 'My Event',
    'eventAction': 'My Event Action',
    'eventLabel': 'My Event Label',
    'eventValue': 15,
    'eventNonInteraction': true
  });
&lt;/script&gt;
</pre>

### 3. Optionally Fire Custom Dimensions And Metrics With Events

Something I've been experimenting with recently has been to add Custom Dimension and Metric macros to my generic events so that I can add that data in if need be. I do this by adding the following values to the Custom Dimensions and Custom Metrics:

![An example of using macros for custom dimensions and metrics to allow for piggybacking with normal events](/images/tml-4.png)

This makes adding custom data to an event as simple as adding <code>'customDimIndexA': 1, 'customDimValueA': 'Data'</code>. I use letters rather than numbers to prevent mixing up dimension keys with the keys being used for the event.

One example of how to use this would be to capture and store user milestones, grouped by the week milestone achievement (using the <code>{{ "{{Week Of The Year"}}}}</code> macro from my last post).
<pre>
&lt;script&gt;
  var dataLayer = dataLayer || [];
  dataLayer.push({
    'event': 'eventFired',
    'eventCategory': 'Customer Milestone',
    'eventAction': 'Submitted Lead Form',
    'eventLabel': weekAndYear,
    'eventValue': 100,
    'eventNonInteraction': true,
    'customDimIndexA': 1,
    'customDimValueA': {{ "{{Week Of The Year"}}}}-{{ "{{Year"}}}}
  });
&lt;/script&gt;
</pre>

#### One Small ~~Problem~~ Feature

I've discovered that Tag Manager will reference the most recent value for a Data Layer variable, even with a default value set, so if you leave a field blank and you've pushed another event with a set value for that variable, you'll wind up copying it's value instead of leaving the field blank. E.g.:

<pre>
&lt;script&gt;
/**
 * Event looks like:
 * Category: Page Load
 * Action: DOM Ready
 * Label: /this-page
 * Value: 100
 * Non-Interaction: True
 * Custom Dimension 1: 4.38 Seconds
**/
  var dataLayer = dataLayer || [];
  dataLayer.push({
    'event': 'eventFiredWithCustomData',
    'eventCategory': 'Page Load',
    'eventAction': 'DOM Ready',
    'eventLabel': {{ "{{page path"}}}},
    'eventValue': 100,
    'eventNonInteraction': true,
    'customDimIndexA': 1,
    'customDimValueA': {{ "{{Page Load Time"}}}}
  });
&lt;/script&gt;

// Later ...

&lt;script&gt;
/**
 * Event looks like:
 * Category: Customer Milestone
 * Action: Submitted Lead Form
 * <strong>Label: /this-page
 * Value: 100
 * Non-Interaction: True
 * Custom Dimension 1: 4.38 Seconds</strong>
**/
  var dataLayer = dataLayer || [];
  dataLayer.push({
    'event': 'eventFiredWithCustomData',
    'eventCategory': 'Customer Milestone',
    'eventAction': 'Submitted Lead Form'
  });
&lt;/script&gt;
</pre>

This means you have to fill out every one of the values for this tag each time you fire it, even if they're simply '0' or ''. I'm curious if anyone else has had this problem, and what solutions they've used to tackle it. I'd like to write a post at some point on how best to deal with this.

<strong>Update:</strong> The very helpful [Simo Ahava](https://twitter.com/SimoAhava) let me know this was both intended and useful behavior. He also wrote a very detailed post on [how Tag Manager and the Data Layer interact](http://www.simoahava.com/analytics/google-tag-manager-data-model/). In order to avoid duplicate values for tags that fire twice, be sure to fill out every value in the object you push into the Data Layer.

### 4. Fire Analytics After Test Variation Is Set
If you're running a test using Content Experiments, you need to bucket your user into an experiment group before the first pageview is sent to Google Analytics. Putting the experiment code above your Tag Manager snippet will avoid this issue.

But, if you're running your experiment in Google Tag Manager using the [Content Experiment API](https://developers.google.com/analytics/solutions/experiments-client-side), you'll need to delay your default pageview from firing until after this happens.

<strong>Note:</strong> You probably <strong>shouldn't</strong> run your test in Google Tag Manager; ideally, test bucketing and changes occur before anything is rendered on the page, and if your snippet is after the opening &lt;body&gt; tag, page elements might load before you can handle this appropriately. Make sure that this is a good solution for your needs, but defer to putting the snippet as high in the &lt;head&gt; as possible.

I handle this by creating blocking rules that match the conditions users might be bucketed into the test, then adding the following after calling the <code>cxApi.chooseVariation()</code> method:
<pre>
&lt;script&gt;
  var dataLayer = dataLayer || [];
  dataLayer.push({
    'event': 'variationChosen'
  });
&lt;/script&gt;
</pre>

Then I copy my Google Analytics tag and set its firing rule to {{ "{{event"}}}} equals 'variationChosen'. If you're using the copy and paste tag from the configuration menu, this shouldn't be an issue.

<i>Have a tag you'd like to contribute, or notice something I could tweak and improve? Tweet it to me at [@notdanwilkerson](https://twitter.com/notdanwilkerson) or email it to me at [notdanwilkerson@gmail.com](mailto:notdanwilkerson@gmail.com).</i>
