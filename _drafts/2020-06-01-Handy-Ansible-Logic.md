---
layout: post
title: Handy Ansible Logic
tags: Automation Ansible Tips
---

Recently I was writing what initially looked like a pretty straight forward ansible play. Everything was progressing quite well until I came to a point where I needed to set a a variable only when, another variable was set. My first pass at this logic was to repeat the task, and control which task was run with a  ```when:``` statement. This worked, but just wasn't nice with all of the duplicated code. I just knew there was a better way and there is.   

<!--more-->

