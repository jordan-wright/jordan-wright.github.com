---
layout: post
title: "Reverse Engineering the We Heart It API"
date: 2014-10-12 20:40
comments: true
published: true
categories:
- weheartit
- reverse engineering
- dynamic analysis
- static analysis
---
<img src="{{root_url}}/images/headers/weheartit_api.png"/>
### Introduction
A while back, I came across the article from the The Washington Post describing We Heart It, a social network claiming [over 30 million users](http://www.washingtonpost.com/news/the-intersect/wp/2014/05/07/30-million-people-use-this-social-network-and-youve-probably-never-heard-of-it/). If you haven't seen it, [We Heart It](http://weheartit.com) (from here on out abbreviated as WHI) is a social network which encourages people to post and share photos and images of things that inspire them.

Having such a large user-base, I was interested in seeing what kind of API the site offered developers. However, I was disappointed when I found out that the API was [closed to "partners"](https://weheartit.com/partners), and even this is not a full REST API, but rather a simple button developers can place on their website to allow users to interact with WHI.

With this being the case, I decided to take a look at the Android and iPhone apps using both static and dynamic analysis in an experiment to see if I could reverse engineer the API used on the backend. Here are the results.
<!--more-->
### Dynamic Analysis

Dynamic analysis is generally a bit quicker and can help give a good overview into how the application functions. By [intercepting traffic using a proxy]({{root_url}}/blog/2013/11/07/how-to-pentest-iphone-apps-with-burp/), we can easily analyze the structure of requests/responses in use by the application.

#### Registration & Authentication

Loading up the WHI iPhone app and attempting to register an account shows the following request/response:

```
POST /api/users HTTP/1.1
Host: api.weheartit.com

<snip>
client_id=redacted
&client_secret=redacted
&user[email]=redacted
&user[name]=redacted
&user[password]=redacted
&user[username]=redacted

{
    "result":"success",
    "object":
    {   "id":27215547,
            "username":"jordan_test2",
            "name":"Jordan Test",
            "location":"",
            "bio":"",
            <snip>
            "public_account":true,
            "verified":false,
            "created_at":"2014-10-12T18:38:35Z",
            "email":"redacted",
            "cover_image": { <snip> }
    }
}
```

*Note: I originally contacted the WHI security team since the login page (and every other page) was served via HTTP. This has since been fixed.*

Registering an account automatically logs the user in. This launches the OAuth2 authentication flow to generate an access token which is used in an ```Authorization: Bearer``` header for all API calls:

```
POST /oauth/token HTTP/1.1
Host: api.weheartit.com

<snip>
client_id=redacted
&client_secret=redacted
&grant_type=password
&password=redacted
&username=redacted

{
    "access_token":"redacted",
    "token_type":"bearer",
    "expires_in":31104000,
    "refresh_token":"redacted",
    "scope":"public"
}
```

This token is then used to make API calls to the WeHeartIt backend using a RESTful JSON API. Here are a couple of examples of this (some response details have been removed for brevity):

```
GET /api/v2/entries/85847648 HTTP/1.1
Host: api.weheartit.com
Authorization: Bearer [redacted]

{
    "id": 85847648,
    "title": "Colors",
    <snip>
    "hearts_count": 11285,
    "created_at": "2013-11-10T17:09:49Z",
    "hearted": false,
    "via_hearts_count": 9919,
    "tags": [<snip>],
    "creator": {
        "id": 6043210,
        "username": "Natasja4205",
        <snip>
    },
    "user": {
        "id": 6043210,
        "username": "Natasja4205",
        <snip>
    }
}
```

```
GET /api/v2/users/27215547 HTTP/1.1
Host: api.weheartit.com
Authorization: Bearer [redacted]

{
    "id": 27215547,
    "username": "jordan_test2",
    "name": "Jordan Test",
    "avatar": [<snip>],
    "public_account": true,
    "verified": false,
    "location": null,
    "bio": null,
    "link": null,
    "hearts_count": 0,
    "following_count": 1,
    "followers_count": 2,
    "sets_count": 0,
    "created_at": "2014-10-12T18:38:35Z",
    "cover": {<snip>}
}
```

Great - we have an API! We could likely automate this app crawling to enumerate the available API calls and parameters. However, to get a more comprehensive idea as to what exact API endpoints are available, let's go straight to the source.

### Static Analysis

Now that we have a feel of how the application operates, let's verify our findings by [decompiling the Android app]({{root_url}}/blog/2014/08/10/decompiling-android-apps-the-easy-way/) and doing some static analysis.

#### App Structure
The app has quite a few dependencies, but the core package (```com.weheartit```) is fairly straight-forward. The API we are concerned with is located at ```com.weheartit.api```. Searching for the string "/api", we can find that the bulk of the API functionality is found in APIRequest.java and APIRequestv2.java

#### APIRequest.java
APIRequest.java essentially contains the schema for version 1 of the OAuth enabled RESTful API. For example, we can find the OAuth ```client_id``` and ```client_secret``` parameters used by the Android app to get an access token:

```java
private void a(Map map, LoginServices loginservices, ApiResponseCallback apiresponsecallback)
    {
        HashMap hashmap = new HashMap();
        hashmap.put("client_id", "25ieooqr");
        hashmap.put("client_secret", "zlype4airg41b33uwafbe8a6p8bwcgiw");
        hashmap.put("grant_type", "password");
        hashmap.putAll(map);
        a.b("oauth/token", hashmap, new OAuthDataResponseHandler(apiresponsecallback, LoginServices.a(loginservices)));
    }
```

This function is used to obtain an access token using the appropriate LoginServices call. After we have an access token associated with a ```User``` object, this access token is used in subsequent API requests. We can enumerate through the functions in APIRequest.java to find each of the valid API calls, as well as the arguments to each call. Here are a couple of examples (more comprehensive documentation below):

```java
public void a(UserSettings usersettings, ApiResponseCallback apiresponsecallback)
    {
        User user = WhiSession.b();
        String s = user.getAccessToken();
        WhiLog.d("ApiRequest", (new StringBuilder()).append("updateUserSettings() for user (").append(user.getId()).append(")").toString());
        HashMap hashmap = new HashMap(12);
        hashmap.put("access_token", s);
        hashmap.put("user[name]", usersettings.getFullName());
        hashmap.put("user[username]", usersettings.getUsername());
        hashmap.put("user[email]", usersettings.getEmail());
        hashmap.put("user[bio]", usersettings.getBio());
        hashmap.put("user[location]", usersettings.getLocation());
        hashmap.put("user[link]", usersettings.getLink());
        hashmap.put("user[show_unsafe_content]", Boolean.toString(usersettings.isUnsafeContentEnabled()));
        <snip: other settings>
        hashmap.put("privacy_options[public]", Boolean.toString(usersettings.isUserPublic()));
        hashmap.put("privacy_options[findable]", Boolean.toString(usersettings.isUserFindable()));
        a.c("api/settings", hashmap, new ApiOperationResponseHandler(apiresponsecallback, "Failed to update user settings"));
    }
```

```java
public void a(String s, long l, Long long1, ApiResponseCallback apiresponsecallback)
    {
        Object aobj[] = new Object[3];
        aobj[0] = s;
        aobj[1] = Long.valueOf(l);
        aobj[2] = long1;
        WhiLog.a("ApiRequest", String.format("getEntryCollectionDetails() with %s, %s, %s", aobj));
        HashMap hashmap = new HashMap();
        hashmap.put("access_token", s);
        if (long1 != null)
        {
            hashmap.put("heart_id", long1.toString());
        }
        a.a((new StringBuilder()).append("api/entry_sets/").append(l).toString(), hashmap, new EntryListResponseHandler(apiresponsecallback, c));
    }
```

#### APIRequestv2.java
It appears as though the backend API is being upgraded to a new version, as evidenced by the existence of APIRequestv2.java. Additionally, there are some endpoints that are located at ```/api/v2/*```. Here are a couple of examples:

```java
public void a(String s, long l, int i, ApiResponseCallback apiresponsecallback)
    {
        WhiLog.a("ApiRequestV2", (new StringBuilder()).append("getUserEntryCollections() with accessToken ").append(s).append(", ").append(l).append(", ").append(i).toString());
        HashMap hashmap = new HashMap();
        hashmap.put("access_token", s);
        hashmap.put("page", String.valueOf(i));
        e.a((new StringBuilder()).append("api/v2/users/").append(l).append("/collections").toString(), hashmap, new EntryCollectionListV2ResponseHandler(apiresponsecallback, a));
    }
```

```java
public void a(String s, Long long1, ApiPagedResponseCallback apipagedresponsecallback)
    {
        WhiLog.a("ApiRequestV2", String.format("getUserDashboard() with %s, %d", new Object[] {
            s, long1
        }));
        HashMap hashmap = new HashMap();
        hashmap.put("access_token", s);
        if (long1 != null)
        {
            hashmap.put("before", String.valueOf(long1));
        }
        e.a("api/v2/user/dashboard", hashmap, new UserDashboardResponseHandler(apipagedresponsecallback, b));
    }
```

### API Documentation
I have created a repository, [weheartit-api](http://github.com/jordan-wright/weheartit-api), containing the full details found from this short study. The information in the repo is by no means comprehensive (consider it a work in progress), but it will hopefully shed some light into the available API functionality.

### WHI Sitemaps

While on the subject of WHI, there are a couple of other interesting facts worth mentioning. For example, if your goal is to get general information about users on the network, there is no need to use the API! In fact, to help Google crawl the network, WHI publishes the following information in its ```sitemaps``` files:

*  Username
*  Link to Profile
*  Date Modified
*  List of Collections

You can find these files here:

* [http://sitemaps.weheartit.com/collections.xml](http://sitemaps.weheartit.com/collections.xml)
* [http://sitemaps.weheartit.com/entry_groups.xml](http://sitemaps.weheartit.com/entry_groups.xml)
* [http://sitemaps.weheartit.com/entries.xml](http://sitemaps.weheartit.com/entries.xml)
* [http://sitemaps.weheartit.com/popular_images.xml](http://sitemaps.weheartit.com/popular_images.xml)
* [http://sitemaps.weheartit.com/users.xml](http://sitemaps.weheartit.com/users.xml)
* [http://sitemaps.weheartit.com/tags.xml](http://sitemaps.weheartit.com/tags.xml)

### Conclusion
For not being exposed to developers, the API supported by WHI is surprisingly robust and complete. While this effort was simply a reverse engineering project out of curiosity, I hope that showing what the API is capable of will encourage WHI devs to open the official API to everyone.

As always, let me know if you have any questions or comments!

-Jordan ([@jw_sec](http://twitter.com/jw_sec))
