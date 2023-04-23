---
layout: post
title: YAML Anchors
tags:
- YAML
- Ansible
show: true
---

In Ansible, you can define variables in a variety of places, including inventory files, playbook files, and role defaults and vars files. When defining variables, you can use YAML anchors and aliases to reduce repetition and make your code more readable.

{% highlight yaml linenos %}
vars:
  # Define an anchor called "common_vars"
  common_vars: &common_vars
      version: 2.0
      name: My Application

  # Define a variable that uses the "common_vars" anchor and adds additional information
  prod_vars: &prod_vars
      <<: *common_vars
      environment: production
{% endhighlight %}


<!--more-->

Here's an example of using anchors and aliases in an Ansible playbook:

{% highlight yaml linenos %}
---
- name: Example 1
  hosts: localhost
  gather_facts: false
  vars:
    # Define an anchor called "common_vars"
    common_vars: &common_vars
      version: 2.0
      name: My Application
      servers:
        - server1

    # Define a variable that uses the "common_vars" anchor and adds additional information
    app1_vars:
      <<: *common_vars
      environment: production
      servers:
        - prod_svr1
        - prod_svr2

    # Define another variable that uses the "common_vars" anchor and adds additional information
    app2_vars:
      <<: *common_vars
      environment: staging
      servers:
        - stg_svr1

  tasks:
    # Use the variables in tasks
    - name: Print app 1
      debug:
        var: app1_vars

    - name: Print app 2
      debug:
        var: app2_vars
...
{% endhighlight %}

In this example, we define an anchor called "common_vars" that contains some common variables used by multiple applications. We then use the alias syntax ```*``` to define two new variables, "app1_vars" and "app2_vars", that use the "common_vars" anchor and add additional information specific to each application.

Note that the "servers" key in the "app1_vars" and "app2_vars" variables overrides the "servers" key in the "common_vars" anchor, so each application has its own list of servers.

Finally, we use the variables in tasks to display the configuration of each application. Below is the output for each application, which includes the common variables and the additional information specific to each application.

{% highlight bash linenos %}
user@box:~/$ ansible-playbook example1.yml

PLAY [Example 1] **************************************************************
TASK [Print app 1] ************************************************************
ok: [localhost] => {
    "app1_vars": {
        "environment": "production",
        "name": "My Application",
        "servers": [
            "prod_svr1",
            "prod_svr2"
        ],
        "version": 2.0
    }
}

TASK [Print app 2] ************************************************************
ok: [localhost] => {
    "app2_vars": {
        "environment": "staging",
        "name": "My Application",
        "servers": [
            "stg_svr1"
        ],
        "version": 2.0
    }
}

PLAY RECAP ********************************************************************
{% endhighlight %}


Using anchors and aliases can be particularly helpful when defining complex variables that are used in multiple places in your playbook or role. Instead of copy-pasting the same data multiple times, you can define an anchor once and then use it throughout your code. This can make your code more readable, easier to maintain, and less error-prone.


Though anchors and aliases are really handy it can be easy over use them or used them when there is no benefit from using them. Here is one example that highlights a situation where I have found myself using an anchor with no benefit.

{% highlight yaml linenos %}

---
- name: Example 2
  hosts: localhost
  gather_facts: false
  vars:

    app1_vars: &app1_vars
      environment: production
      servers:
        - server1

    app2_vars: &app2_vars
      environment: staging
      servers:
        - server2
    # Define a list using the alias
    app_list1:
      - <<: *app1_vars
      - <<: *app2_vars

    # Define a list with YAML variables
    app_list2:
      - "{{app1_vars}}"
      - "{{app2_vars}}"


  tasks:
    # Use the variables in tasks
    - name: Print App List 1
      debug:
        var: app_list1

    - name: Print App List 2
      debug:
        var: app_list2
...
{% endhighlight %}

In this example, we define an anchor called "app1_vars" and "app2_vars" that contains some common variables describing our applications. We then use the alias syntax to define a list of our applications, "app_list1", that use the anchors of app1 and app2. Next we define the list "app2_list" using the regular YAML variable syntax.

Here both lists that have been created are identical. The use of an anchor in a situation like this may just be making the code harder to read and possibly a bit harder for the next person to understand what the goal was.

{% highlight bash linenos %}
user@box:~/$ ansible-playbook example2.yml
PLAY [Example 3] **************************************************************

TASK [Print App List 1] *******************************************************
ok: [localhost] => {
    "app_list1": [
        {
            "environment": "production",
            "servers": [
                "server1"
            ]
        },
        {
            "environment": "staging",
            "servers": [
                "server2"
            ]
        }
    ]
}

TASK [Print App List 2] *******************************************************
ok: [localhost] => {
    "app_list2": [
        {
            "environment": "production",
            "servers": [
                "server1"
            ]
        },
        {
            "environment": "staging",
            "servers": [
                "server2"
            ]
        }
    ]
}

PLAY RECAP ********************************************************************
{% endhighlight %}

In summary, YAML anchors and aliases are a powerful feature that can help you reduce repetition in your Ansible playbooks and roles and make them more readable and maintainable. By using anchors and aliases, you can avoid copy-pasting the same data multiple times and make it easier to update your variables in the future.

All of the examples shown are available [here](https://github.com/matthewdennett/2023-01-01-YAML-Anchors) if you would like a copy.