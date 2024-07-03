---
layout: post
title: Python Link Checker
tags:
- python
show: true
---

With the update for my site to use “chips” to link to related pages its very easy for me tag a post with a tag that doesn't have the supporting “tagged” page and resulting in a broken link.  With the number of chips and tags I have across this site manually verifying each one manually is far to inefficient. Using [LinkChecker](https://pypi.org/project/LinkChecker/) makes this process quick and easy.


<!--more-->

#### Install
<p>
The process to install install <a href="https://pypi.org/project/LinkChecker/">LinkChecker</a> is the same as any{% include chip.html name="Python" %}package. I tend to always work in a python virtual environment as shown in my example below but you don't have to.
</p>
1. First create a Python virtual environment

{% highlight bash linenos %}
user@box:~/linkchecker$ python3 -m venv venv
user@box:~/linkchecker$ source venv/bin/activate
{% endhighlight %}

2. Install the linkchecker package with pip

{% highlight bash linenos %}
(venv) user@box:~/linkchecker$ pip install linkchecker
Collecting linkchecker
  Downloading LinkChecker-10.2.1-py3-none-any.whl (286 kB)
     |████████████████████████████████| 286 kB 6.6 MB/s
Collecting beautifulsoup4>=4.8.1
  Downloading beautifulsoup4-4.12.2-py3-none-any.whl (142 kB)
     |████████████████████████████████| 142 kB 14.2 MB/s
Collecting dnspython>=2.0
  Downloading dnspython-2.3.0-py3-none-any.whl (283 kB)
     |████████████████████████████████| 283 kB 14.8 MB/s
Collecting requests>=2.20
  Downloading requests-2.31.0-py3-none-any.whl (62 kB)
     |████████████████████████████████| 62 kB 1.1 MB/s
Collecting soupsieve>1.2
  Downloading soupsieve-2.4.1-py3-none-any.whl (36 kB)
Collecting charset-normalizer<4,>=2
  Using cached charset_normalizer-3.1.0-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (195 kB)
Collecting idna<4,>=2.5
  Using cached idna-3.4-py3-none-any.whl (61 kB)
Collecting certifi>=2017.4.17
  Downloading certifi-2023.5.7-py3-none-any.whl (156 kB)
     |████████████████████████████████| 156 kB 14.8 MB/s
Collecting urllib3<3,>=1.21.1
  Downloading urllib3-2.0.2-py3-none-any.whl (123 kB)
     |████████████████████████████████| 123 kB 15.9 MB/s
Installing collected packages: soupsieve, beautifulsoup4, dnspython, charset-normalizer, idna, certifi, urllib3, requests, linkchecker
Successfully installed beautifulsoup4-4.12.2 certifi-2023.5.7 charset-normalizer-3.1.0 dnspython-2.3.0 idna-3.4 linkchecker-10.2.1 requests-2.31.0 soupsieve-2.4.1 urllib3-2.0.2
(venv) user@box:~/linkchecker$
{% endhighlight %}

If you choose not to use a virtual environment you will most likely need to update your ```$PATH``` to include the instillation directory.

#### Usage

This package couldn't be much easier to use. Like shown in the example below you simply need to pass in the URL of the page you want to check. By default this will recursively check all link for the single domain passed in.

{% highlight bash linenos %}
(venv) user@box:~/linkchecker$ linkchecker http://127.0.0.1:4000
INFO linkcheck.cmdline 2023-05-25 20:47:32,237 MainThread Checking intern URLs only; use --check-extern to check extern URLs.
LinkChecker 10.2.1
Copyright (C) 2000-2016 Bastian Kleineidam, 2010-2022 LinkChecker Authors
LinkChecker comes with ABSOLUTELY NO WARRANTY!
This is free software, and you are welcome to redistribute it under
certain conditions. Look at the file `LICENSE` within this distribution.
Read the documentation at https://linkchecker.github.io/linkchecker/
Write comments and bugs to https://github.com/linkchecker/linkchecker/issues

Start checking at 2023-05-25 20:47:32+010
10 threads active,    24 links queued,   21 links in  56 URLs checked, runtime 1 seconds
10 threads active,    35 links queued,   46 links in 106 URLs checked, runtime 6 seconds
10 threads active,    20 links queued,   61 links in 117 URLs checked, runtime 11 seconds
10 threads active,     6 links queued,   77 links in 125 URLs checked, runtime 16 seconds
 3 threads active,     0 links queued,  102 links in 143 URLs checked, runtime 21 seconds

Statistics:
Downloaded: 504.68KB.
Content types: 12 image, 53 text, 0 video, 0 audio, 0 application, 0 mail and 40 other.
URL lengths: min=19, max=110, avg=47.

Thats it. 105 links in 143 URLs checked. 0 warnings found. 0 errors found.
Stopped checking at 2023-05-25 20:47:55+010 (23 seconds)
{% endhighlight %}


The default behavior also restricts checks to the domain that you pass in. So you will commonly want to use the ```--check-extern``` flag verify that any external links you have.


#### Error Output

Here is an example of the output produced when a broken link is identified. You can easily see the page `Parrent URL` and even the line number of the broken link.

{% highlight bash linenos %}
URL        `https://mdennett.id.au/2020/06/01/Handy-Ansible-Logic/'
Name       `Handy Ansible Logic'
Parent URL http://127.0.0.1:4000/blog/, line 422, col 28
Real URL   https://mdennett.id.au/2020/06/01/Handy-Ansible-Logic/
Check time 3.138 seconds
Size       1KB
Result     Error: 404 Not Found
10 threads active,     4 links queued,   86 links in 138 URLs checked, runtime 21 seconds
{% endhighlight %}

There are a number of helpful output formats such as text, CSV, HTML. Check out the ```--help``` information for a full list.

#### Excludes
<p>
It's quite possible that you have a broken link that you want to exclude from the result which can be achieved with the ```--ignore_url``` flag. If needed you can list the flag multiple times. Just pass in a {% include chip.html name="Python" %} REGEX to match the links to be ignored.
</p>
{% highlight bash linenos %}
(venv) user@box:~/linkchecker$ linkchecker http://127.0.0.1:4000 --ignore-url medium.com
{% endhighlight %}


There are number of online tool that provide a very similar result. However, the down side of this is that your site needs to be published and available online, which might not always be possible. In no particular order here are a couple that I found.
1. [https://www.drlinkcheck.com/](https://www.drlinkcheck.com/)
2. [https://www.deadlinkchecker.com/](https://www.deadlinkchecker.com/)
3. [https://www.brokenlinkcheck.com/#](https://www.brokenlinkcheck.com/#)

I can see this being really handy next time I have to implement url/uri rewrites on a load balancer. Automatically being able to test and report what rewrites are working and not will save me plenty of time and definitely help prevent me missing a something.