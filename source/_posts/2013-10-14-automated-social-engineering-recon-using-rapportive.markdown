---
layout: post
title: "Automated Social Engineering Recon Using Rapportive"
date: 2013-10-14 02:55
comments: true
categories:
- Social Engineering
- Python
- OSINT
---
<img src="{{root_url}}/images/headers/rapportive_small.png"/>

### Introduction
When performing a social engineering engagement, recon is key. In a [previous post](http://raidersec.blogspot.com/2012/12/automated-open-source-intelligence.html), I demonstrated a few ways in which we could automate the recon process. However, the methods I showed were simply ways to find the profiles of people that might belong to a particular organization.

During SE engagements, we often either run across email addresses (by, say, simply scraping the main website) or want to enumerate the email address structure in use by an organization (generating possible alternatives using tools like jigsaw.rb). It would be helpful if it were possible to automate the process of validating those email addresses by associating them to additional information or social networking profiles. This is where Rapportive comes in handy.

<!-- more -->
### What is Rapportive?
[Rapporitve](http://rapportive.com/), acquired by Linkedin in February 2012, is an add-on for multiple browsers which embeds information about your contacts directly into Gmail. The extension sends your contact's email address to the Rapportive server, which responds with known public information associated with the email address such as social networking profiles.

### Reverse Engineering the Rapportive Extension
Let's take a look at how the Rapportive Extension works (disclosure: I'm using Chrome). The extension simply consists of a Javascript file called "user.js". When we navigate to Gmail, this file will load another JS file called ["launchpad"](https://rapportive.com/load/launchpad). The launchpad file then fetches the main file called ["application"](https://rapportive.com/load/application), which implements all the functionality provided by Rapportive. If you want to do a full static analysis of them - feel free. For this post, we'll take a look at the behavior of the extension, and see if we can automate its functionality.

Here are the requests made when using the extension:

<a href="{{root_url}}/images/blog/rapportive/burp.PNG" target="_blank"><img src="{{root_url}}/images/blog/rapportive/burp.PNG"/></a>

There are two interesting requests to note. The first one is a call to /immediate_login. This is used to login to Rapportive using Google's [OpenID](https://developers.google.com/accounts/docs/OpenID). The next request to note is to /contacts/email?[target_email]. This is the call made when we open an email from one of our contacts. Here is an example of the data returned (I've only removed unneccessary data for brevity):

```
{
   "contact":{
      "image_url_raw":"https:\/\/secure.gravatar.com\/avatar\/97754d23d40bbe7dce50f3424991b697?s=80&d=404",
      "raplets":[],
      "memberships":[
         {
            "profile_url":"http:\/\/www.linkedin.com\/pub\/54\/795\/752",
            "view_text":"",
            "site_name":"LinkedIn",
         },
         {
            "display_name":"Blogger",
            "username":null,
            "profile_url":"http:\/\/raidersec.blogspot.com",
            "view_text":"View Jordan's profile on Blogger",
            "site_name":"Blogger",
         },
         {
            "display_name":"GitHub",
            "username":"jordan-wright",
            "profile_id":"jordan-wright",
            "profile_url":"https:\/\/github.com\/jordan-wright",
            "view_text":"View Jordan's profile on GitHub",
            "site_name":"GitHub",
         }
      ],
      "headline":"Jordan Wright - Security Engineer",
      "email":"jmwright798@gmail.com",
      "location":"Texas Area",
      "occupations":[
         {
            "job_title":"Security Engineer"
         }
      ],
      "name":"Jordan Wright",
   },
   "success":"image_or_occupation_or_useful_membership",
}
```

We can see that quite a bit of data was returned by this query. As social engineers, not only would we know that this is likely a valid email address (and therefore a valid email address format for other potential targets of the same organization), but we could also see the social networking profiles linked to this email address. This could reveal other infrmation which may be useful to us later.

So now we want to see how we can:

*	Automate the login process (*if needed*)
*	Deconstruct the /contact/email query to find any special parameters that we need to pass

Let's start by taking a look at the parameters passed to the /contact/email request, and see what kind of information is needed:

<a href="{{root_url}}/images/blog/rapportive/contact_email_headers.PNG" target="_blank"><img src="{{root_url}}/images/blog/rapportive/contact_email_headers.PNG"/></a>

We can immediately see an interesting header being sent with the request titled "X-Session-Token". It turns out that **the entire authentication of the request relies on sending this token**. We can test this using an API console like [this one](https://apigee.com/console/others).

Now we just need to figure out where to get our session token. There was one request that Burp didn't catch that is made once we first log into Gmail. It is a request to /login_status?[email_address]. The response to this query looks like this:

```
{
   "user_full_name":" ",
   "status":200,
   "authenticated_as":null,
   "active_auths":[],
   "claimed_emails":null,
   "plan":{
      "key":"free",
      "price":0,
      "name":"Free"
   },
   "session_token":"BAgiX09ON0&lt;snip&lt;ece684be8",
   "user_preferences":{&lt;snip&lt;},
   "authenticity_token":"7KNIosqb&lt;snip&gt;csaXzbQFOFi7jg=",
}
```

This request will return a session token for the currently logged in user. However, here's the trick: ***any email will work***. Seriously - [try it](http://rapportive.com/login_status?user_email=this_doesnt_exist_@gmail.com). Even without being logged in to Rapportive (ie, by using a random email address), we can still receive a session token and can still make valid queries on email addresses like the one seen originally. So - we don't need to worry about automating the authentication. 

Let's take a look at how we can at least automate the process of getting the session token and querying for an email address.

### Automating Rapportive Queries
Now that we know how Rapportive works, we can automate the process to perform lookups for users of our choosing. Let's start by getting the session token for a particular email address:

```
import requests
random_email = 'asdfqwer@gmail.com'
response = requests.get('https://rapportive.com/login_status?user_email=' + random_email).json()
print response['session_token']
# output - BAgiX0IvUmJHMUQ3Z2pOUWhmM0ZKRX...
```

Easy as that! Now that we have the session token, we can make a request for a given target email address as follows:

```
profile = requests.get('https://profiles.rapportive.com/contacts/email/' + target_email, 
            headers = {'X-Session-Token' : response['session_token']}).json()
```

The profile returned will now be in JSON format where we can parse out any information such as social networking profile links, usernames, etc. I won't bore you with the implementation of all the parsing - so let's move onto the final script!

### Introducing rapportive.py
I've created a simple Python script called [rapportive.py](https://github.com/jordan-wright/rapportive) to do this work for us. It takes in a list of email addresses from STDIN or a single email via the -e argument and prints out any results found. This can be useful if we want to pass in emails from other tools such as [jigsaw.rb](https://github.com/pentestgeek/jigsaw/blob/master/jigsaw.rb). It can also log to an output file, if needed.

Here is an example of how to use it:

```
~# cat emails.txt | ./rapportive.py
10-25 23:37 rapportive   INFO     Using owcdf@gmail.com
10-25 23:37 rapportive   INFO     Found match for jmwright798@gmail.com

Name: Jordan Wright
Position: Security  Engineer
Company: CoNetrix
        LinkedIn - http://www.linkedin.com/pub/54/795/752
        Blogger - http://raidersec.blogspot.com
        GitHub - https://github.com/jordan-wright
```
```
~# ./rapportive.py -e jmwright798@gmail.com
10-25 23:31 rapportive   INFO     Using ntmai@gmail.com
10-25 23:31 rapportive   INFO     Found match for jmwright798@gmail.com
Name: Jordan Wright
Position: Security  Engineer
Company: CoNetrix
        LinkedIn - http://www.linkedin.com/pub/54/795/752
        Blogger - http://raidersec.blogspot.com
        GitHub - https://github.com/jordan-wright

```

You can find it [Github](https://github.com/jordan-wright/rapportive).

Enjoy!

Jordan