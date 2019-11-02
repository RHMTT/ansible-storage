# Exercise 1 - Exploring the lab environment

## Table of Contents

- [Objective](#objective)
- [Diagram](#diagram)
- [Guide](#guide)


# Objective

Explore and understand the lab environment.  This exercise will cover
- Determining the Ansible version running on the control node
- Locating and understanding the Ansible configuration file (`ansible.cfg`)
- Locating and understanding an `ini` formatted inventory file


# Diagram

![Red Hat Ansible Automation Lab Diagram](../../../images/ansible-storage-workshop.png)

There is the Ansible Control Node, Netapp Cloud Ontap working environment and two RHEL 7 nodes.  Most of this workshop will be spent configuring the storage device.


# Guide

## Step 1

Navigate to the `storage-workshop` directory.

```
[student1@ansible ~]$ cd ~/storage-workshop/
[student1@ansible storage-workshop]$
[student1@ansible storage-workshop]$ pwd
/home/student1/storage-workshop
```
 - `~` - the tilde in this context is a shortcut for `/home/student1`
 - `cd` - Linux command to change directory
 - `pwd` - Linux command for print working directory.  This will show the full path to the current working directory.

## Step 2

Run the `ansible` command with the `--version` command to look at what is configured:


```
ansible 2.8.1
  config file = /home/student1/.ansible.cfg
  configured module search path = [u'/home/student1/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Jun 11 2019, 12:19:05) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
```

> Note: The ansible version you see might differ from the above output

This command gives you information about the version of Ansible, location of the executable, version of Python, search path for the modules and location of the `ansible configuration file`.

## Step 3

Use the `cat` command to view the contents of the `ansible.cfg` file.

```
[student1@ansible ~]$ cat ~/.ansible.cfg
[defaults]
connection = smart
timeout = 60
forks = 10
inventory = ~student1/lightbulb/lessons/lab_inventory/
host_key_checking = False
```

Note the following parameters within the `ansible.cfg` file:

 - `inventory`: shows the location of the ansible inventory being used

## Step 4

The scope of a `play` within a `playbook` is limited to the groups of hosts declared within an Ansible **inventory**. Ansible supports multiple [inventory](http://docs.ansible.com/ansible/latest/intro_inventory.html) types. An inventory could be a simple flat file with a collection of hosts defined within it or it could be a dynamic script (potentially querying a CMDB backend) that generates a list of devices to run the playbook against.

In this lab you will work with a file based inventory written in the **ini** format. Use the `cat` command to view the contents of your inventory:

```bash
[student1@ansible ~]$ cat ~/lightbulb/lessons/lab_inventory/inventory
```

```
[web]
node-2 ansible_host=3.95.216.234
node-1 ansible_host=54.196.255.155

[ansible]
control ansible_host=3.80.127.176

[all:vars]
ansible_user=student1
ansible_ssh_pass=r3dh4t
ansible_port=22
vserver=citi_student1
data_ips=172.48.15.38

[ontap]
ontap-mgr ansible_host=172.48.0.80 netapp_hostname=172.48.0.80 netapp_username=admin netapp_password=redhat123
```

## Step 5

In the above output every `[ ]` defines a group. For example `[web]` is a group that contains the hosts `node-1` and `node-2`. Groups can also be _nested_.

> Parent groups are declared using the `children` directive. Having nested groups allows the flexibility of assigining more specific values to variables.


> Note: A group called **all** always exists and contains all groups and hosts defined within an inventory.


We can associate variables to groups and hosts.

Host variables can be defined on the same line as the host themselves. For example for the host `rtr1`:

```
ontap-mgr ansible_host=172.48.0.80 ansible_user=admin ansible_password=redhat123
```

 - `ontap-mgr` - The name that Ansible will use.  This can but does not have to rely on DNS
 - `ansible_host` - The IP address that ansible will use, if not configured it will default to DNS
 - `private_ip` - This value is not reserved by ansible so it will default to a [host variable](http://docs.ansible.com/ansible/latest/intro_inventory.html#host-variables).  This variable can be used by playbooks or ignored completely.

Group variables groups are declared using the `vars` directive. Having groups allows the flexibility of assigning common variables to multiple hosts. Multiple group variables can be defined under the `[group_name:vars]` section. For example look at the group `all`:

```
[[all:vars]
ansible_user=student1
ansible_ssh_pass=r3dh4t
ansible_port=22
netapp_hostname=172.48.0.80
netapp_username=admin
netapp_password=redhat123
```

 - `ansible_user` - The user ansible will be used to login to this host, if not configured it will default to the user the playbook is run from
 - `ansible_network_os` - This variable is necessary while using the `network_cli` connection type within a play definition, as we will see shortly.
 - `ansible_connection` - This variable sets the [connection plugin](https://docs.ansible.com/ansible/latest/plugins/connection.html) for this group.  This can be set to values such as `netconf`, `httpapi` and `network_cli` depending on what this particular network platform supports.


# Complete

You have completed lab exercise 1

---
[Click Here to return to the Ansible Storage Automation Workshop](../README.md)
