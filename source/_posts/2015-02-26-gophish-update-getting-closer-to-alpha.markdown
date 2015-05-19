---
layout: post
title: "Gophish Update: Getting Closer to Alpha!"
date: 2015-02-26 20:50
comments: true
published: true
categories:
- gophish
- go
---
<img src="{{root_url}}/images/headers/gophish_purple.png"/>
### Introduction
It's been a busy couple of months!

I thought it would be worth providing a long-overdue update into the development status of [gophish](http://github.com/jordan-wright/gophish). Overall, the project is getting closer to beta status every day, and I'm hoping to see a [0.1 release](https://github.com/jordan-wright/gophish/milestones) at the end of March.

Without further ado, let's dive in and see where we're at.
<!--more-->
### Design
I moved away from the previous black and white look to a friendlier color scheme using the fantastic [Flat-UI](http://designmodo.github.io/Flat-UI/) package from DesignModo.

### Features
My goal is to make using gophish both easy-to-use and powerful. With this in mind, I've implemented some neat features to make setting up awesome campaigns as simple as possible.

Let's take a look at a few:

#### WYSIWYG Editing of HTML Templates
You can make pixel perfect email and landing page templates and customize the content seamlessly. While I started with some ```contenteditable``` hacks, I couldn't find a reliable way to allow full page rendering (without opening up security issues or design incompatibility problems). So, I looked around and found that [CKEditor](http://ckeditor.com) which takes care of this and more!

Now we can switch between a raw HTML source view and a fully rendered view in just one-click. Want to see what a template would look like full-screen? Also no problem!

<img src="{{root_url}}/images/blog/gophish_screenshots/gophish_template.gif"/>

#### Email Attachments
Sending emails with links in them is great, but if we can't add "malicious" attachments to emails we send, then we miss a *huge* attack tactic deployed against our users. I'm excited to report that attaching files to email templates is just a matter of selecting the "Add Files" button on the email template modal and choosing the file you want to attach. Easy as that!

#### Bulk Importing of Users
Adding users to a group manually is a *pain*. So, I've implemented bulk inserting that accepts a CSV file, and adds the users automatically. Right now, it allows for the following fields:

* First Name
* Last Name
* Email
* Position

<img src="{{root_url}}/images/blog/gophish_screenshots/gophish_group.gif"/>

#### Campaign Results Dashboard
Executing campaigns isn't helpful if you don't have a way to see the results. This is why I've been working on a campaign results dashboard that will serve as a one-stop-shop for viewing campaign results. The plan is to start by having information about the campaign itself, such as clicks over time and overall success rate. However, I want to expand this to also include information about the user, such as browser plugin information, demographics and location, etc.

<img src="{{root_url}}/images/blog/gophish_screenshots/gophish_campaign_results.gif"/>

#### Full API Support
From the get-go, gophish was **designed for automation**. Setting up campaigns, importing users, getting results can all be done manually through the Web UI. However, all the UI does is call out to the API. For darn-near *everything*.

To take a look at the API documentation, just load up gophish and head over to ```/api```!

#### Coming Soon
While I'm proud of all that's been accomplished with gophish so far, it's not even close to being done.

Here are just a few of the *many* features coming down the pipeline to get excited about:

* A more fleshed out Campaign Results dashboard (<span><i class="fa fa-heart-o"></i></span> metrics)
* Ability to schedule campaigns in advance
* Email tracking - know when an email is opened!
* The ability to clone a landing page template with one click
* Support for importing emails from the "Source" of an existing email (or hopefully even an IMAP service directly!)
* "Teams" support to share and coordinate phishing campaigns
* Realtime updates to campaign results dashboard
* Client API libraries (eg Python)

### Conclusion
Gophish development continues to push forward as much as possible. While I'm balancing time between a few different projets, I hope to get gophish alpha out the door as soon as possible. As always, please don't hesitate to let me know if you have any questions or comments! Also, if you use gophish and have any ideas/issues, let me know on [Github](http://github.com/jordan-wright/gophish/issues)!
