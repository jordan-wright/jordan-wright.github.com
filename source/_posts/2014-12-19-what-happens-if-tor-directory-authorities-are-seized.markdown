---
layout: post
title: "What Happens if Tor Directory Authorities are Seized?"
date: 2014-12-19 18:43
comments: true
published: true
categories:
- tor
---
<img src="{{root_url}}/images/headers/tor_seized.png"/>
### Introduction
The Tor Project [has announced](https://blog.torproject.org/blog/possible-upcoming-attempts-disable-tor-network) that they have received threats about possible upcoming attempts to disable the Tor network through the seizure of Directory Authority (DA) servers. While we don't know the legitimacy behind these threats, it's worth looking at the role DA's play in the Tor network, showing what effects their seizure could have on the Tor network.*
<!--more-->
### What are Directory Authorities?
Simply put, think of the DA servers as the trusted providers of a phonebook. This phonebook - called the *consensus* - contains the complete information about each known Tor relay, and is updated every hour. When it's time to update the list, a majority of the directory authorities must agree on the accuracy of the new list by cryptographically signing the proposed consensus. Once this process is complete, clients are able to download the updated list of relays.

There are currently 10 DA's whose information is [hardcoded into Tor clients](https://gitweb.torproject.org/tor.git/tree/src/or/config.c#n824) - one of which (Tonga) is used for bridge access. This means that, to keep the network updated and stable, **5 DA's must still be operational**. If a seizure attempt is able to take down 5 or more DA's, the network will enter an unstable state, and the integrity of any updates to the consensus cannot be guaranteed.

### Where are the DA's Located?
The seizure of 5 or more DA's would be a large feat, but it is absolutely possible. As one commenter on HN [mentioned](https://news.ycombinator.com/item?id=8775028), it would only take a joint effort by the US and Germany to take down 5 DA servers. Another [comment](https://news.ycombinator.com/item?id=8775009) provides the geolocation and organization of each DA.

### The Aftermath
An attack seizing the DA servers would severely cripple the Tor network. The Tor Project would not only need to replace the DA servers, but would then need to introduce a client update with the new DA information. During this time, the integrity of the consensus could not be trusted, and it would be [increasingly difficult](https://blog.torproject.org/blog/possible-upcoming-attempts-disable-tor-network#comment-83762) for new clients to be introduced into the Tor network.

### This Doesn't Solve the Problem
It's important to note that severing the Tor network doesn't solve any problem. Tor provides an invaluable escape from censorship, and the means to having privacy from otherwise prying eyes. I'm confident that the Tor Project will be resilient in recovering from any attempted takedown attempts.

### More Information
Further information and detailed status (obtained from the updated consensus) about each of the Tor Directory Authorities can be found at the following links:

* [Faravahar - 154.35.32.5:443](https://globe.torproject.org/#/relay/CF6D0AAFB385BE71B8E111FC5CFF4B47923733BC)
* [urras - 208.83.223.34:80](https://globe.torproject.org/#/relay/0AD3FA884D18F89EEA2D89C019379E0E7FD94417)
* [dannenberg - 193.23.244.244:443](https://globe.torproject.org/#/relay/7BE683E65D48141321C5ED92F075C55364AC7123)
* [Tonga (**bridge DA**) - 82.94.251.203:443](https://globe.torproject.org/#/relay/4A0CCD2DDC7995083D73F5D667100C8A5831F16D)
* [moria1 - 128.31.0.34:9101](https://globe.torproject.org/#/relay/9695DFC35FFEB861329B9F1AB04C46397020CE31)
* [maatuska - 171.25.193.9:80](https://globe.torproject.org/#/relay/BD6A829255CB08E66FBE7D3748363586E46B3810)
* [gabelmoo - 131.188.40.189:443](https://globe.torproject.org/#/relay/F2044413DAC2E02E3D6BCF4735A19BCA1DE97281)
* [dizum - 194.109.206.212:443](https://globe.torproject.org/#/relay/7EA6EAD6FD83083C538F44038BBFA077587DD755)
* [longclaw - 199.254.238.52:443](https://globe.torproject.org/#/relay/74A910646BCEEFBCD2E874FC1DC997430F968145)
* [tor26 - 86.59.21.38:443](https://globe.torproject.org/#/relay/847B1F850344D7876491A54892F904934E4EB85D)

Information regarding any updates to this situation can be found on the [Tor Project blog](https://blog.torproject.org/blog/).

As always, let me know if you have any questions or comments below!

-Jordan ([@jw_sec](http://twitter.com/jw_sec))

**It is worth noting that I am by no means a Tor expert, and am relying on knowledge gained from [previous]({{root_url}}/blog/2014/10/06/creating-tor-hidden-services-with-python/) [research](http://raidersec.blogspot.com/2013/09/mapping-tor-relays-and-exit-nodes.html) into the Tor network structure.*
