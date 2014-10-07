---
layout: post
title: "Decompiling Android Apps the Easy Way"
date: 2014-08-10 19:51
comments: true
categories: apps, android, decompile, tutorial
---
<img src="{{root_url}}/images/headers/android_apps.png"/>
### Introduction
Mobile applications are often viewed as black-box applications. However, these applications often suffer from the same (or similar) vulnerabilities as their web application counterparts.

In a [previous post](http://jordan-wright.github.io/blog/2013/11/07/how-to-pentest-iphone-apps-with-burp/), I showed how we can perform dynamic analysis on iPhone applications by intercepting the inbound/outbound traffic with the Burp proxy. In this post, we'll explore static analysis of Android apps by looing at a couple of online tools that make decompiling apps into equivalent Java and Smali code trivial.
<!-- more -->
### Step One: Get the APK
The first step in analyzing an Android app is to get the actual APK that's loaded onto the device. Fortunately, there is a site called [APK Downloader](http://apps.evozi.com/apk-downloader/) which allows us to do just that. By getting the link to the app of your choice from the [Google Play Store](https://play.google.com/store/apps?hl=en) and pasting it into APK Downloader, you can download a local copy of the app APK.

### Step Two: Unpack and Decompile
After we have the APK, we need to unpack and decompile the contents into its equivalent Java code. This used to be a pretty lengthy process using tools such as the [android apktool](https://code.google.com/p/android-apktool/). Fortunately for us, someone has also setup a web-based service which automatically performs the decompilation process on a given APK.

By uploading the APK to [decompileandroid.com](http://www.decompileandroid.com/), the user will be presented with the option to download the decompiled contents of the app, including the Java source code. *Awesome.*

### Things to Look For
Decompiling Android apps can allow researchers to analyze the exact code that is running on the app for vulnerabilities, as well as reveal any hardcoded credentials or other sensitive data. For example, in June of 2014, researchers from Columbia university created a tool called [PlayDrone](https://github.com/nviennot/playdrone) which automatically crawled thousands of Android apps found on the Google Play store and [discovered the existence of sensitive credentials](http://www.cs.columbia.edu/~nieh/pubs/sigmetrics2014_playdrone.pdf) in many of them.  

### Conclusion
Hopefully this short post not only shows researchers how to easily decompile Android apps, but also sends a gentle reminder to app developers that the source code of the app can easily be recovered. As such, precautions should be taken to ensure there is no sensitive information "hidden in plain sight". As always, leave any questions or comments below!

-Jordan