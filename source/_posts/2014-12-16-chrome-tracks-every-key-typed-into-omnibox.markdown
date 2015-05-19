---
layout: post
title: "Chrome Tracks Every Key Typed into the Omnibox"
date: 2014-12-18 23:43
comments: true
published: true
categories:
- browsers
---

<img src="{{root_url}}/images/headers/browser_tracking.png"/>
> Friendly Reminder: Browser makers may track every key you type in the URL bar

### Introduction
Technologies like Google Chrome's Omnibox makes searching easier. However, these quick search suggestions come at a price. This post is a friendly reminder that you may want to consider turning off predictive search to protect your privacy.
<!--more-->
### How the Omnibox Works
Predictive search isn't magic. Chrome doesn't come shipped with a built-in collection of popular searches. No, the only way to get suggestions for what you are searching is to *ask Google*. This happens by sending a request to Google for search suggestions **for every key typed in the omnibox** - by default.

What does this look like? After installing Burp's CA certificate and starting ```chrome.exe``` with the ```--allow-ssl-mitm-proxies``` option, we can see the following requests pop up as we search:

```
GET /complete/search?client=chrome-omni&gs_ri=chrome-ext&xssi=t&q=o
GET /complete/search?client=chrome-omni&gs_ri=chrome-ext&xssi=t&q=om
GET /complete/search?client=chrome-omni&gs_ri=chrome-ext&xssi=t&q=omg
GET /complete/search?client=chrome-omni&gs_ri=chrome-ext&xssi=t&q=omgw
GET /complete/search?client=chrome-omni&gs_ri=chrome-ext&xssi=t&q=omgwt
GET /complete/search?client=chrome-omni&gs_ri=chrome-ext&xssi=t&q=omgwtf
GET /complete/search?client=chrome-omni&gs_ri=chrome-ext&xssi=t&q=omgwtfb
GET /complete/search?client=chrome-omni&gs_ri=chrome-ext&xssi=t&q=omgwtfbb
GET /complete/search?client=chrome-omni&gs_ri=chrome-ext&xssi=t&q=omgwtfbbq
```

>*But I'm searching these things anyway!*

Sure, this might be fine for searches, but what else could be sent via this method? Hostnames, web addresses, and IP addresses are ***all*** sent to Google before you press enter. This means that Google knows if you visit a website, even if you don't visit the site from search results.

#### Disabling Omnibox
Since this setting is enabled by default, here's how to disable it:

<span class="fa-stack fa-lg fa-1x">
<i class="fa fa-circle fa-stack-2x" style="color:#019875"></i>
<i class="fa fa-stack-1x" style="color:white; font-family:inherit;">1</i>
</span> Navigate to ```chrome://settings```

<span class="fa-stack fa-lg fa-1x">
<i class="fa fa-circle fa-stack-2x" style="color:#019875"></i>
<i class="fa fa-stack-1x" style="color:white; font-family:inherit;">2</i>
</span> Click "Show Advanced Settings"

<span class="fa-stack fa-lg fa-1x">
<i class="fa fa-circle fa-stack-2x" style="color:#019875"></i>
<i class="fa fa-stack-1x" style="color:white; font-family:inherit;">3</i>
</span> Uncheck "Use a prediction service to help complete searches..."

<img src="{{root_url}}/images/blog/browser_track/omnibox.png"/>

### Chrome Isn't the Only One
It's important to note that Chrome isn't the only browser that has this capability. Internet Explorer has the same feature from Bing. The only difference is that this isn't default behavior, and has to be explicitly enabled.

### Putting Things in Perspective
This isn't new information - more of a friendly reminder. It's important to put these privacy "risks" in perspective and determine what is more important to you - keeping your browsing history and IP/hostname scheme private, or getting solid search suggestions.

As always, let me know if you have any questions/comments below.

-Jordan ([@jw_sec](http://twitter.com/jw_sec))
