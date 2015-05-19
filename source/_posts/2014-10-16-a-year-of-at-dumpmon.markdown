---
layout: post
title: "A Year of @dumpmon"
date: 2014-10-16 23:13
comments: true
published: false
categories:
- dumpmon
- elk
---
<img src="{{root_url}}/images/headers/dumpmon_header.png"/>
### Introduction
This post is long overdue.

Back in May 2013, I [released](http://raidersec.blogspot.com/2013/03/introducing-dumpmon-twitter-bot-that.html) a Twitter bot called [@dumpmon](http://twitter.com/dumpmon) whose sole purpose was to track and report password dumps and other sensitive information shared on pasting sites such as Pastebin. Since that time, dumpmon has proven - to my excitement - to be valuable to researchers, being featured in [news articles](ars), [Defcon](pic of defcon), and [HIBP](http://haveibeenpwned.com)!

After well over a year, I figured it was time to post an overdue status update providing some insight into the data @dumpmon has collected over this time, as well as sharing some lessons learned in the process.

*Note: This is a pretty long post, so feel free to skip here if you just want to see the numbers.*
<!--more-->

### Collecting the Data: Mongo + ELK
Soon after @dumpmon went live, I noticed that sensitive pastes were getting deleting shortly after being posted. This makes it difficult to use the data for any long term research, so I started considering ways to maintain long-term copies.

First, I used MongoDB to store each paste text and meta information (such as ```type```, ```num_emails```, ```num_hashes```, etc.). This worked well for about a year, but it put quite a bit of strain on my Raspberry Pi. Additionally, it was cumbersome to query the data and visualize the statistics over time.

I decided to move everything over to an ELK instance. Unfortunately, in the process of doing so, the DB got corrupted causing *most* of the data to be lost. *Not a good day.*

However, eventually I got everything up and running with copies of the data going to both MongoDB (just in case), as well as ELK. Here's a partial screenshot of the dashboard:

<img src="{{root_url}}/images/blog/dumpmon_report/dashboard.png"/>

Having this dashboard lets me see trends over time, search for individual paste content, as well as track pastes related to a campaign over time.

### Analyzing the Data
Enough history - let's take a look at the data!

As mentioned before, most of the older data was lost in the process of migrating to ELK, but we still have data dating back to June to analyze.

Let's first talk about the overall kinds of data kinds of data found. Currently, @dumpmon looks for database dumps, cisco configuration files, and google API keys (based on my blog post here). Overwhemingly, database dumps are the most common type of paste found, coming in at 94%.

> **Database dumps account for 94% of the sensitive pastes found.**

Each of these database dumps can contain a combination of usernames, email addresses, and passwords (plain text or hashed).

#### Which Email Provider is the Most Common?
In case you were wondering which email provider shows up the most in database dumps, here you go!

#### Password Leaks are Common

After a few weeks of running the bot, is was clear that password dumps occur far more often than I thought. On average, dumpmon found x significant (by dumpmon's thresholds) database dumps every day. Here are some numbers by the month:



> **Approximately x significant database dumps are posted to Pastebin every day**

<table>
<thead>
<th>Category</th>
<th>Value</th>
</thead>
<tbody>
<tr>
<td>Test</td>
<td>645234</td>
</tr>
</tbody>
</table>

#### Cumulative Results
Here are a few more cumulative numbers for those interested:

X database dumps total
X number of emails found
X number of hashes found

### Lessons Learned
#### Hackers are Copy-Cats
One of the first things I noticed is that many malicious actors are copy cats. Once a password leak is posted, dumpmon usually quickly comes across the same dump posted with a different attribution.

#### False Positives are Hard to Remove
Almost immediately after launching the bot, I noticed I was getting a *ton* of false positives. As it turns out, the regex for things like social security numbers tends to match quite a few unexpected things. However, by far the largest number of false positives I get are from crash dumps that contain a ton of hashes. I eventually moved to having a "banlist", which looks for key strings to identify crash dumps. It's not perfect, but it helps.

### Twitter Account Statistics
Thanks to the fantastic Twitter Analytics platform, we can get insight into the statistics regarding the actual @dumpmon Twitter account.

<img src="{{root_url}}/images/blog/dumpmon_report/twitter_followers.png"/>

As you can see above, the number of followers has steadily increased over time, which is exciting!

Additionally, the links dumpmon posts are being seen, and people are responding.

### Conclusion
I hope this post gave a long-overdue update of where @dumpmon is today. I have recently moved dumpmon to a new home on a Digital Ocean VPS that's bigger, faster, and has more storage than my rasPi. Hopefully, this will eliminate freeze-ups to due to trying to parse large pastes - things look promising so far!

It should be noted that @dumpmon is just getting started. While there haven't been too many changes to the codebase in a while, I have plans to continue improving and refining the bot, as well as integrating with other services such as HIBP.

All in all, @dumpmon is here to stay.


As always, please don't hesitate to let me know if you have any questions or comments!

-Jordan ([@jw_sec](http://twitter.com/jw_sec))
