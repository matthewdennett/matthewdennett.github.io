---
layout: post
title: Updating My Site to Use the Material Design Lite
tags:
- Jekyll
- Github Pages
- Material Design
show: true
---

Recently, I stumbled across [Rachel V. Madrigal's blog](https://rachelmad.github.io/) and really liked the design and functionality of their site. In the post [Using Material Design Lite with Jekyll for a simple blog](https://rachelmad.github.io/entries/2016/08/14/material-design-jekyll) they describe how their site has been put together using a template from using the Material Design Lite (MDL).  Impressed with the results, I decided to update my own site to be built from the same template.

<!--more-->

#### MDL Portfolio Template

The MDL Portfolio template is a pre-built website layout designed by Google's Material Design team. It features a clean and modern design with a focus on showcasing your work and skills. The template includes sections for an intro, about me, skills, projects, and a contact form(which I removed). The layout is fully responsive, meaning it should look great on any device, whether it's a desktop, tablet, or mobile phone.

One of the things I love about this template is how easy it is to customize. The source code is well-organized and easy to understand, making it simple to make changes to the design or add your own content. MDL also provides extensive documentation and code samples to help you get started.

#### Updating My Site

Updating my site to use the MDL Portfolio template was a straightforward process. I downloaded the template from the MDL website and then replaced the existing content on my site with the template's HTML, CSS, and JavaScript files. I then customized the template to match the style I wanted and added my own content.

Rather than starting from scratch, I was able to use the template as a starting point and then make changes as needed. This not only saved me time, but it also ensured that my site had a professional and polished look.

#### Small fixes

Rachel's use of clickable chips to display related post, experience and projects is a really nice feature.  Based on their implementation, I have implemented the same for my site with one small but notable improvement for the chips and tags to be case insensitive. Previously a chip “Jinja” wouldn’t match posts tagged with "jinja" and even cause a 404 if I hadn’t set up a tagged page to match both.

There are two changes I made to make this case insensitive.

##### 1. Related Pages
The first change is to ensure that when searching for related posts to display on the tagged page, I ensure that all the tags of the current page are converted to lower case and that the current tagged page name is lower case before checking if the page name is in the list of tags.

I can thank this [Stack Overflow answer](https://stackoverflow.com/questions/51977204/if-statement-with-contains-and-downcase-in-liquid#52008487) for the method of converting the case of an array.

{% highlight html linenos %}
{% raw %}
{% for post in project_posts %}
	{% assign post_tags = post.tags | join: '~~~' | downcase | split: '~~~' %}
	{% assign page_name = page.name | downcase %}
	{% if post_tags contains page_name %}
{% endraw %}
{% endhighlight %}


##### 2.Chips
The second change was along the same line, making sure that the chip was converted to lower case to ensure it would consistently match the tagged page.

{% highlight html linenos %}
{% raw %}
{% assign page_name = include.name | downcase %}
{% case page_name %}
	{% when "github pages" %}
		{% assign tagName="githubpages" %}
...
	{% when "cisco aci" %}
		{% assign tagName="ciscoaci" %}
	{% else %}
		{% assign tagName=page_name %}
<a href="/tagged/{{ tagName | downcase }}">
    <span class="mdl-chip">
        <span class="mdl-chip__text">{{ include.name }}</span>
    </span>
</a>
{% endcase %}
{% endraw %}
{% endhighlight %}