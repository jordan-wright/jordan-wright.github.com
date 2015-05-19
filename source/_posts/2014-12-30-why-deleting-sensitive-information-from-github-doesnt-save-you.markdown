---
layout: post
title: "Why Deleting Sensitive Information from Github Doesn't Save You"
date: 2014-12-30 19:19
comments: true
published: true
categories:
- github
---
<img src="{{root_url}}/images/headers/harvest_github.png"/>

So you accidentally committed a password or API key to Github. ***Ouch.***

"No problem!", you think, "I'll just follow Github's helpful information on [how to delete sensitive information](https://help.github.com/articles/remove-sensitive-data/) and I'll be fine!"

Just today, I saw a [great article](http://www.devfactor.net/2014/12/30/2375-amazon-mistake/) detailing one developer's experience with committing sensitive information to Github. Unfortunately, this article missed the main point. In this post, I'm going to show exactly how hackers *instantly* harvest information committed to public Github repositories, and why deleting this information doesn't solve the problem.
<!--more-->
### Drinking from Github's Firehose
Github has an extensive API. One of the most useful endpoints is located at [```/events```](https://developer.github.com/v3/activity/events/). This endpoint basically provides a firehose of *all* public events as they happen. This includes account creation, code commits, and more. Just "star" a repository? It was published at ```/events```.

Hackers can use this endpoint to watch for any and all code commits. Once a commit is found, they can instantly make a request to the [```repos/:owner/:repo/git/commits/:sha1```](https://developer.github.com/v3/git/commits/) endpoint to get the details for that commit. Part of these details is a reference to the project snapshot, or ```tree```, for that commit (the files themselves). This tree has it's own SHA1, that can be used in the [```repos/:owner/:repo/git/tree/:sha1?recursive=1```](https://developer.github.com/v3/git/trees/) endpoint to get a full listing of files in the repository.

Each of these listings has a reference to the file's actual content for the commit. A request can be made to the [```/repos/:owner/:repo/git/blobs/:sha1```](https://developer.github.com/v3/git/blobs/) endpoint to get this file content, which will include the sensitive information!

These 4 requests are made in a matter of seconds, and can be sped up by caching the SHA1 of files to determine if the file has been changed. It's a good thing that no one has made a system that's been caching all this data this entire time, right?

### Say Hello to GHTorrent
[GHTorrent](http://ghtorrent.org/) advertises itself as an "offline mirror of data". In a nutshell, it keeps track of *a ton* of data that flows through Github's Events API stream, and recursively resolves dependencies to relate, say, a commit object to an event object. Currently, they suggest they have accumulated the data from 2012-2014.

This database has incredible potential for researchers, but also allows for hackers to pull previously deleted or changed data en masse. Granted, from what I can tell they don't store the actual file content (so your accidentally committed password won't be stored), but that doesn't mean that there isn't sensitive data to be had.

Consider the email address used to create a Github account, or commit a code change. Both of these actions created an event that was harvested by GHTorrent. Here's an example showing the details of a particularly [talented developer](https://github.com/jordan-wright):

<img src="{{root_url}}/images/blog/harvest_github/db.png"/>

Currently, it looks like there are about 4.7 million accounts cached in GHTorrent, with over 3.4 million having a non-null email address. That's a lot of email addresses.

GHTorrent is just an example. While it doesn't appear to store all content, it would be trivial for hackers to reproduce the project with the added feature of searching commits for sensitive information as the events are generated. This searching can be done using keywords such as "password", "key", etc.

### The Only Way to be Safe
Hopefully it's clear that deleting sensitive information from Github doesn't solve the problem. The **only** way to protect your assets after committing sensitive information is to consider the information compromised and to change the password/API key/whatever. Then, make sure to avoid committing this data in the future!

Be smart - protect your data.

-Jordan ([@jw_sec](https://twitter.com/jw_sec))
