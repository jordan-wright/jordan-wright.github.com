---
layout: post
title: "2 Years of @dumpmon"
date: 2015-05-26 9:00
comments: true
published: true
categories:
- dumpmon
- elk
---
<img src="{{root_url}}/images/headers/dumpmon_header.png"/>
### Introduction
This post is long overdue.

Back in May 2013, I [released](http://raidersec.blogspot.com/2013/03/introducing-dumpmon-twitter-bot-that.html) a Twitter bot called [@dumpmon](http://twitter.com/dumpmon) whose sole purpose was to track and report password dumps and other sensitive information shared on paste sites such as Pastebin. Since that time, dumpmon has proven - to my excitement - to be valuable to researchers, being featured in [news articles](http://arstechnica.com/security/2013/06/raspberry-pi-bot-tracks-hacker-posts-to-vacuum-up-passwords-and-more/), Defcon slides, and [HIBP](http://haveibeenpwned.com)!

After two years, it's time to post an overdue status update providing some insight into the data dumpmon has collected over this time.

*Note: This is a pretty long post, so feel free to skip <a href="#data">here</a> if you just want the data.*
<!--more-->

### Collecting the Data: Mongo + ELK
Soon after dumpmon went live, I noticed that sensitive pastes were getting deleting shortly after being posted. This makes it difficult to use the data for any long term research, so I started keeping copies.

First, I used MongoDB to store each paste and meta information (such as ```type```, ```num_emails```, ```num_hashes```, etc.). This worked well for about a year, but it put quite a bit of strain on my Raspberry Pi. Additionally, it was cumbersome to query the data and visualize the statistics over time.

I decided to move everything over to ELK. Unfortunately, in the process of doing so, the DB got corrupted causing most of the data to be lost. *Not a good day.*

However, I eventually got everything up and running with copies of the data going to both MongoDB (just in case), as well as ELK. Here's a partial screenshot of the dashboard:

<img src="{{root_url}}/images/blog/dumpmon_report/dashboard.png"/>

Having this dashboard in ELK lets me see trends over time, search for individual paste content, as well as track pastes related to a campaign over time.

### Analyzing the Data
Enough history - let's take a look at the data!

As mentioned before, most of the older data was lost in the process of migrating to ELK, but we still have data dating back to June 2014 to analyze.

Let's first talk about the overall kinds of data kinds of data found. Currently, dumpmon looks for database dumps, Cisco config files, Google API keys, and SSH private keys. Overwhemingly, database dumps are the most common type of paste found, coming in at 92%.

Each of these database dumps can contain a combination of usernames, email addresses, passwords (plain text or hashed), and more.

After a few weeks of running the bot, is was clear that password dumps occur far more often than I thought. On average, **dumpmon found 46 significant (by its thresholds) database dumps every day.**

#### Cumulative Results
Here are a few more cumulative numbers for those interested:

* 32.8k potential database dumps total
* 540 emails, on average, per dump (since June)
* 5 million unique emails found (since June)
* 1 million unique hashes found (since June)

### Twitter Account Statistics
Thanks to the Twitter Analytics platform, we can get insight into the statistics regarding the actual dumpmon Twitter account.

<img src="{{root_url}}/images/blog/dumpmon_report/twitter_followers.png"/>

As you can see above, the number of followers has steadily increased over time. While Twitter won't give me trends for the entire time dumpmon has been running, here is some data for the last 28 days:

<img src="{{root_url}}/images/blog/dumpmon_report/general_stats.png"/>

<div id="data"></div>
### Here, Have Some Data
I'm a big fan of giving out raw data. While I have hesitations to release all the raw data dumpmon has found ([contact](/contact) me if you're a researcher who needs this data), I'm happy to give out all of the information I have about dumpmon's Twitter activity. This includes dumpmon's entire tweet archive, aggregated analytics data, and more.

<img style="vertical-align:middle;" src="{{root_url}}/images/blog/dumpmon_report/zip.png"/>
**Tweet Archive**

[This]({{root_url}}/downloads/dumpmon_archive.zip) is dumpmon's exported Twitter archive. It contains **every tweet** sent out by dumpmon. You should use this data if you want to extract out metrics regarding number of emails per dump, etc.

Specifically, the ```tweets.csv``` file in this archive contains the following fields:

```
tweet_id, in_reply_to_status_id, in_reply_to_user_id, timestamp, source, text, retweeted_status_id, retweeted_status_user_id, retweeted_status_timestamp, expanded_urls
```

[Download]({{root_url}}/downloads/dumpmon_archive.zip)

<img style="vertical-align:middle;" src="{{root_url}}/images/blog/dumpmon_report/analytics.png"/>
**Tweet Analytics**

[This file]({{root_url}}/downloads/tweet_activity_metrics_dumpmon.csv) contains the analytics of every tweet after 04/30/2014 (22k tweets). It's in CSV format and has the following fields:

```
Tweet id, Tweet permalink, Tweet text, time, impressions, engagements, engagement rate, retweets, replies, favorites, user profile clicks, url clicks, hashtag clicks, detail expands, permalink clicks
```

[Download]({{root_url}}/downloads/tweet_activity_metrics_dumpmon.csv)

### Conclusion
I hope this post gave a long-overdue update of where dumpmon is today. I recently moved dumpmon to a new home on a Digital Ocean VPS that's bigger, faster, and has more storage than my Pi.

It should be noted that dumpmon is just getting started. While there haven't been too many changes to the codebase in a while, I've seen people expanding the codebase in ways I'm excited to bring mainline.

**All in all, @dumpmon is here to stay.**

As always, please don't hesitate to let me know if you have any questions or comments!

-Jordan ([@jw_sec](http://twitter.com/jw_sec))
