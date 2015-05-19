---
layout: post
title: "Tracking Bad Guys and Botnets Using ELK"
date: 2014-11-30 21:17
comments: true
published: false
categories:
- elk
- elasticsearch
- osint
---

<img src="{{root_url}}/images/headers/tracking_elk.png"/>
### Introduction
If you've followed this blog, you know that I've talked about how the [ELK stack](http://www.elasticsearch.org/) has helped me keep track of large amounts of data. Logstash (the "L" in E***L***K) is a key player in this process that allows us to take in data from multiple sources, manipulate it in meaningful ways, and send it off to Elasticsearch (***E***LK) for indexing.

Today, I want to show how two useful Logstash "input" filters can help us track threat actor Twitter accounts and IRC-based botnets: specifically the [```twitter```](http://logstash.net/docs/1.4.2/inputs/twitter) and [```irc```](http://logstash.net/docs/1.4.2/inputs/irc) filters.
<!--more-->
### A Brief Intro to Logstash
Before jumping into the configuration files used for this, it is worth talking a little about how Logstash works. Logstash is simply a tool to help manage logs. It operates by taking in logs from one or more sources, filtering them and enriching them with meaningful data, and then outputting them somewhere. More specifically, this consists of:

<span class="fa-stack fa-lg fa-1x">
<i class="fa fa-circle fa-stack-2x" style="color:#74b73f"></i>
<i class="fa fa-stack-1x" style="color:white; font-family:inherit;">1</i>
</span>
<span style="color:#74b73f; font-weight:bold;">Input</span> - Logstash can receive logs from numerous sources such as files (e.g. Apache's ```access_log```), syslogs, or even - as is the case in this post - messages from Twitter's streaming API or IRC channels.

<span class="fa-stack fa-lg fa-1x">
<i class="fa fa-circle fa-stack-2x" style="color:#74b73f"></i>
<i class="fa fa-stack-1x" style="color:white; font-family:inherit;">2</i>
</span>
<span style="color:#74b73f; font-weight:bold;">Filters</span> - As raw logs are received, they are filtered and enriched using one or more of Logstash's many filters. These include operations such as grep'ing for certain patters, performing reverse DNS lookups on IP's and hostnames, or even anonymizing sensitive fields.

<span class="fa-stack fa-lg fa-1x">
<i class="fa fa-circle fa-stack-2x" style="color:#74b73f"></i>
<i class="fa fa-stack-1x" style="color:white; font-family:inherit;">3</i>
</span>
<span style="color:#74b73f; font-weight:bold;">Output</span> - The newly filtered and enriched logs now need to be sent somewhere. Logstash comes with plugins to ship these logs to raw files, directly to TCP or UDP sockets, or indexing engines such as Elasticsearch.

All these together look like the following:

<img src="{{root_url}}/images/blog/elk/logstash_flow.png"/>

### Setting up the Logstash Config
I'll leave [installing the ELK stack](http://www.elasticsearch.org/overview/elkdownloads/) as an exercise for the reader, so let's jump right into the configuration file used to get events from Twitter and IRC. Let's take a look at each scenario individually, and then combine the pieces into a fully working configuration later.

#### Tracking Bad Guys on Twitter
The first scenario we'll take a look at is how to track the activity of multiple users on Twitter using the [```twitter```](http://logstash.net/docs/1.4.2/inputs/twitter) input method. Behind the scenes, Logstash is connecting to Twitter's [streaming API](https://dev.twitter.com/streaming/public) to continuously filter through public statuses.

> Note: The current major release of logstash doesn't support the ```followers``` parameter supported by the Twitter API. I submitted a pull request to add this feature.

Getting a list of known malicious actor Twitter accounts is also left as an exercise to the readers (I may follow-up with a post on how to do this later). For the sake of this post, we'll track the tweets from [@dumpmon](http://twitter.com/dumpmon) (who you should follow!). However, let's setup logstash to monitor our list of accounts, and dump any tweets and events into elasticsearch.

Here's the (commented) config:

```
input {
  twitter {
    # Setup Twitter authentication
    consumer_key => redacted
    consumer_secret => redacted
    oauth_token => redacted
    oauth_secret => redacted
    # Whether or not we want the full tweet
    full_tweet => true
    # A comma-separated list of accounts to follow
    # max: 500
    followers => dumpmon, raidersec
  }
}
```

Let's see what this looks like in Kibana.

#### Tracking IRC-based Botnets
The next scenario we'll take a look at is tracking IRC-based botnets. IRC-based bots are still very common, and are easy to track since they use a standard protocol.

Connecting to the botnet generally involves just connecting to the IRC channel in use. We can do this by simply setting the ```irc``` logstash input.


```
input {
  irc {
    channels =>
    host =>
    nick =>
    password =>
    real =>
    secure => true
    user =>
  }
}
```

#### Final Configuration

We can put our previous config pieces together to make a full logstash configuration, which you can find [here](gist).

### Conclusion
While the ELK stack started as a way to manage logs, it can be leveraged into an invaluable tool in the infosec toolkit. As always, please don't hesitate to let me know if you have any questions or comments below.

-Jordan ([@jw_sec](http://twitter.com/jw_sec))
