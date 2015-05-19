---
layout: post
title: "Creating Tor Hidden Services with Python"
date: 2014-10-06 20:00
comments: true
published: true
categories:
- tor
- python
- hidden services
---
<img src="{{root_url}}/images/headers/hidden_services.png"/>
### Introduction
Tor is often used to protect the anonymity of someone who is trying to connect to a service. However, it is also possible to use Tor to protect the anonymity of a service provider via [***hidden services***](https://www.torproject.org/docs/hidden-services.html.en). These services, operating under the ```.onion``` TLD, allow publishers to anonymously create and host content viewable only by other Tor users.

The Tor project has [instructions](https://www.torproject.org/docs/tor-hidden-service.html.en) on how to create hidden services, but this can be a manual and arduous process if you want to setup multiple services. This post will show how we can use the fantastic ```stem``` Python library to automatically create and host a Tor hidden service.
<!--more-->
### Creating Hidden Services Manually
The [instructions provided by the Tor project](https://www.torproject.org/docs/tor-hidden-service.html.en#two) show that creating hidden services simply involves setting up the service locally (such as a web server listening on localhost), and then setting a few configuration options to make the service available via Tor.

There are two configuration settings necessary to setup a hidden service: ```HiddenServiceDir```, the directory to store the ```hostname``` and ```private_key``` files, and ```HiddenServicePort```, the ports used to proxy hidden service connections.

As the instructions show, each hidden service requires a variation of the following two lines to be present in the ```torrc``` configuration file (setting the directory, host, and ports appropriately):

```
HiddenServiceDir /path/to/store/hidden_service/
HiddenServicePort 80 127.0.0.1:5000
```

### A Bit About the Tor Control Protocol
Changing the configuration file and restarting Tor everytime a change is needed can be a pain. Fortunately, Tor provides a way to dynamically change the running configuration using a simple text based protocol (similar to Telnet) called the Tor Control Protocol.

The [full specification](https://gitweb.torproject.org/torspec.git?a=blob_plain;hb=HEAD;f=control-spec.txt) of the protocol is available, however here is a quick example of getting the valid authentication methods:

```
$ telnet localhost 9151
PROTOCOLINFO

250-PROTOCOLINFO 1
250-AUTH METHODS=COOKIE,SAFECOOKIE,HASHEDPASSWORD COOKIEFILE="Tor\\control_auth_cookie"
250-VERSION Tor="0.2.4.24"
250 OK
```

Other examples using this extensive protocol can be found [here](https://www.thesprawl.org/research/tor-control-protocol/) or in the full protocol spec.

### Introducing Stem
To make interacting with the Tor control port both easier and programmatic, the Tor project maintains a fantastic Python library called [Stem](https://stem.torproject.org/).

#### ```stem.Controller```
Interaction with the Tor control port is performed using the ```stem.Controller``` class. Creating an instance of the class involves connecting to the port and authenticating as follows:

```
from stem.control import Controller
controller = Controller.from_port(address="127.0.0.1", port=9151)
try:
    controller.authenticate(password="")
except Exception as e:
    print e
```

Now that we have a Controller, we can access the local configuration, pull the current descriptors for relays, and more.

Let's use the Controller to automatically set the configuration settings we saw in the previous section. When set, these configuration options will cause Tor to create the two files, ```hostname``` and ```private_key```, necessary to run the hidden service. Here is a short script that will setup a hidden service to listen on TCP port 80 and proxy all requests to an (already established) web server listening on http://127.0.0.1:5000:

```
host = "127.0.0.1"
port = 5000
hidden_svc_dir = "/tmp/hidden_service/"
controller.set_options([
    ("HiddenServiceDir", hidden_svc_dir),
    ("HiddenServicePort", "80 %s:%s" % (host, str(port)))
])
svc_name = open(hidden_svc_dir + "/hostname", "r").read().strip()
print "Created host: %s" % svc_name
```

Easy as that! Now that we have the configuration setup, our service should be ready to go.

### An Example Service
Now that we've seen a little about how Stem works, here's an extremely basic example showing how the hidden service can be setup to work with a Flask application:

```
from stem.control import Controller
from flask import Flask

if __name__ == "__main__":

    app = Flask("example")
    port = 5000
    host = "127.0.0.1"
    hidden_svc_dir = "c:/temp/"

    @app.route('/')
    def index():
        return "<h1>Tor works!</h1>"
    print " * Getting controller"
    controller = Controller.from_port(address="127.0.0.1", port=9151)
    try:
        controller.authenticate(password="")
        controller.set_options([
            ("HiddenServiceDir", hidden_svc_dir),
            ("HiddenServicePort", "80 %s:%s" % (host, str(port)))
            ])
        svc_name = open(hidden_svc_dir + "/hostname", "r").read().strip()
        print " * Created host: %s" % svc_name
    except Exception as e:
        print e
    app.run()
```

Here's what this looks like in action:
```
C:\>python tor_example.py
 * Getting controller
 * Created host: 4yrbax6gwnemqh7n.onion
 * Running on http://127.0.0.1:5000/
```

<img src="{{root_url}}/images/blog/hidden_services/screenshot.png"/>

### Caveats
It is important to note that the security of the hidden service depends on protecting the location of the server. To do this, consider ways to prevent leaking the real server IP through debug messages, etc. There has been some [great discussion](https://news.ycombinator.com/item?id=8404511) on the topic that might be worth looking into.

### Conclusion
Hidden services deliver freedom of speech and the free exchange of ideas without censorship. By using Stem Python library, it's possible to take the pain out of manual configuration and instead programmatically create and manage multiple hidden services.

As always, let me know if you have any questions or comments.

-Jordan ([@jw_sec](http://twitter.com/jw_sec))
