---
layout: post
title: "Gophish Update: Getting Closer to Beta!"
date: 2014-06-21 23:39
comments: true
published: false
categories: 
- gophish
- go
---
<img src="{{root_url}}/images/headers/gophish_purple.png"/>
### Introduction
It's been a busy couple of months! I thought it would be worth providing a much-overdue update into the development status of [gophish](http://github.com/jordan-wright/gophish). Overall, the project is getting closer to beta status every day, and I'd like to tentatively expect an ETA of early August!

Without further ado, let's dive in and see what's changed.

### Design
I moved away from the previous black and white look to a friendler color scheme using the fantastic [Flat-UI](http://designmodo.github.io/Flat-UI/) package from DesignModo.

Let's take a look at some the key features to see the new design in action.

### Features
My goal is to make using gophish both easy and powerful. With this in mind, I've implemented some neat features to make setting up campaigns as simple as possible:

### Campaign Results Dashboard

<img src="{{root_url}}/images/blog/gophish_screenshots/gophish_campaign_results.gif"/>

#### WSYWIG Editing of HTML Templates
You can make pixel perfect templates and customize the content seamlessly.

<img src="{{root_url}}/images/blog/gophish_screenshots/gophish_template.gif"/>

#### Bulk Importing of Users
Adding users to a group manually is a pain. I've implemented bulk inserting that accepts a CSV file, and adds the users automatically.

<img src="{{root_url}}/images/blog/gophish_screenshots/gophish_group.gif"/>

#### Coming Soon
These are features that I am working to implement as soon as possible:

* Landing pages from clicked emails
* Support for email attachments
* Support bulk-inserting groups
* Support for importing emails as a template
* Realtime updates to campaign results dashboard
* Client API libraries (eg Python) 

### Conclusion
Gophish development continues to push forward as much as possible. While I'm balancing time between a few different projets, I hope to get gophish beta out the door as soon as possible. As always, please don't hesitate to let me know if you have any questions or comments! Also, if you use gophish and have any ideas/issues, let me know on [Github](http://github.com/jordan-wright/gophish/issues)!