---
layout: post
title: "What InfoSec Learned in 2014"
date: 2014-12-28 20:38
comments: true
published: true
categories:
- infosec
---
<img src="{{root_url}}/images/headers/what_infosec_2014.png"/>

*Busy year.*

It seems as though each year brings more and more events that throw our industry into the forefront of attention - and they're never good. At the same time, it's key to remember that these events allow us to learn and evolve as an industry. Let's take a look at some of the key things we as an infosec industry can learn from 2014:
<!--more-->
<img class="section_header" src="{{root_url}}/images/blog/infosec_2014/heartbleed.png"/>

### No Software is Safe, or Why Your Vulnerability Doesn't Need a Logo
Heartbleed and Shellshock were arguably two of the most critical vulnerabilities our industry has seen in a while. These vulnerabilities provided a stark reminder that - as the security community often quips - no software is safe. The importance and widespread use of both OpenSSL and Bash contributed to a large, immediate impact when the vulnerabilities were announced.

This impact was also largely fueled by the media. Now, many people would take the stark stance that assigning names and logos to vulnerabilities is ridiculous, and I mostly agree. However, it's important to at least consider the benefits this approach had. Too often, vulnerabilities aren't patched because they simply aren't kept up with. It's mind numbing trying to keep up with every CVE-*xxxx* affecting every piece of software. In these cases, since the vulnerabilities were critical enough, this approach of marketing to the media worked really well - many systems were patched quickly<sup>1</sup>.

But sadly, we can already see this being overused. It seems as though anyone who thinks they have found a critical vulnerability needs to find some kind of catchy name to put with it. And I get it - CVE-2014-0160 isn't as dangerous sounding as ***Heartbleed***, and CVE-2014-6271 + related isn't as hacker-scary sounding as ***Shellshock***. Add logos to go with the catchy names? Marketing people **love** it.

I have no doubt that 2015 will bring with it more critical vulnerabilities in well-known, established pieces of software. I only hope that we can use the lessons-learned from Heartbleed and Shellshock to mature as an industry and approach the marketing and distribution of these details in a responsible, controlled way. Not every vulnerability needs a full marketing team behind it.

<img class="section_header" src="{{root_url}}/images/blog/infosec_2014/router.png"/>

### Embedded Devices: Hacking Like it's 1995
While the (in)security of embedded devices such as SOHO routers isn't particularly new, there were a substantial number of critical vulnerabilities recently released for these devices. These vulnerabilities includes [blatant](https://github.com/elvanderb/TCP-32764) [backdoors](http://www.devttys0.com/2013/10/reverse-engineering-a-d-link-backdoor/), web-app security 101 [blunders](http://mis.fortunecook.ie/), or just bad software.

The primary cause for concern with these vulnerabilities is that they affect a *ton* of devices. For example, the recently reported ["Misfortune Cookie"](http://mis.fortunecook.ie/) vulnerability - complete with a name and logo - claims to affect "12 million ... unique devices". Unfortunately, security appears to be put on the backburner when it comes to the development of embedded devices, likely due to either infosec ignorance or simply because manufacturers don't think attackers will target their systems - I am a fan of the latter.

I wholeheartedly expect to see a wider emphasis put on attacking embedded devices - particularly network devices - in 2015. The fact that in 2012 over 400,000 devices can be [commandeered using default passwords](http://internetcensus2012.bitbucket.org/paper.html) alone should be considered a substantial threat. Sadly, it will only get worse before it gets better. The only way to fix the situation will be to continue finding and publishing vulnerabilities in these devices in the hopes that manufacturers will start placing a higher emphasis on device security.

Here's hoping for the best.

<img class="section_header" src="{{root_url}}/images/blog/infosec_2014/pos.png"/>

### POS Software = POS
Compromising Point-of-Sale (POS) devices became the new-hotness for cybercriminals this year. Notable breaches of retailers such as Target and Home Depot caused massive re-issuing of bank credit/debit cards, hefty financial losses, and resulted in some nice [class action complaint readings](http://cdn.arstechnica.net/wp-content/uploads/2014/12/document4.pdf).

The key thing to note about these breaches is that POS malware such as [Alina](http://blog.spiderlabs.com/2013/05/alina-shedding-some-light-on-this-malware-family.html), [JackPOS](http://blog.spiderlabs.com/2014/02/jackpos-the-house-always-wins.html), etc. **is not sophisticated**. It's not. Generally, it works like this:

* Check if a POS software process is running
* Dump the memory of that process
* Grep for 16 numbers in a row
* Validate it's a valid CC using something like Luhn's algorithm
* Send it off to a server
* Repeat

*Groundbreaking.*

So why did this work so well? Turns out, these POS systems (like embedded devices) have not gone through through the gamut of security scrutiny that standard software has. This makes sense, since the market-share for this software was much smaller (meaning less installs of the software than, say, Java or Flash).

But the criminals figured something out. Maybe market share isn't as important as we thought it was. Maybe there is low-hanging fruit that can give enough yield when compromised that it's worth looking into. A [recent report](https://www.fox-it.com/en/files/2014/12/Anunak_APT-against-financial-institutions2.pdf) by security firm Fox-IT summed it up nicely:

> [Hackers] can steal $2000 a thousand times, and earn $2 million, but also they can steal it in one time and immediately get it with much less effort...

Don't get me wrong - consumer malware families like Zeus will still have a place in the industry while there is money to be made in selling harvested information. However, I have no doubt that Target and Home Depot will only be the beginning of these retailer breaches, at least until chip-and-pin cards are the industry default.

### Conclusion
It's clear that our industry is showing no signs of slowing down (and hey, I <span><i class="fa fa-heart-o"></i> job security</span>). However, we need to push forward into the next year with the mindset that *we're still losing*. This next year will bring more breaches, more vulnerabilities, and more lessons to be learned.

As always, let me know if you have any questions or comments below.

[1] [https://jhalderm.com/pub/papers/heartbleed-imc14.pdf](https://jhalderm.com/pub/papers/heartbleed-imc14.pdf)

-Jordan ([@jw_sec](http://twitter.com/jw_sec))
