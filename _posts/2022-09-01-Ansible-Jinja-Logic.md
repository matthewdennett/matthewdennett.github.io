---
layout: post
title: Ansible Jinja Logic
tags: Automation Ansible Tips Jinja
---

In the post [Handy Ansible Logic](https://mdennett.id.au/2020/06/01/Handy-Ansible-Logic/) I described a situation where moving ansible logic out of ```when:``` statement and into an inline Jinja achieved a much cleaner solution. Recently, I had a similar situation where I needed to extend what I was doing in the Jinja. This time, I needed to dynamically construct the a variable to pass into a modules parameter.

```yml
{% raw %}
- name: Example 1 - Set facts
    set_fact:
      output: >-
        {%- set output_list = [] -%}
        {%- for server in ip_list -%}
          {%- set my_server = {'name': server.name, 'type': 'IPv4'} -%}
          {{ output_list.append( my_server ) }}
        {%- endfor -%}
        {{ output_list }}
{% endraw %}
```

<!--more-->


The module I was working with needed a parameter set to a list of servers. Each server in the list needed its corresponding type (IPv4 or IPv6) passed in as well. Or to remove all servers previously configured in the list, an empty list was needed to be passed in. So I needed to set the parameter to a variable like either of these variables: 

```yml
{% raw %}
server_list1:
  - name: 10.1.1.10
	type: 'IPv4'
  - name: 2001:db8::10:1:1:10
	type: 'IPv6'

server_list2: [ ]

{% endraw %}
```
That is all pretty straight forward. 


Now, the goal of this play was to be executed by others in the team, and require the minimal amount of variables defined for it to function. Especially, if the server_list was to be empty, they should be able to omit the server_list variable completely and the task still function as expected. We could use the ```| default( [ ] )``` operator and set a empty list if the server_list variable wasn't set. That would work, but that still meant that when a list was passed in we had to define the type for each server in the list. This is where the Jinja logic is really handy.


This is what a cut down version my first iteration of this looked like:

```yml
{% raw %}
---
- name: Ansible Jinja Logic
  gather_facts: false
  hosts: localhost
  tasks:
  
  - name: Example 1A - Server list defined
    vars:
      server_list:
        - name: server1.example.com
        - name: server2.example.com
        - name: server3.example.com
    set_fact:
      output_1a: >-
        {%- set output_list = [] -%}
        {%- if server_list is defined -%}
          {%- for server in server_list -%}
            {%- set my_server = {
                  'name': server.name, 
                  'type': 'IPv4'
                } 
            -%}
            {{ output_list.append( my_server ) }}
          {%- endfor -%}
        {%- endif -%}
        {{ output_list }}

  - name: Print output - Server list defined
    debug: 
      var: output_1a
{% endraw %}
```

In the above example, the inline Jinja is doing a couple things

1. Create an empty list called “output_list” 
2. Iterate over each item in the input variable “server_list”
3. For each item, create the new variable “my_server” and store the name and set it’s type to ‘IPv4’
4. Append "my_server" to “output_list”
4. Return the “output_list” variable.

Here's the output generated when “server_list” is and isn’t defined.

```bash
{% raw %}
user@box:~/$ ansible-playbook example1.yml 
PLAY [Ansible Jinja Logic] ******************************************

TASK [Example 1A - Server list defined] *****************************
ok: [localhost]

TASK [Example 1B - Server list NOT defined] *************************
ok: [localhost]

TASK [Print output - Server list defined] ***************************
ok: [localhost] => {
    "output_1a": [
        {
            "name": "server1.example.com",
            "type": "IPv4"
        },
        {
            "name": "server2.example.com",
            "type": "IPv4"
        },
        {
            "name": "server3.example.com",
            "type": "IPv4"
        }
    ]
}

TASK [Print output - Server list NOT defined] ***********************
ok: [localhost] => {
    "output_1b": []
}

PLAY RECAP **********************************************************
{% endraw %}
```

So using this fairly simple jinja logic we can handle the situation where the server_list is both defined and undefined. We just needed to fix up support IPv6 server type. 

```yml
{% raw %}
---
- name: Ansible Jinja Logic
  gather_facts: false
  hosts: localhost
  vars:
    server_list:
      - name: server1.example.com
      - name: server2.example.com
      - name: server3.example.com
        ipv6: true
  tasks:

  - name: Example 3 - Set facts 
    set_fact:
      output: >-
        {%- set output_list = [] -%}
        {%- if server_list is defined -%}
          {%- for server in server_list -%}
            {%- if server.ipv6 is defined and server.ipv6 == true -%}
              {%- set v4v6 = 'IPv6' -%}
            {%- else -%}
              {%- set v4v6 = 'IPv4' -%}
            {%- endif -%}
            {%- set my_server = {
                  'name': server.name, 
                  'type': v4v6
                } 
            -%}
            {{ output_list.append( my_server ) }}
          {%- endfor -%}
        {%- endif -%}
        {{ output_list }}

  - name: Print Output
    debug: 
      var: output
{% endraw %}
```

Extending the jinja a little further we’ve added a check to see if the input in the “server_list” has specified that the server to be IPv6 or we default back to IPv4. 

Here’s the output of the above example


```bash
{% raw %}
user@box:~/$ ansible-playbook example2.yml 
PLAY [Ansible Jinja Logic] ******************************************

TASK [Example 3 - Set facts] ****************************************
ok: [localhost]

TASK [Print Output] *************************************************
ok: [localhost] => {
    "output": [
        {
            "name": "server1.example.com",
            "type": "IPv4"
        },
        {
            "name": "server2.example.com",
            "type": "IPv4"
        },
        {
            "name": "server3.example.com",
            "type": "IPv6"
        }
    ]
}

PLAY RECAP **********************************************************
{% endraw %}
```

### Note
I’m sure you noticed I've used ```{% raw %} {%- {% endraw %}``` and ```{% raw %} -%} {% endraw %}``` throughout all the inline Jinja instead of the standard ```{% raw %} {% {% endraw %}``` and ```{% raw %} %} {% endraw %}```. This strips any white space that may be getting added into the template, which helps ensure the template will produce valid JSON as output. You can read about this and other ways you can control white spaces in the [Jinja documentation](https://jinja.palletsprojects.com/en/3.1.x/templates/#whitespace-control). 


## The Take Away
What you can do with ansible and jinja is really impressive, both are powerful tools on their own  and when you put them together like this they really can complement each other. 

My only concern of using inline jinja like this especially if its getting more complex is it’s supportability. I haven’t seen it being used heavily in the wild and think that I might be a bit of a foreign concept to some. That being said, the documentation is extensive and jinja templates shouldn't be a new thing if you have even a small amount of experience with ansible, it’s just a slightly different way of using it.

All of the examples shown are available [here](https://github.com/matthewdennett/2022-09-01-Ansible-Jinja-Logic) if you would like a copy. 
