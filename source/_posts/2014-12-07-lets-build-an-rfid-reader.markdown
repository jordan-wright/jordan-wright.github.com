---
layout: post
title: "Let's Build an RFID Reader"
date: 2014-12-07 17:30
comments: true
published: false
categories:
- hardware
- rfid
---
<img src={{root_url}}/images/headers/rfid.png/>
### Introduction

Before getting started, I should make something clear: I ~~suck at making hardware~~ am better at software than hardware.

That being said, RFID is everywhere these days. Credit cards, badges, door keys all contain RFID chips to enable for easy, touchless transfer of data. The problem with this is that RFID is *inherently insecure*. The way RFID works by design (as will be shown in detail later) is that anyone with a reader can get the data from an RFID-tagged device without the device owner knowing.

In this post, we'll explore the process of making an RFID reader from scratch to read some sweet, sweet RFID data. Let's get to it.
<!--more-->
### RFID 101
The security of RFID-enabled devices is nothing new. In fact, it's a standard part of many pentests (if the target uses RFID). There have been some great talks on the subject of building RFID readers in the past, so this material won't be anything groundbreaking.

RFID is a reader/writer model of transferring data.
