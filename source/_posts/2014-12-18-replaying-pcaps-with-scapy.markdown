---
layout: post
title: "Replaying PCAP's with Scapy"
date: 2014-12-18 17:22
comments: true
published: false
categories:
---
<img src="{{root_url}}/images/headers/scapy_replay.png"/>

### Introduction
Replaying PCAP traffic is useful for network troubleshooting, exploit development, and more. Fantastic tools such as [tcprelay](http://tcpreplay.synfin.net/wiki/tcpreplay) exist that make this process easy. This blog post will show how to use the fantastic ```scapy``` Python library to create our own PCAP replaying utility.
<!--more-->
### Starting Simple
I've written about ```scapy``` before, and for good reason. It's powerful API makes packet creation and manipulation easy. Let's start by reading in a PCAP file called ```traffic.pcap```:

```
from scapy import scapy
packets = scapy.rdpcap("traffic.pcap")
```
