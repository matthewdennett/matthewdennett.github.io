---
layout: post
title: Ansible Jinja Logic
tags: Automation Ansible Tips Jinja
---

In the post [Handy Ansible Logic](https://mdennett.id.au/2020/06/01/Handy-Ansible-Logic/) I described a situation where I move ansible logic out of ```when:``` statement and into some inline Jinja to achieve a much cleaner solution. Recently, I had a similar situation but needed to extend what I was doing in the Jinja. This time I needed to dynamically construct the input variable I needed to pass into a module.  
  
<!--more-->

# The Problem



```yml
{% raw %}
---

{% endraw %}
```

Heres the output of the above play.
```bash
{% raw %}

{% endraw %}
```


```bash
{% raw %}

{% endraw %}
```



# The Solution


```yaml
{% raw %}
---

{% endraw %}
```



```bash 
{% raw %}

{% endraw %}
```


```bash 
{% raw %}

{% endraw %}
```

We can still use jinja's filters like any other variable with this approach too. 

```yaml
{% raw %}

{% endraw %}
```

All of the examples shown are available [here]() if you would like a copy. 

