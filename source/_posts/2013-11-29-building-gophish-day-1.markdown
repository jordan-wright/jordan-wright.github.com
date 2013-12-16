---
layout: post
title: "Building Gophish - Day 1"
date: 2013-11-29 16:15
comments: true
published: true
categories: 
- gophish
- go
---
<img src="{{root_url}}/images/headers/gophish.png"/>
### Introduction
Since the [Simple Phishing Toolkit (SPT)](http://sptoolkit.com/the_end.php) was discontinued, I've wanted to create a simple, effective, and open-source phishing toolkit. In recent years, we've seen a rise in spear-phishing attacks targeting large organizations, most of which are largely successful. The goal of this toolkit will be to provide businesses and penetration testers with the ability to quickly and easily perform in-house or contracted phishing engagements, and track the responses to see where improvements can be made. This toolkit will be called [gophish](https://github.com/jordan-wright/gophish).

In addition to this, I've been casually poking around at [```go```](http://golang.org/) for a while now, and have decided it would be good to finally put it to use in a larger project. I'm a fan of seeing the steady development and updates of projects as they are created. I believe it can help keep the developer motivated and the users informed and involved, so this is what I'm going to do. Hopefully, these posts will allow others to learn alongside me, as well as spur improvements from experienced ```go``` developers so that [gophish](https://github.com/jordan-wright/gophish) can be the best product possible.

With that being said - let's get started!
<!--more-->
### Why Go?
In addition to learning a new language, here are a few reasons why I chose ```go``` for this project:

* Cross-compile binaries by default
* Only distribute one binary - just download and run (no more dependencies!)
* Low memory overhead

I believe the second point is crucial, in that it makes it *dead-simple* to get up and running. Existing solutions (such as SPT or [Phishing Frenzy](https://github.com/pentestgeek/phishing-frenzy)) require an already running webserver or other framework installation.

### Gophish Requirements
Here are a few of the things I want to be able to do with gophish:

* Create "campaigns", each of which are a simulation of a phishing attack
* Create HTML or text templates for phishing emails, providing as many as possible built-in
* Allow users to clone existing sites for use in templates
* Allow users to import and group phishing targets easily
* Provide intuitive analytics, and allow reports to be exported
* If possible, integrate with existing products such as the Social Engineer's Toolkit

### Getting Started
I am *tentatively* planning on creating a REST API for all backend functionality, so as to allow developers to automate gophish. This will also make it easier to separate front-end logic from the backend.

I have looked around at different ```go``` web frameworks, and I am going to start by taking a look at the tools provided in the [Gorilla toolkit](http://www.gorillatoolkit.org/), since they seem to sit on top of the standard ```net/http``` library without abstracting too many things away.

That's all I have for now - stay tuned for progress updates (and a big initial commit)! And, as always, let me know if you have any questions or suggestions in the comments below!

-Jordan