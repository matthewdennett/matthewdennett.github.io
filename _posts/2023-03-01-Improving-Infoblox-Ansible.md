---
layout: post
title: Improving Infoblox Ansible modules
tags:
- Python
- Ansible
- Infoblox
- Open Source
show: true
---

Infoblox is a popular tool that enables teams to centrally manage DNS, DHCP, and IP addresses (DDI) efficiently. One way to automate Infoblox is through the use of Ansible with the use of the [Infoblox Ansible collection](https://galaxy.ansible.com/infoblox/nios_modules).

However, when using the [Infoblox Ansible collection](https://galaxy.ansible.com/infoblox/nios_modules) I quickly encountered some missing functionality…

- Define a member as a Grid Master Candidate
- Assign members to a network
- Define a DHCP range

In this blog post, I will discuss some of the changes I have proposed to fix these issues that exists in [v1.4.1](https://galaxy.ansible.com/infoblox/nios_modules).


<!--more-->

#### Grid Master Candidate

The role of a GMC in the Infoblox Grid is quite important, it's pretty much the DR solution for management the Infoblox platform. As such I was really surprised that it wasn't already included in the nios_member module.

This was a fairly simple fix to implement. The Infoblox API(WAPI) member endpoint accepts a boolean for the ```master_candidate``` field to enable or disable a member as a Grid Master Candidate flag for a member. As the nios_modules use the WAPI API to apply configuration changes, I just need to update the nios_member module to accept the ```master_candidate``` field that would in turn be passed to the API endpoint.

To achieve this I just needed to add a small update to the ```ib_spec``` definition in the nios_member.py file. Here's the updated change...

{% highlight Python linenos %}
master_candidate=dict(type='bool', default=False)
{% endhighlight %}

This update adds the ```master_candidtate``` element with the default value being set to false.

Now you can define or update a Grid member to be a Grid Master Candiadate like show in the example below.

{% highlight yaml linenos %}
---
- name: Add a Grid Master Candidate to the grid with IPv4 address
  infoblox.nios_modules.nios_member:
    host_name: member01.localdomain
    master_candidate: true
    vip_setting:
      - address: 192.168.1.100
        subnet_mask: 255.255.255.0
        gateway: 192.168.1.1
    config_addr_type: IPV4
    platform: VNIOS
    comment: "Created by Ansible"
    state: present
    provider:
      host: "{{ inventory_hostname_short }}"
      username: admin
      password: admin
  connection: local
...
{% endhighlight %}

#### Network Member Assignment

Being able to assign Grid members to a network is a prerequisite for networks to be configured to server DHCP for fixed addresses or ranges. Not being able to assign members directly though the ansible nios_network module is quite limiting for a DDI product.

The WAPI API commonly uses the concept of a ‘struct’ when there is the need for data to be passed in with a specific structure. The WAPI doco describes a wide variety of structs and their associated properties. Each struct is identified with the key ```_struct``` and the value naming the type of struct being passed in.  Below is an example 'dhcpmember' struct needed to be passed to the WAPI API to associate the Grid member 'member1.localdomain' to a network object.

{% highlight yaml linenos %}
members:
  - _struct: 'dhcpmember'
    name: 'member1.localdomain'
{% endhighlight %}

First step to address this issue was to update the nios_network module to accept the addition ‘members’ element. Like earlier to as pretty straight forward updating the ```ib_spec``` definition in nios_network.py.

{% highlight Python linenos %}
members=dict(type='list', elements='dict', default=[])
{% endhighlight %}

This update adds the ```members``` element as a list of dictionaries with the default value being an empty list.

This change itself would be enough to enable member assignment, however it would require the user of the module to pass in a list of structs like show in the earlier example. It would work, but it a bit clunky needing the user to know the details of the WAPI struct required. To keep the module use simple I was thinking that the struct should be created by the module from a simple list passed in by the user. So the user would only need  to add the ```members``` element like this.

{% highlight yaml linenos %}
members:
  - name: 'member1.localdomain'
{% endhighlight %}

Much cleaner and straight forward for someone not **familiar** with the WAPI API structs.

To apply this conversion I added the following function to the ```api.py``` file, which is called when a network v4 or v6 object is being configured.

{% highlight Python linenos %}
def convert_members_to_struct(member_spec):
    ''' Transforms the members list of the Network module arguments into a
    valid WAPI struct. This function will change arguments into the valid
    wapi structure.
    '''
    if 'members' in member_spec.keys():
        member_spec['members'] = [{'_struct': 'dhcpmember', 'name': k['name']} for k in member_spec['members']]
    return member_spec
{% endhighlight %}

This function uses list comprehension to construct, the required struct key value pairs for each member in the list, when the ```members``` key has been defined in the ansible code.

Now you can assigne Grid Members to a newwork like the example below.

{% highlight yaml linenos %}
- name: Create network with member assignment for a network ipv4
  infoblox.nios_modules.nios_network:
    network: 192.168.10.0/24
    comment: This is a test comment
    members:
      - name: member1.infoblox
      - name: member2.infoblox
    state: present
    provider:
      host: "{{ inventory_hostname_short }}"
      username: admin
      password: admin
  connection: local
{% endhighlight %}


#### DHCP range

As assigning Grid members to a network wasn’t possible though the nios_network module a module for creating a DHCP range wouldn’t be of much use or be able to function correctly. Now with the above changes implemented, a new module to manage nios_range objects is possible. As this was a new module being added to the collection the changes required were quite a lot bigger.

Other than the creation of the module and ```ib_spec```  definition of the available input options, some addition logic to handle the require input data structs for the DHCP member type. This time some error checking is required to ensure only one DHCP member type is defined at any one time. For this, I added  the function `def convert_range_member_to_struct(module):` to the nios_range.py file that is called before the WAPI API object is created.

Here is what the function looks like…


{% highlight Python linenos %}
def convert_range_member_to_struct(module):
    '''This function will check the module input to ensure that only one member assignment type is specified at once.
    Member passed in is converted to the correct struct for the API to understand bassed on the member type.
    '''
    # Error checking that only one member type was defined
    params = [k for k in module.params.keys() if module.params[k] is not None]
    opts = list(set(params).intersection(['member', 'failover_association', 'ms_server']))
    if len(opts) > 1:
        raise AttributeError("'%s' can not be defined when '%s' is defined!" % (opts[0], opts[1]))

    # A member node was passed in. Ehsure the correct type and struct
    if 'member' in opts:
        module.params['member'] = {'_struct': 'dhcpmember', 'name': module.params['member']}
        module.params['server_association_type'] = 'MEMBER'
    # A FO association was passed in. Ensure the correct type is set
    elif 'failover_association' in opts:
        module.params['server_association_type'] = 'FAILOVER'
    # MS server was passed in. Ensure the correct type and struct
    elif 'ms_server' in opts:
        module.params['ms_server'] = {'_struct': 'msdhcpserver', 'ipv4addr': module.params['ms_server']}
        module.params['server_association_type'] = 'MS_SERVER'

    return module
{% endhighlight %}

Using list comprehension, a list of option keys is created for any option that has been configured with a value. This allows for us to check that only one DHCP member type has been defined by checking the length of the intersection between the two list and raise a descriptive error when more than one exists.

Next we set the appropriate ```server_association_type``` for the options passed in and construct the required data struct for the API to process.

So now we can create a network range like this...

{% highlight yaml linenos %}
- name: Configure a ipv4 range served by a member
  infoblox.nios_modules.nios_range:
    network: 192.168.10.0/24
    start: 192.168.10.10
    end: 192.168.10.20
    name: Test Range 1
    member: infoblox1.localdomain
    comment: this is a test comment
    state: present
    provider:
      host: "{{ inventory_hostname_short }}"
      username: admin
      password: admin
  connection: local
{% endhighlight %}

#### Conclusion

The collection includes a not of really helpfull modules for managing an Infoblox Grid. Unfortunatly, there are a number of modules that are yet to be developed and several modules are missing some options which are avaliable in the WAPI API. I hope that I can find some more time to commit to this project and help develop some of the missing modules. 

The GitHub project has recently changed over to a new set of maintainers, which has slowed down the review and QA process quite a lot whole they are still getting up to spped with the project. At the time of writing this post, the PR is yet to approved and merged. You can find this PR here if you would like to have a closer look.