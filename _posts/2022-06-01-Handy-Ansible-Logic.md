---
layout: post
title: Handy Ansible Logic
tags: Automation Ansible Tips Jinja
---

Recently I was writing what initially looked like a pretty straight forward ansible play. Everything was progressing quite well until I came to a point where I needed to set one variable, only when another variable was set. My first pass at this logic was to repeat the task, and control which task was run with a  ```when:``` statement. This worked, but just wasn't ideal with all of the duplicated code. I just knew there had to be a better way and there was.   

<!--more-->

# The Problem

In this situation I needed to set the module parameter ```parm_b``` to the value of a variable ```var_two``` when ```var_one``` was set to a certain value, but in all other cases parameter ```parm_b``` needed to be set to ```var_three```. 

Here's a cut down version of my first approach: 

```yml
{% raw %}
---
- name: Handy Ansible Logic
  gather_facts: false
  hosts: localhost
  vars:
    one: 1
    two: 2
    three: 3
  tasks:

  - name: Set facts for 1 
    set_fact:
      output:
        param_a: "{{ one }}"
        param_b: "{{ two }}"
    when: one == 1

  - name: Set facts for not 1
    set_fact:
      output:
        param_a: "{{ one }}"
        param_b: "{{ three }}"
    when: not one == 1

  - name: Print Output
    debug: 
      var: output
{% endraw %}
```

Here's the output of the above play:
```bash
{% raw %}
user@box:~/$ ansible-playbook example1.yml

PLAY [Handy Ansible Logic] ********************************

TASK [Set facts for 1] ************************************
ok: [localhost]

TASK [Set facts for not 1] ********************************
skipping: [localhost]

TASK [Print Output] ***************************************
ok: [localhost] => {
    "output": {
        "param_a": 1,
        "param_b": 2
    }
}
{% endraw %}
```
or, when we pass in a value for ```var_one``` to trigger the other condition: 

```bash
{% raw %}
user@box:~/$ ansible-playbook example1.yml -e "var_one=2"

PLAY [Handy Ansible Logic] ********************************

TASK [Set facts for 1] ************************************
skipping: [localhost]

TASK [Set facts for not 1] ********************************
ok: [localhost]

TASK [Print Output] ***************************************
ok: [localhost] => {
    "output": {
        "param_a": 2,
        "param_b": 3
    }
}
{% endraw %}
```

It's really straight forward and in this simple example the duplication isn't too bad, but you can see how this would get ugly pretty quickly if you're using a module that needs lots of parameters to be set, or if you have multiple parameters that need similar conditional logic - that would blow out really quickly. Even just setting 3 parameters with this logic would require 8 different tasks to cover the possibilities and some pretty nasty logic in the ```when:``` statements.

# The Solution

The answer to this mess turns out to be quite neat. We move the conditional logic out of the ```when:``` and into the call to the variable, the jinja statement, as variables in ansible are jinja templates. Here is what that would look like:

```yaml
{% raw %}
---
- name: Handy Ansible Logic
  gather_facts: false
  hosts: localhost
  vars:
    var_one: 1
    var_two: 2
    var_three: 3
  tasks:

  - name: Set values
    set_fact:
      output:
        param_a: "{{ var_one }}" 
        param_b: "{{ var_two if var_one == 1 else var_three }}"
      
  - name: Print Output
    debug: 
      var: output
{% endraw %}
```

As you can see we have been able to remove the duplicated code and the when statement. What I really like is the if condition doesn't make the logic hard to read - it's almost plain english. 

Here's the output of the above improved play logic:

```bash 
{% raw %}
user@box:~/$ ansible-playbook example2.yml

PLAY [Handy Ansible Logic] ********************************

TASK [Set values] *****************************************
ok: [localhost]

TASK [Print Output] ***************************************
ok: [localhost] => {
    "output": {
        "param_a": 1,
        "param_b": 2
    }
}
{% endraw %}
```

And, again to trigger the other condition:

```bash 
{% raw %}
user@box:~/$ ansible-playbook example2.yml -e "var_one=2"

PLAY [Handy Ansible Logic] ********************************

TASK [Set values] *****************************************
ok: [localhost]

TASK [Print Output] ***************************************
ok: [localhost] => {
    "output": {
        "param_a": 2,
        "param_b": 3
    }
}
{% endraw %}
```

We can still use jinja's filters like any other variable with this approach too: 

```yaml
{% raw %}

  - name: Set 
    set_fact:
      output:
        param_a: "{{ var_two if var_one == 1 else omit }}"

{% endraw %}
```
Since learning how to use this logic it has been really helpful in quite a few situations. 

All of the examples shown are available [here](https://github.com/matthewdennett/2022-06-01-Handy-Ansible-Logic) if you would like a copy. 

