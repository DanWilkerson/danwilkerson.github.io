Oft overlooked, error tracking is one of the most valuable ways to use Google Analytics. This essential guide covers how to use Google Analytics to track:
###### click a link to jump ahead)
- [404 Errors](# 04errors)
- [Javascript Errors](# avascripterrors)
- [Server-side Errors](# erversideerrors)
- [Modal/Dialog Errors](# odaldialogerrors)

Using:

- Google Tag Manager
- Universal Analytics
- Classic Google Analytics

## 04 Errors

Tracking 404s is great for:

- Finding broken links (internal and external)
- Proactively catching broken functionality
- Tidying up older site architecture

Use Events to capture the previous page of the visit and the complete location of where the 404 occured. These are available at <code>document.referrer</code> and <code>document.location.href</code>, respectively. These two pieces of information can let you track down exactly where the offending link happens to be; capture these instead of relying on the 'Page' or 'Full Referrer' dimension in Google Analytics. Those values can easily be altered by filters, campaign parameters, or by other features built into Google Analytics.

#### OR GOOGLE TAG MANAGER:
If you're using Google Tag Manager, checking the title element for the default '404' text is a sturdy test to fire a tracking event. Something like <code>{{title}} equals '404 Page'</code> or <code>{{title}} contains '404 Page'</code>, as recommended by Samantha Barnes in her [excellent guide to capturing 404 metrics with Google Tag Manager](http://www.lunametrics.com/blog/2014/08/19/404-errors-google-analytics-google-tag-manager/). You can also reference other tags that are on the page - be creative and find a standard value to watch for. 

If that's not possible, you'll need to get your developers to populate the dataLayer with an event that you can watch for in Tag Manager, like:
<pre>
var dataLayer = dataLayer||[];
dataLayer.push({'event':'404 Error'});
</pre>

Then, you add a rule that watches for the 404 Error event, like so:

{<1>}![](/content/images/2014/Aug/Screen-Shot-2014-08-21-at-11-47-47-PM.png)

Then you create a Google Analytics Event tag and populate the Event Action and Event Label with the full URL of the page and the referrer.

{<2>}![](/content/images/2014/Aug/Screen-Shot-2014-08-20-at-1-09-37-PM.png)

As a best practice, I recommend that you try and decouple the mechanism that fires the event into Google Analytics from the actual event. Doing this means you can use a single 'generic event' tag to handle the processing of all of your events, and instead just push data into the dataLayer for the event tag to reference. For example:

{<3>}![](/content/images/2014/Aug/Screen-Shot-2014-08-20-at-1-06-27-PM.png)

In the above, each macro simply returns a corresponding dataLayer variable for an event value. It watches for 'event':'eventFired' in the dataLayer. To generate a 404 event, all that needs to happen is the following be pushed into the dataLayer:

<pre>
var referrer = document.referrer;
if (referrer === '') {
  referrer = 'No Referrer';
}
dataLayer.push({
  'event': 'eventFired',
  'eventCategory': '404 Error',
  'eventAction': document.location.href,
  'eventLabel': referrer,
  'eventValue': 0,
  'eventNonInteraction': true
});
</pre>

This pattern makes adding future events a snap - all you have to do is push the 'eventFired' event into the dataLayer with the corresponding data; no need to create a whole new Google Analytics event tag.

If you're not using Google Tag Manager, you'll have to manually fire these events into Google Analytics. You can either include the following code as part of the 404 page template, or add it wrapped in a test into the header of each page on your site, e.g.:
<pre>
if (document.title === 'Oops! You've found a 404 Page') {
  // Appropriate event tracking code from below goes here
}
</pre>

#### OR UNIVERSAL ANALYTICS:
<pre>
var referrer = document.referrer;
if (referrer === '') {
  referrer = 'No Referrer';
}
ga('send', 'event', '404 Error', document.location.href, referrer, 0, {'nonInteraction': true});
</pre>

#### OR CLASSIC GOOGLE ANALYTICS:
<pre>
var referrer = document.referrer;
if (referrer === '') {
  referrer = 'No Referrer';
}
_gaq.push(['_trackEvent','404 Pages', document.location.href, referrer, 0, true]);
</pre>

## avaScript Errors
JavaScript errors are errors that the browser throws while executing the code that you've delivered to it. There are many common causes for JavaScript errors - browser compatibility, coding mistakes, and namespace collisions, to name a few. Tracking JavaScript errors can help you ensure a consistent user experience across browsers, catch bugs that were missed during development, and gauge the impact an error is truly having on users so that a fix can be appropriately prioritized.

#### OR GOOGLE TAG MANAGER:
It's your lucky day. Google Tag Manager offers a JavaScript Error Listener right out of the box. Just select 'JavaScript Error Listener' from the 'Event Listeners'; I recommend you add this to every page.

{<4>}![](/content/images/2014/Aug/Screen-Shot-2014-08-20-at-1-13-16-AM.png)

Then, you'll need to add another tag that listens for 'gtm.pageError' and uses the associated values to populate an Event for Analytics. I recommend reusing the pattern above for events, like this:
<pre>
dataLayer.push({
  'event':'eventFired',
  'eventCategory': 'JS Errors',
  'eventAction': {{gtm.errorMessage}},
  'eventLabel': URL: {{href}} | File: {{gtm.errorUrl}} | Line: {{gtm.errorLineNumber}},
  'eventValue': 0,
  'eventNonInteraction': true
});
</pre>

Otherwise, you'll have to create another tag specifically for turning errors into events, like this:

{<5>}![](/content/images/2014/Aug/Screen-Shot-2014-08-20-at-1-18-30-PM.png)

I make the Event Action the <code>gtm.errorMessage</code> value and the Event Label <code>URL: {{href}} | File: {{gtm.errorUrl}} | Line: {{gtm.errorLineNumber}}</code>. This makes an for an easy-to-read format that can be drilled into for deeper context.

#### OR UNIVERSAL ANALYTICS & CLASSIC ANALYTICS:
To track errors in the browser, you'll need to bind an event listener to the <code>window.onerror</code> event that will generate a hit to send to Google Analytics. Doing this in a 100% backwards-compatible, error-free way [can be super tricky](http://alistapart.com/article/modern-debugging-tips-and-tricks), but here's a copy-and-paste-friendly snippet you could utilize that should work with most browsers and Google Analytics configurations. Feel free to copy and modify to your tastes:
<pre>
if (typeof window.onerror === 'object' && window.onerror === null) {  // Checks if this seat is taken/exists
  window.onerror = function(message, file, lineNumber) {
    if (typeof ga === 'object') {  // Detects Universal Analytics
      ga('send','event','JS Error', message, 'URL: ' + document.location.href + '| File: ' + file + '| Line: ' + lineNumber, 0, {'nonInteraction': true});
    } else if (typeof _gaq === 'object') {  // Detects Classic Analytics
      _gaq.push(['_trackEvent','JS Error', message, 'URL: ' + document.location.href + '| File: ' + file + '| Line: ' + lineNumber, 0, true]);
    }
  }
}
</pre>

## erver-side Errors
Sending application error data into Google Analytics directly from your server let's you connect backend performance with front-end behavior. It can also contextualize seemingly unconnected issues across your site.
#### OR GOOGLE TAG MANAGER:
Although technically not _server-side_, you could expose some error information into the dataLayer and then pass it into Google Analytics. On your server, this might look something like:
<pre>
var dataLayer = dataLayer||[];
&lt;?php
foreach ( $err in $errors ) { ?&gt;
  dataLayer.push({
    'event': 'eventFired',
    'eventCategory': 'Server Error',
    'eventAction': '&lt;?php echo $err->errno . $err->errstr ?&gt;',
    'eventLabel': 'URL: ' + {{href}} + '| File: &lt;?php echo $err->errfile ?&gt; | Line: &lt;?php echo $err->errline ?&gt;';
  });
&lt;?php } ?&gt;
</pre>

And on the client, you'd see:
<pre>
&lt;script&gt;
  var dataLayer = dataLayer||[];
  dataLayer.push({
    'event': 'eventFired',
    'eventCategory': 'Server Error',
    'eventAction': 'Fatal error: Undefined class constant "MYSQL_ATTR_USE_BUFFERED_QUERY"'
    'eventLabel': 'URL: http://example.com/test/ | File: /local/www/example.com/includes/database/mysql/database.inc | Line: 43',
    'eventValue': 0,
    'eventNonInteraction': true
  });
&lt;/script&gt;
</pre>

#### idenote:
You'll notice me using the syntax <code>var dataLayer = dataLayer||[];</code> and then using <code>dataLayer.push({})</code> in order to populate the dataLayer in my code; I recommend this as best practice to avoid accidentally overwriting needed information in the dataLayer. Basically, this is read by the browser as 'The dataLayer is equal to itself, or it's an empty array'. If the dataLayer doesn't exists, the next section of the 'OR' (||) statement is evaluated, namely 'dataLayer is an empty array'. In this way, if it exists, it will preserve itself, and if it doesn't, it will create a new array named dataLayer for us to use. 

Not instantiating dataLayer in this way can lead to problems if your code gets messy. For example, if you're loading the dataLayer at the top of the page, then server errors inside the &lt;content&gt; of the page, this could happen:
<pre>
&lt;!-- Other HTML --&gt;
&lt;script&gt;
  var dataLayer = [{'PageType':'Local'}, {'timestamp': 2849200492}];
  console.log(dataLayer); // Logs '[ { "PageType": "Local" }, {'timestamp': 2849200492} ]'
&lt;/script&gt;
&lt;!-- Other HTML --&gt;
&lt;script&gt;
  var dataLayer = [{'event': 'Server Error'}];
  console.log(dataLayer); // Logs '[ { "event": "Server Error" } ]'
&lt;/script&gt;
</pre>

Should you need to reference either the PageType or timestamp values later, they would return <code>undefined</code>. Using <code>dataLayer = dataLayer||[];</code> and then <code>dataLayer.push()</code> instead of just instantiating the dataLayer prevents this from happening.

#### OR UNIVERSAL ANALYTICS:
The [Universal Analytics Measurement Protocol](https://developers.google.com/analytics/devguides/collection/protocol/v1/parameters) provides a relatively painless way to send hits to Google Analytics directly from your server. All it takes is a POST request, which can be pretty simple to put together. Here's a quick and dirty example using NodeJS:

<pre>
var http = require('http');

function fireMeasurementProtocolEvent(category, action, label, value, nonInteraction, cid) {

  var ni = !nonInteraction ? '1' : '0';

  var eventData = "?v=1&tid=UA-XXXXXXX-XX&t=event&cid=" + cid + "&ec=" + category + "&ea=" + action + "&el=" + label + "&ev=" + value + "&ni=" + ni;
  
  var payload = {

    'hostname': 'www.google-analytics.com',
    'path': '/collect' + eventData,
    'method': 'POST'

  };

  var req = http.request(payload);
  req.end();

};
</pre>

#### OR CLASSIC GOOGLE ANALYTICS:
The truly intrepid can manufacturer fake \_\_utm.gif requests - all you need to do is generate a GET request to http://www.google-analytics.com/\_\_utm.gif with the correct parameters appended to the request. Here's a [great reference from Google](https://developers.google.com/analytics/resources/articles/gaTrackingTroubleshooting?csw=1# ifParameters) on how to do just that.

## odal & Dialog Errors
Finally, a great 'error' type that I like to catch is any kind of modal or error dialog presented to a user. Even mundane error messages can reveal more sinister patterns lurking underneath. Tracking and analyzing common site messages is essential for good site hygiene.

#### OR GOOGLE TAG MANAGER:
Either create an event tag and a macro to grab the text of the modal, or using the generic tracking event outlined above:
<pre>
var modal = document.getElementById('modal-message');
dataLayer.push({
  'event': 'eventFired',
  'eventCategory': 'Error Modal',
  'eventAction': modal.textContent,
  'eventLabel': '',
  'eventValue':0,
  'eventNonInteraction': true
});
</pre>

#### OR UNIVERSAL ANALYTICS:
<pre>
var modal = document.getElementById('modal-message');
ga('send', 'event', 'Error Modal', modal.textContent, '', 0, {'nonInteraction': true});
</pre>

#### OR CLASSIC GOOGLE ANALYTICS:
<pre>
var modal = document.getElementById('modal-message');
_gaq.push(['_trackEvent','Error Modal', modal.textContent, '', 0, true]);
</pre>

*Have any error-collecting techniques of your own that I missed? Tweet them to me at [@notdanwilkerson](https://twitter.com/notdanwilkerson).*
