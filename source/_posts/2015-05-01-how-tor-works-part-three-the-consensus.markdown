---
layout: post
title: "How Tor Works Part Three - The Consensus"
date: 2015-05-14 18:30
comments: true
published: true
categories:
- tor
- consensus
---
<img src="{{root_url}}/images/headers/how_tor_works_2.png"/>
### Introduction
Welcome to the third post in my series on how Tor works! In the [past two posts](/blog/categories/tor/), we talked about how clients tunnel traffic through relays, as well as introduced the idea of unpublished relays called bridges.

But how do clients know what relays are active? How is the Tor network actually organized and maintained? This post will answer this question by talking about a living document called the **consensus** as well as introducing a few very important Tor nodes that run the show behind the scenes.
<!--more-->
### Respect My Authoritah
In the last post, we mentioned that there is a master list of Tor relays as well as a master list of Tor bridges. Before talking about *how* this list is maintained, we need to talk about *who* maintains it.

[Hardcoded into each Tor client](https://gitweb.torproject.org/tor.git/tree/src/or/config.c#n824) is the information about 10 beefy Tor nodes run by trusted volunteers. These nodes have a very special role - to maintain the status of the entire Tor network. These nodes are known as **directory authorities** (DA's).

I've [written a bit](/blog/2014/12/19/what-happens-if-tor-directory-authorities-are-seized/) about DA's in the past. Distributed around the world, DA's are in charge of distributing an ever-updated master list of all known Tor relays. They are the gatekeepers that choose what relays are valid, and when.

So why 10? We know it's usually a bad idea to have an even number of things voting on something, since there could be ties. You'll recall that in the previous post I mentioned that there is a master list of relays *and* a master list of bridges. This is where the split happens. 9 of the DA's maintain the master list of relays, while one DA (Tonga) maintains the list of bridges.

Here are the DA's:

<img src="/images/blog/how_tor_works/authorities.png"/>
### Reaching a Consensus
So there are DA's and they maintain the status of the Tor network. But, **how**?

The status of all the Tor relays is maintained in a living document called the **consensus**. DA's maintain this document and update it every hour by a vote. Here's a basic flow of how this updating process works.

* Each DA compiles a list of all known relays
* Each DA then computes the other needed data, such as relay flags, bandwidth weights, and more
* The DA then submits this data as a "status-vote" to all the other authorities
* Each DA next will go get any other votes it is missing from the other authorities
* All the parameters, relay information, etc. from each vote are combined or computed and then **signed** by each DA
* This signature is then posted to the other DA's
* There should be a majority of the DA's that agree on the data, validating the new consensus
* The consensus is then published by each DA

You'll notice that I mention each DA publishes this consensus. This is done over HTTP such that anyone can download the latest copy at ```http://directory_authority/tor/status-vote/current/consensus/```. You can see for yourself by downloading the most current consensus from ```tor26``` [here](http://86.59.21.38/tor/status-vote/current/consensus/).

So we see a consensus... but what does it mean?

### Anatomy of a Consensus
The consensus document is a bit tough to get a handle on right away just by reading [the spec](https://gitweb.torproject.org/torspec.git/tree/dir-spec.txt). I find it helps to have things broken down visually to get an idea of how things are structured.

To help with this, I made a poster in the fantastic [corkami](https://github.com/corkami/) style. Here's a dissection of a snipped down version of the Tor consensus document (click for full resolution!):

<a href="/images/blog/how_tor_works/consensus.png"><img src="/images/blog/how_tor_works/consensus_small.png"/></a>

### Next Steps
The consensus is a powerful document. By having trusted authorities keeping a master list of relays and their capabilities, it is easy for new and existing clients to keep track of the addition and removal of Tor relays.

Now, you'll notice that we haven't really covered exit relays. These relays hold a very important position in the Tor network, and deserve their own discussion. So, in the next post we'll talk about how exit relays work, and what happens when exit relay operators decide to "break bad", or wreak havoc on Tor users.

Until then, let me know if you have any questions or comments below!

-Jordan ([@jw_sec](https://twitter.com/jw_sec))
