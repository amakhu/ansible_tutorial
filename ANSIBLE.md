# ANSIBLE TUTORIAL

## ANSIBLE INSTALLATION

**Controller node requirements**:
- Python 3.x
- An SSH client
- Access to an Ansible Repository
- A dedicated user account that is configured with SSH and sudo permissions on managed hosts

**Using redhat repository to install Ansible**:
- subscription-manager repos --list
- subscription-manager repos --enable=ansible-2-for-rhel-8-x86_64-rpms
- yum install ansible
- ansible --version

**Using python-pip to install Ansible**:
- yum install -y python3-pip
- alternatives --set python /usr/bin/python3
- pip3 install ansible --user
- ansible --version

**Configuring Managed Hosts**:
- systemctl status sshd
- rpm -qa | grep python
- firewall-cmd --list-all

**Configuring the Ansible User**:
- The user must be able to SSH into the managed machines
- The user must be enabled to run tasks as root on the managed machines

**On control node**:
- ssh-keygen
- ssh-copy-id <managed_node_ip/fqdn>
- ssh <managed_node_ip/fqdn>

**Setting up privilege escalation on managed nodes**:
- echo "ansible ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/ansible

## ANSIBLE CONFIGURATION FILE (ansible.cfg)

If Ansible is installed from a package manager: /etc/ansible/ansible.cfg
If Ansible is installed from pip or from source, you can create it to override default settings in Ansible: 
- ansible-config init --disabled -t all > ansible.cfg

## INVENTORY FILE
The purpose of the Ansible inventory is to identify hosts that Ansible has to manage.
Inventory file might by created in Yaml or INI.


#### Adding ranges of hosts

If you have a lot of hosts with a similar pattern, you can add them as a range rather than listing each hostname separately.

**Example**:
```
[webservers]
www[01:50].example.com
```

You can specify a stride (increments between sequence numbers) when defining a numeric range of hosts.
The example would make the subdomains www01, www03, www05, …, www49 match, but not www00, www02, www50 and so on, because the stride (increment) is 2 units each step.

**Example**:
```
[webservers]
www[01:50:2].example.com
```

For numeric patterns, leading zeros can be included or removed, as desired. 

**Example**:
```
[databases]
db-[01:14].example.com
```

You can also define alphabetic ranges:

**Example**:
```
[databases]
db-[a:f].example.com
```

Ranges are inclusive.


#### Grouping hosts in Ansible Inventory

Within the inventory file, you can organize your servers into different groups and subgroups.
Beyond helping to keep your hosts in order, this practice will enable you to use group variables.

**Example**:
```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```


#### Hosts in multiple groups

You can put each host in more than one group.

**Example**:
```
[webservers]
foo.example.com
one.example.com

[dbservers]
one.example.com
two.example.com
```


#### Grouping groups: parent/child group relationships

You can create parent/child relationships among groups. 
Parent groups are also known as nested groups or groups of groups. 
For example, if all your production hosts are already in groups such as atlanta_prod and denver_prod, you can create a production group that includes those smaller groups.
To create parent/child relationships for groups: in INI format, use the **:children** suffix.
In YAML format, use the **children:** entry.

**Example**:
```
[atlanta_prod]
one.example.com
two.example.com

[denver_prod]
three.example.com
four.example.com

[production:children]
atlanta_prod
denver_prod
```

Child groups have a couple of properties to note:
- Any host that is member of a child group is automatically a member of the parent group.
- Groups can have multiple parents and children, but not circular relationships.
- Hosts can also be in multiple groups, but there will only be one instance of a host at runtime. Ansible merges the data from the multiple groups.



#### Default groups

Even if you do not define any groups in your inventory file, Ansible creates two default groups: all and ungrouped.
The all group contains every host. 
The ungrouped group contains all hosts that don’t have another group aside from all.
Every host will always belong to at least 2 groups (all and ungrouped or all and some other group).

The default location for inventory is a file called /etc/ansible/hosts. 
You can specify a different inventory file at the command line using the -i <path> option. 
You can also use multiple inventory files at the same time
  


#### Passing multiple inventory sources
  
You can target multiple inventory sources (directories, dynamic inventory scripts or files supported by inventory plugins) at the same time by giving multiple inventory parameters from the command line or by configuring ANSIBLE_INVENTORY. 
This can be useful when you want to target normally separate environments, like staging and production, at the same time for a specific action.
To target two inventory sources from the command line:
```
ansible-playbook get_logs.yml -i staging -i production
```

 #### Adding variables to inventory
 You can store variable values that relate to a specific host or group in inventory.
 
 Variables may be added directly to the hosts and groups in your main inventory file. 
 As you add more and more managed nodes to your Ansible inventory, however, you will likely want to store variables in separate host and group variable files.
  
 You can easily assign a variable to a single host, then use it later in playbooks. In INI:
 
 ```
 [atlanta]
 host1 http_port=80 maxRequestsPerChild=808
 host2 http_port=303 maxRequestsPerChild=909
 ```
 
Unique values like non-standard SSH ports work well as host variables. You can add them to your Ansible inventory by adding the port number after the hostname with a colon:
```
badwolf.example.com:5309
```

Connection variables also work well as host variables:

```
[targets]

localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=myuser
other2.example.com     ansible_connection=ssh        ansible_user=myotheruser
```
#### Assigning a variable to many machines: group variables

If all hosts in a group share a variable value, you can apply that variable to an entire group at once. In INI:
  
```
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```

#### host_vars and group_vars
Storing separate host and group variables files may help you organize your variable values more easily. 
Host and group variable files must use YAML syntax. 
Valid file extensions include ‘.yml’, ‘.yaml’, ‘.json’, or no file extension.
Ansible loads host and group variable files by searching paths relative to the inventory file or the playbook file.

If your inventory file at /etc/ansible/hosts contains a host named ‘foosball’ that belongs to two groups, ‘raleigh’ and ‘webservers’, that host will use variables at the following locations:
  
 ```
 /etc/ansible/group_vars/raleigh # can optionally end in '.yml', '.yaml', or '.json'
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
```

#### Inventory aliases
Setting the following variables control how Ansible interacts with remote hosts.

- __ansible_connection__: Connection type to the host. This can be the name of any of ansible’s connection plugins. SSH protocol types are smart, ssh or paramiko. The default is smart. 
  
- __ansible_host__: The name of the host to connect to, if different from the alias you wish to give to it.
  
- __ansible_port__: The connection port number, if not the default (22 for ssh)
  
- __ansible_user__: The user name to use when connecting to the host
  
- __ansible_password__: The password to use to authenticate to the host (never store this variable in plain text; always use a vault.
  
- __ansible_become__: Equivalent to ansible_sudo or ansible_su, allows to force privilege escalation

- __ansible_become_method__: Allows to set privilege escalation method

- __ansible_become_user__: Equivalent to ansible_sudo_user or ansible_su_user, allows to set the user you become through privilege escalation

- __ansible_become_password__: Equivalent to ansible_sudo_password or ansible_su_password, allows you to set the privilege escalation password (never store this variable in plain text; always use a vault.
