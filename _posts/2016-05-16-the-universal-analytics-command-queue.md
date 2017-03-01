---
layout: post
---

For those of you who remember Google Analytics classic, the next four characters I write after this sentence should ring with nostalgia:

  _gaq

<code>\_gaq</code> is the name of the global variable that Google Analytics would install when it was executed on a page. The <code>_gaq</code> variable was defined (initially) as an Array. The default snippet had it right at the top:

  <script type="text/javascript">
      var _gaq = _gaq || [];
      _gaq.push(['_setAccount', 'UA-XXXXX-X']);
      _gaq.push(['_trackPageview']);

      (function() {
          var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
          ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
          var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
      })();
  </script>
    
The <code>\_gaq</code> array was a command queue for things we wanted Google Analytics to do. The default snippet passed it some stuff right away - specifically, it passed in the instructions to create a tracker and fire a pageview. This pattern was really cool - it allowed for commands to be registered in-line while the browser was busy fetching Google Analytics and was the backbone of GA's transition to asynchronous loading. No messy callbacks, no need to check if <code>_gaq</code> was defined; once Google Analytics loaded, each command would be executed in the order it appeared in the queue.

Another part of what made this exciting was that non-GA functions could also use the queue, and Google Analytics would call them for us. This allowed for clever folks to push in commands to run before Google Analytics did its tracker creation; for example, if I wanted to fire a callback function when Google Analytics loaded, I could do this:

  var _gaq = window._gaq = window._gaq || [];
    
    _gaq.push(function() {
      notifyGALoaded();
    });

The first thing GA would do after it finished loading and bootstrapping itself would be to call my function, which would call notifyGALoaded, in turn. I could be comfortable adding this code anywhere, because GA would inherit any existing <code>_gaq</code> values non-destructively. This meant the <code>.push()</code> method always would work.

Then Universal Analytics came along and ruined everything - the new syntax didn't use the familiar <code>.push()</code> syntax, and instead opted for commands to be pushed in as arguments to the function <code>ga</code>.

## etting Back Our Queue with the Alternative Syntax

Deep in the bowels of the Universal Analytics documentation, [there's an article that describes an alternative syntax for loading Google Analytics](https://developers.google.com/analytics/devguides/collection/analyticsjs/# lternative_async_tracking_snippet). We can use the first part of this snippet to get back our command queue:

    window.ga=window.ga||function(){(ga.q=ga.q||[]).push(arguments)};ga.l=+new Date;

This snippet instantiates our ga global and configures the internal command queue for us. Now we can go right back to adding commands into our queue:

    window.ga=window.ga||function(){(ga.q=ga.q||[]).push(arguments)};ga.l=+new Date;
    
  ga(function() {
      // Do some stuff
    });
    
Just like with our old <code>_gaq</code> Array, we needn't be concerned about detechting when Google Analytics loads or accidentially overwriting the existing global.

## o...?
You might be thinking "So, what? When would I need that anyways?". Imagine you're providing code to a 3rd party, and they want to dictate whether Google Analytics loads or not. You know Google Analytics *may* load, but you don't want to force it to load if it doesn't, and you don't want to poll every _n_ milliseconds to see if it **has** loaded. Now you can just instantiate the queue, push in your commands, and carry on worry-free.

  window.ga=window.ga||function(){(ga.q=ga.q||[]).push(arguments)};ga.l=+new Date;
    ga('create', 'UA-XXXXXX-YY', ...)
    // etc

You also may want to have code execute only after Google Analytics has loaded. Using this syntax, you can instantiate <code>ga</code>, push in your command, and be guaranteed that it will execute only after Google Analytics has loaded.

  window.ga=window.ga||function(){(ga.q=ga.q||[]).push(arguments)};ga.l=+new Date;
    
    ga(function() {
      // Some code I'd like to fire
    });

This simple interface provides a really simple way to queue up commands before and after the Google Analytics library loads. How will you use it? [Drop me a line](http://danwilkerson.com/contact) or on [Twitter](https://twitter.com/notdanwilkerson).
  
