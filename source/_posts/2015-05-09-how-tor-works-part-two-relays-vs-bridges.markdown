---
layout: post
title: "How Tor Works: Part Two - Relays vs. Bridges"
date: 2015-05-09 15:00
comments: true
published: true
categories:
- tor
---
<img src="{{root_url}}/images/headers/how_tor_works_2.png"/>
### Introduction
Welcome back to my series on how Tor works! In the [last post]({{root_url}}/blog/2015/02/28/how-tor-works-part-one/), we took a look at how Tor operates from a very high level. In this post, we'll dive a bit deeper, taking a look at a potential issue with relays in order to introduce a new concept: **bridges**.
<!--more-->
### Relay Recap
In the last post, we introduced relays as systems designed to move traffic in the Tor network. You'll recall that there were three types of relays:

* **Entry/Guard Relays** - Entry points into the Tor network
* **Middle Relays** - Send traffic from an entry relay to an exit relay
* **Exit Relays** - Send the traffic out of the Tor network to the original destination

Relays are created by volunteers by simply configuring the Tor software to act as a relay. As a reminder, if you have bandwidth to spare, consider setting up a Tor relay!

### The Problem With Relays

When a Tor client starts up, it needs a way to fetch a list of all the entry, middle, and exit relays available. This list of all relays isn't a secret. In fact, in the next post I'll explain in detail how this list is distributed (for a sneak peek check out documents on the *concensus*). While making this list public is necessary, it also presents a problem.

To figure out why this is problem, let's play the role of the attacker and ask ourselves: *What Would an Oppressive Government Do (WWOGD)?*. By thinking to ourselves what a ***real OG*** would do, we can figure out why Tor is built the way it is.

So what would a real OG do? Since censorship is a pretty big deal, and Tor is pretty darn good at getting around it, a real OG would want to block users from using Tor. There are two ways to "block Tor users":

* Block users coming out of Tor
* Block users going into Tor

The first situation is possible, and is up to the discretion of the device (router, etc.) or website owner. All a site owner needs to do is to download the list of Tor exit nodes and block all traffic from those nodes. While this would be unfortunate, there's nothing Tor can really do about it.

However, the second situation is much worse. While blocking incoming Tor users can keep them from a particular site, blocking users from going into Tor will keep them from **every** site, making Tor effectively useless to those under censorship - some of the users who need Tor most. Just using relays, this is possible since real OG's can just download a list of all guard relays and block any traffic to them.

Thankfully, the Tor project thought of exactly this situation and came up with a clever solution around it. Say hello to **bridges**.

### Introducing Bridges
[Bridges](https://www.torproject.org/docs/bridges.html.en) are a clever solution to this problem. At their core, bridges are just unpublished entry relays. Users that are behind censored networks can use bridges as a way to access the Tor network.

So if bridges are unpublished, how do users know where they are? Won't a master list need to be published somewhere? We'll talk more about these master lists of relays and bridges in the next post, but for now the answer is yes - there is a list of bridges maintained by the Tor project.

**This list just isn't made public.**

Instead, the Tor project has created a way for users to receive a small list of bridges so that they can connect to the rest of the Tor network. This project, called [BridgeDB](https://bridges.torproject.org/bridges) gives users the information about a few bridges at a time. This makes sense, since a few bridges should be all any user needs.

By only giving users a few bridges at a time, it is possible to prevent OG's from blocking all possible entry points into the Tor network. Sure, as relays are discovered they can be blocked, but can anyone really discover all the bridges?

### Can Anyone Find Every Bridge?

The list of all bridges is a closely guarded secret. If a real OG were able to gain access to this list, it would be able to completely block users from using Tor. With this being the case, there has been [research](https://blog.torproject.org/blog/research-problems-ten-ways-discover-tor-bridges) done by the Tor project into possible ways people could discover all the bridges.

I'd like to talk very briefly about #2 and #6 on that list, since that's [*exactly*](https://zmap.io/paper.pdf) [what](http://www.cs.uml.edu/~xinwenfu/paper/Bridge.pdf) researchers have done with some significant success. In the first scenario, researchers scanned the entire IPv4 space using a fast port scanner called ZMap looking for Tor bridges and "were able to identify 79–86%"<sup>1</sup> of them. I recommend reading the paper for the really cool technical details (about finding Tor bridges and ZMap in general).

The second scenario is a neat one and introduces an important challenge for Tor (or any network for that matter). It all comes down to a simple concept - *users can't be trusted*. In order to keep the Tor network as anonymous and locked down as possible, the Tor network is designed in such a way to intentionally distrust relay operators. We'll see more examples of this later.

### Next Steps
In this post, we've talked about the need for relays that aren't published in some "master list". But, you'll notice I didn't give many details about *how* this master list is created, or how Tor clients get access to the list.

So, in our next post, we'll take a look at how the Tor network maintains the status of all relays in the network. We'll also introduce some very powerful relays in the Tor network who run the show - **directory authorities**.

As always, please let me know if you have any questions or comments below!

*Update: [Part three - "The Consensus"](/blog/2015/05/14/how-tor-works-part-three-the-consensus/) has been published!*

-Jordan ([@jw_sec](https://twitter.com/jw_sec))

<sup>[1] [https://zmap.io/paper.pdf](https://zmap.io/paper.pdf)</sup>
