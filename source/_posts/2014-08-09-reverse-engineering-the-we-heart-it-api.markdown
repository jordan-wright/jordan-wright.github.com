---
layout: post
title: "Reverse Engineering the We Heart It API"
date: 2014-08-11 23:02
comments: true
published: false
categories: 
---
<img src="{{root_url}}/images/headers/weheartit_api.png"/>
### Introduction
Recently I came across the article from the The Washington Post describing We Heart It, a social network claiming [over 30 million users](http://www.washingtonpost.com/news/the-intersect/wp/2014/05/07/30-million-people-use-this-social-network-and-youve-probably-never-heard-of-it/). If you haven't seen it, We Heart It (from here on out abbreviated as WHI) is a social network which encourages people to post and share photos and images of things that inspire them. 

Having such a large user-base, I was interested in seeing what kind of API the site offered developers. However, I was disappointed when I found out that the API was [closed to "partners"](https://weheartit.com/partners), and even this was not a full REST API, rather it was a simple button developers can place on their website to allow users to interact with WHI.

With this being the case, I decided to take a look at the Android and iPhone apps using both static and dynamic analysis 

### Dynamic Analysis

Dynamic analysis is generally a bit quicker and can help give a good overview into how the application functions. By [intercepting traffic using a proxy]({{root_url}}/blog/2013/11/07/how-to-pentest-iphone-apps-with-burp/), we can easily analyze the structure of requests/responses in use by the application.

#### Registration & Authentication

Loading up the WeHeartIt iPhone app and attempting to register an account shows the following request:

```
```

*Note: I originally contacted the WeHeartIt security team since the login page (and every other page) was served via HTTP. This has since been fixed.*

### Static Analysis

Now that we have a feel of how the application operates, let's verify our findings by [decompiling the Android app]({{root_url}}/blog/) and doing some static analysis.

#### App Structure

#### APIRequest.java

#### 

### API Documentation
I have created a repository, weheartit-api, containing the full details found from this short study. The information in the repo is by no means comprehensive, but it will hopefully shed some light into the available API functionality.

### Other Fun Facts

While on the subject of the application, there are a couple of other interesting facts worth mentioning. For example, if your goal is to get general information about users on the network, there is no need to use the API! In fact, to help Google crawl the network, WeHeartIt publishes the following information in it's ```sitemaps``` files:

*  Username
*  Link to Profile
*  Date Modified
*  List of Collections

### Conclusion
In conclusion...

As always, let me know if you have any questions or comments!

Jordan