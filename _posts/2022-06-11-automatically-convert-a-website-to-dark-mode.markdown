---
title: "Automatically convert a website to dark mode."
layout: post
date: 2022-06-11 07:00
tag: 
  - frontend
  - javascript
image: false
headerImage: false
projects: false
hidden: false # don't count this post in blog pagination
description: "A quick way to implement dark mode to a website."
category: blog
author: blasferna
externalLink: false
---

![Screenshot](https://user-images.githubusercontent.com/8385910/173192842-488c18b5-16e9-42bd-8af8-f296502385dc.png)

A few days ago I had the need to apply the dark mode to a website and I didn't want to spend a lot of time in the conversion process, so I started looking for a utility to facilitate the process, luckily I found [Darkreader](https://github.com/darkreader/darkreader) an open source browser plugin that automatically applies dark mode.

Inquiring into the functionalities it offers, I found that it can be applied to a website using it as a javascript library, directly from a CDN or installed from NPM.


Today I bring a guide on how to apply it using its CDN version.

## Dependencies

* [Darkreader](https://github.com/darkreader/darkreader)

## Converting to dark mode

Add the dependency in the head of the HTML

{% highlight html %}
<script src="https://cdn.jsdelivr.net/npm/darkreader@4.9.46/darkreader.min.js"></script>
{% endhighlight %}


Create the toggle button to apply dark mode

{% highlight html %}
<input type="checkbox" id="dark-switch">
{% endhighlight %}

For the mode change to be persistent, it is necessary to store the user's preference in the browser's Local Storage.

Get user preference from Local Storage.

{% highlight javascript %}
  const getCurrentMode = function () {
    let mode = localStorage.getItem("mode");
    if (!mode) {
      mode = "light";
    }
    return mode;
  };
{% endhighlight %}

Function to change the mode according to the preference stored in the browser.

{% highlight javascript %}
  const changeMode = function (toggler) {
    let mode = getCurrentMode();
    if (toggler) {
      toggler.checked = mode === "dark";
    }
    if (mode === "dark") {
      DarkReader.setFetchMethod(window.fetch);
      DarkReader.enable({
        brightness: 100,
        contrast: 90,
        sepia: 10,
      });
    } else {
      DarkReader.disable();
    }
  };
{% endhighlight %}


The following function allows the toggler to be configured to apply the user's preference based on change events.

{% highlight javascript %}
 const configure = function () {
    const darkSwitch = document.getElementById("dark-switch");
    if (darkSwitch) {
      darkSwitch.addEventListener("change", function (e) {
        if (darkSwitch.checked) {
          localStorage.setItem("mode", "dark");
        } else {
          localStorage.setItem("mode", "light");
        }
        changeMode(darkSwitch);
      });
    }
    changeMode(darkSwitch);
  };
{% endhighlight %}

The following block of code allows the change to be applied safely without the user feeling the screen flicker when the library is making the change.


{% highlight javascript %}
  function ready(fn) {
    if (document.readyState !== "loading") {
      fn();
    } else {
      document.addEventListener("DOMContentLoaded", fn);
    }
  }

  ready(function () {
    configure();
  });

  changeMode();
{% endhighlight %}


Create a `dark_mode.js` file to include all the code explained above:

{% gist blasferna/8c9f4848f9b74a700ab2cddf4049bb6a %}

Finally add the `dark_mode.js` in the HTML head block.

{% highlight html %}
<script type="text/javascript"  src="./dark_mode.js"></script>
{% endhighlight %}


## Demo

You can see the code in action by accessing the following link [Darkreader](https://blasferna.github.io/auto-dark-mode/)

### Source

The demo page source code is available on [Github](https://github.com/blasferna/auto-dark-mode) with more details on how to apply it to a web site.


