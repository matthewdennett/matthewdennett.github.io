---
layout: post
title: Python HTTP Server
tags:
- Python
show: true
---

I have been using the http.server python module for some time but recently, it's been more handy than ever. Whether it's testing network controls, Load Balancer configurations or simply copying a file from one location to another. Whichever the reason, the Python http.server module is so simple to use you can't afford not to know.

<!--more-->

The python module 'http.server' lets you create a really simple Web server, with python being almost ubiquitous and this module being installed by default you will very rarely be without it. You can use the module within your code or directly from the CLI (example below).

By default, when you run the module from the CLI it will start a web server listening on port 8000. If a index.html in the current directory, it will be served as the default page, if not a directory listing will be served.

{% highlight bash linenos %}
user@host :~/tmp$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
127.0.0.1 - - [16/Nov/2022 11:25:08] "GET / HTTP/1.1" 200 -
{% endhighlight %}


The modules default settings work perfect for about 90% of the situation where I'm using this module. Here are some of the situations where I have used the 'https.server' module in the last week:

- Validating load balancer configuration is functioning correctly by mimicking the backend web server.
- Host Cisco router firmware to be downloaded to remote devices before a software upgrade.
- Copying files from one server to another, where SCP or a remote file share wasn't available.
- Sharing files from my local machine to colleagues without having to hop through a file share.
- Copying files out from the WSL session on my local machine to another location. Maybe this is just being a tad lazy, I could definately just browse down to the WSL install location and copy the flies out that way too.


#### Module Options

You can find the [module documentation here](https://docs.python.org/3/library/http.server.html), but here's a quick rundown of the options that I commonly use.

##### Port Number

The listening port can be set by passing the desired port in as a positional parameter to the module:

{% highlight bash linenos %}
user@host :~/tmp$ python3 -m http.server 1234
Serving HTTP on 0.0.0.0 port 1234 (http://0.0.0.0:1234/) ...
{% endhighlight %}


##### Web Directory

The web root can be specified with ```-d``` or ```--directory```

{% highlight bash linenos %}
user@host :~/tmp$ python3 -m http.server --directory /tmp/www/
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
{% endhighlight %}


##### IP Binding

Use the ```-b``` or ```--bind``` to bind to a specific IP that exists on the local machine:

{% highlight bash linenos %}
user@host :~/tmp$ python3 -m http.server --bind 192.168.10.10
Serving HTTP on 192.168.10.10 port 8000 (http://192.168.10.10:8000/) ...
{% endhighlight %}


#### SSL/TLS

Unfortunately, the module doesn't support SSL directly, but this can be easily achieved with a small script. [Sergi Alfonso](https://medium.com/@SergiAlfonso) has a really good post [Python Simple HTTP Server With SSL Certificate](https://medium.com/@SergiAlfonso/python-simple-http-server-with-ssl-certificate-encrypted-traffic-9c5cbe1fd750) that I have often referred to when I need a to enable SSL. This post is definately worth a read as he highlights how this can be used to hide the transmission of malware or other nefarious content.

