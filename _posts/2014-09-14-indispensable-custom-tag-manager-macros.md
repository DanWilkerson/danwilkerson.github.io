---
layout: post
---

Variables are dynamic bits of information that can be used within Google Tag Manager Tags, Rules, and even between Variables. They look something like this: <code>{{Variable Name}}</code>. These can get the value of 1st Party Cookies, JavaScript Variables, Data Layer Variables, and the 'Custom JavaScript' macro lets you extend the functionality of Variables nearly limitlessly. For a great overview of everything on offer, check out [Simo Ahava's complete guide to all of the default Variables](http://www.simoahava.com/analytics/macro-guide-google-tag-manager/). Here's a list of some of my personal favorites.

### . User Agent
The User Agent is a string of text that identifies the ["software (a software agent) that is acting on behalf of a user"](http://en.wikipedia.org/wiki/User_agent). In our case, this is the browser and device used to access our website. For example, here's yours:

<span id="print-ua"></span>

You can test the User Agent to detect what browser a user is accessing your site with. This can be helpful if you want to track certain browser-based information (outside of what GA already collects) or change or exclude portions of your tag for users on particular browsers. 

Since the User Agent is stored in the <code>navigator.userAgent</code> property, all you have to do is create a JavaScript Variable Variable (yes, 'Variable Variable') like below:

{<1>}![Select the variable type 'JavaScript Variable' and enter 'navigator.userAgent' into the input](/content/images/2014/Sep/Screen-Shot-2014-09-18-at-8-01-44-AM.png)

For example, if you wanted to run a test and exclude any users of Internet Explorer 8 or below, you could do the following:
<pre>
var isIeEightOrBelow = {{User Agent}}.match(/MSIE [2-8]/i) > -1 ? true : false;
if (isIeEightOrBelow) {
  // some code
} else {
  // some other code, or nothing.
}
</pre>

You could also block an entire tag by using the same match as a blocking rule:
{<2>}![Blocking IE8 with Tag Manager](/content/images/2014/Sep/Screen-Shot-2014-09-11-at-10-40-58-PM-1.png)
<strong>WARNING:</strong> User agent tests are often unreliable. Never hinge site-critical functionality on a user agent test; use feature detection instead.

### . Week of the Year
Having the week of the year handy is more useful than you might first expect; one thing I like to do is store the week and year with a converting user, so that I can perform acquisition cohort analysis within Google Analytics. This snippet (courtesy of [RobG on Stack Overflow](http://stackoverflow.com/questions/6117814/get-week-of-year-in-javascript-like-in-php)) extends the built-in Date object to allow you to get the current week of the year.

<pre>
function() {
  Date.prototype.getWeekNumber = function(){
    var d = new Date(+this);
    d.setHours(0,0,0);
    d.setDate(d.getDate()+4-(d.getDay()||7));
    return Math.ceil((((d-new Date(d.getFullYear(),0,1))/8.64e7)+1)/7);
  };
  return new Date().getWeekNumber();
}
</pre>

This is a great way to segment data and learn how user behavior changes over time. This can also be helpful for analyzing the impact new products or features has on your business.

<strong>Update:</strong> With the new [Cohort Analysis Report](https://support.google.com/analytics/answer/6074676?hl=en), you can do some of this style of reporting natively. That said, you can only look at a limited set of cohorts in the data, so this is still a good back-up.

### . Mobile User
This is a snippet courtesy of [Michael Zaporozhets on Stack Overflow](http://stackoverflow.com/questions/11381673/javascript-solution-to-detect-mobile-browser); it's a function that will return <code>True</code> if the user is on a mobile device, according to their user agent.

<pre>
function() {
  window.mobilecheck = function() {
  var check = false;
  (function(a){if(/(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows (ce|phone)|xda|xiino/i.test(a)||/1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-/i.test(a.substr(0,4)))check = true})(navigator.userAgent||navigator.vendor||window.opera);
  return check; }
  return window.mobilecheck();
}
</pre>
This can be a useful piece of information if you'd like to prevent certain tags from firing on mobile devices, or if you'd like to exclude mobile users from a Content Experiment. The above warning about using the User Agent still applies.

### . The Page Title
This is a simple set of Macros that can come in handy for a number of reasons; say, for example, your site returns content on the same path, but with a huge variety in what content it returns based on some byzantine query string structure. In that case, the Page Title might expose some useful information. Fortunately, this is easily accessible through the <code>document.title</code> property. Just configure the Macro like you see below:
{<3>}![Select 'JavaScript Variable' for your Macro type, then enter 'document.title' in the input](/content/images/2014/Sep/Screen-Shot-2014-09-17-at-11-16-40-PM.png)

Of course, Google Analytics will already capture the Page Title for you, but should you need to combine that data with another data point for deeper context, this is handy to have.

### . The Meta Description
The meta description can also be a useful piece of information to have, either to pull data from or to push into Google Analytics. One example of how this can be used is to fire an event on every pageview that stores the meta description and the URL it was fired on; this can be helpful for tracking down duplicate or blank meta descriptions. Of course, you can also see this data inside the [HTML Suggestions section of Google Webmaster Tools](https://support.google.com/webmasters/answer/35624?hl=en).

<pre>
function() {
  var metas = document.getElementsByTagName('meta');
  for (var i = 0; i < metas.length; i++) {
    if (metas[i].name === 'description') {
      return metas[i].content;
    }
  }
  return 'None found';
}
</pre>

<strong>Update:</strong> Jim Gianoglio of [LunaMetrics](http://www.lunametrics.com/) fame pointed out that this could be accomplished JavaScript-free by using the below JavaScript Variable macro:
{<4>}![Example JavaScript Macro with document.head.children.description.content entered as the value](/content/images/2014/Sep/Screen-Shot-2014-09-25-at-12-43-20-PM.png)

The value entered above is <code>document.head.children.description.content</code>, which will return the Meta Description or <code>Undefined</code>. Google Tag Manager helpfully catches errors for missing properties.

### . The Environment
If your site is developed and runs in several different environments, such as a development environment, a staging environment, and a production environment, it can be useful to modify tag behavior based on which enviroment the tag is operating in. For example, I wouldn't want my Google Analytics tag sending data to my production Google Analytics account from activity taking place in my development area. This could dirty my data with extra transactions, events, and other site metrics. Even a few extra test transactions run for Quality Assurance can skew the data.

There are a few ways to handle this. 

#### .) Have your developers populate the environment in the Data Layer
You can have your development team add a Data Layer variable that specifies what environment the site is currently being loaded in, like this:
<pre>
var dataLayer = window.dataLayer|| (window.dataLayer = []);
dataLayer.push({'environment':'staging'});
</pre>

#### .) Detect the environment based on the hostname
If your development and staging environments live at different hostnames, like 'mysitestaging.com', you can create a Custom JavaScript Macro that returns the environment based on what it sees in the hostname, like this:
<pre>
function() {
  var hostname = {{Page Hostname}};
  if (hostname === 'mysite.com') return 'Production';
  if (hostname === 'mysitestaging.com') return 'Staging';
}
</pre>
<strong>Update:</strong> [Pavel Jašek](http://www.dlouhychvost.cz/) pointed out to me on Twitter that lookup tables could be used in place of custom JavaScript; this is a much better way to go about it. Something like below would suffice:
{<5>}![An example lookup table with a list of potential hostnames and their corresponding environment strings](/content/images/2014/Sep/Screen-Shot-2014-09-25-at-12-32-42-PM.png)
#### .) Detect the environment by testing something else
If, for some reason, your testing enviroment is operating at the same hostname that your production environment is in, you'll have to be creative and determine another authoritative test to run in order to figure out which environment you're running in. Here's a handful of ideas:

- Try and access a service available only in one environment, like an outside API
- Check the hostname of resources loaded on the page, like images, CSS, or JavaScript
- Search external stylesheets or JavaScript for comments - typically these will be minified out in Production

### . The Google Analytics UA Number
The UA number is the number associated with the Google Analytics property that a hit is being sent off to. Having this handy can be useful for either configuring things on the fly or testing data before takng certain actions. If your site only operates in one environment, this is a snap:
<pre>
function() {
  return 'UA-XXXXX-XX';  // Enter in your UA number here
}
</pre>
<strong>Update:</strong> The above is a valid, but more roundabout way to accomplish this - I've left it there for posterity, but I would recommend instead using a Constant macro for a single UA number, like below:
{<6>}![Example Constant macro with a Google Analytics UA number stored as the value](/content/images/2014/Sep/Screen-Shot-2014-09-25-at-12-39-12-PM.png)
However, if you'd like to use a different property for each environment (you should), then you might do something like this:
<pre>
function() {
  var environment = {{environment}};
  if ( environment === 'Production' ) {
    return 'UA-XXXXX-XX';
  }
  if ( environment === 'Staging' ) {
    return 'UA-YYYYY-YY'; 
  }
}
</pre>
<strong>Update:</strong> Per the above update, a lookup table would be a better option here. Here's what it might look like:
{<7>}![An example lookup table for a Google Analytics UA Number, keyed by environment](/content/images/2014/Sep/Screen-Shot-2014-09-25-at-12-35-33-PM.png)
Pavel also recommended [this article on lookup tables by Simo Ahava](http://www.simoahava.com/analytics/google-tag-manager-lookup-table-macro/).
### . Google Analytics User ID
Whenever a user visits your site, Google Analytics assigns them a unique, random identifier. This value is stored inside the cookie that Google Analytics sets on the browser. I like to capture it and store it inside Google Analytics as a Custom Dimension or Custom Variable; it can come in handy when trying to track down specific user behavior across visits or transactions.

There are two ways to get to this data - either by extracting it from the Google Analytics cookies directly, or, with the Universal Analytics library, asking for it politely.
#### Option 1: Cookie Sniffing
##### OR CLASSIC ANALYTICS:
<pre>
// Only works if __utma cookie has been set
function() {
  var __utmaCookie = document.cookie.match(/__utma=(.*?)(?:;|$)/);
  if(__utmaCookie && __utmaCookie[1]) {
    var utmaVals = __utmaCookie[1].split('.');
    if(utmaVals[2]) {
      return utmaVals[1] + '.' + utmaVals[2];
    }
  }
}
</pre>

##### OR UNIVERSAL ANALYTICS:
<pre>
// Only works if _ga cookie has been set
function () {
  var _gaCookie = document.cookie.match(/_ga=(.*?)(?:;|$)/);
  if(_gaCookie && _gaCookie[1]) {
    return  _gaCookie[1].match(/\d+\.\d+$/)[0];
  }
}
</pre>
#### ption 2: Asking Politely (Tracker Properties; Universal Only)
##### OR UNIVERSAL ANALYTICS WITHOUT A NAMED TRACKER (TYPICAL):
<pre>
function() {

  // Fetch the GA Library
  var gaLib = window[ window.GoogleAnalyticsObject ];
  var trackers = gaLib.getAll();
  var uaNumber = {{UA Number}};

  for ( var i = 0; i < trackers.length; i++ ) {
    if ( trackers[i].get('trackingId') === uaNumber ) {
      return trackers[i].get('clientId');
    }
  }
}
</pre>

#### OR UNIVERSAL ANALYTICS WITH A NAMED TRACKER (UNUSUAL):
<pre>
function() {

  var gaLib = window[window.GoogleAnalyticsObject];
  var tracker = gaLib.getByName({{Tracker Name}});
  
  if(tracker) {
    
    return tracker.get('clientId');
    
  }
  
}
</pre>
<script>
  document.getElementById('print-ua').innerText = navigator.userAgent || "Just kidding! I can't see yours.";
</script>
Those are some of my favorite Custom Variables; what are yours? Share with me on Twitter at [@notdanwilkerson](https://twitter.com/notdanwilkerson).
