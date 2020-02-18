# Exercise 2 - First Ansible Playbook

## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
- [Takeaways](#takeaways)
- [Solution](#solution)

# Objective

Use Ansible to update the configuration of routers.  This exercise will not create an Ansible Playbook, but use an existing provided one.

This exercise will cover
- examining an existing Ansible Playbook
- executing an Ansible Playbook on the command line using the `ansible-playbook` command
- check mode (the `--check` parameter)
- verbose mode (the `--verbose` or `-v` parameter)

# Guide

#### Step 1

Navigate to the `storage-workshop` directory if you are not already there.

```bash
[student1@ansible ~]$ cd ~/storage-workshop/
[student1@ansible storage-workshop]$
[student1@ansible storage-workshop]$ pwd
/home/student1/storage-workshop
```

Examine the provided Ansible Playbook named `playbook.yml`.  Feel free to use your text editor of choice to open the file.  The sample below will use the Linux `cat` command.

```bash
[student1@ansible storage-workshop]$ cat playbook.yml
---
- hosts: ontap
  name: Set Ontap SNMP String
  gather_facts: no
  connection: local

  tasks:
  - name: ensure the the desired snmp strings are present
    na_ontap_snmp:
      state: present
      community_name: ansible_public
      access_control: 'ro'
      hostname: "{{ netapp_hostname }}"
      username: "{{ netapp_username }}"
      password: "{{ netapp_password }}"
      https: true
      validate_certs: false

```

 - `cat` - Linux command allowing us to view file contents
 - `playbook.yml` - provided Ansible Playbook

We will explore in detail the components of an Ansible Playbook in the next exercise.  It is suffice for now to see that this playbook will set one Netapp snmp string

```
snmp-server community ansible-public RO
```

#### Step 3

Run the playbook using the `ansible-playbook` command:

```bash
[student1@ansible storage-workshop]$ ansible-playbook playbook.yml

PLAY [localhost] ****************************************************************************************************************************

TASK [ensure the the desired snmp strings are present] **************************************************************************************
ok: [localhost]

---
PLAY RECAP **********************************************************************************************************************************
localhost                  : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

#### Step 4

Verify that the Ansible Playbook worked.  Login to ontap cluster and check the running configuration on the Cloud Ontap cluster manager.

```bash
[student1@ansible storage-workshop]$ ssh admin@<ip_address_of_cluster_mgr>

citi_student1::> system snmp show
contact:

location:

authtrap:
        0
init:
        1
traphosts:
        -
community:
        ro  ansible_public
```


#### Step 5

The `na_ontap_snmp` module is idempotent. This means, a configuration change is pushed to the device if and only if that configuration does not exist on the end hosts.

>Need help with Ansible Automation terminology?  Check out the [glossary here](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html) for more information on terms like idempotency.

To validate the concept of idempotency, re-run the playbook:

```bash
[student1@ansible storage-workshop]$  ansible-playbook playbook.yml

PLAY [localhost] ****************************************************************************************************************************

TASK [ensure the the desired snmp strings are present] **************************************************************************************
ok: [localhost]

PLAY RECAP **********************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

[student1@ansible storage-workshop]$
```

> Note: See that the **changed** parameter in the **PLAY RECAP** indicates 0 changes.

Re-running the Ansible Playbook multiple times will result in the same exact output, with **ok=1** and **change=0**.  Unless another operator or process removes or modifies the existing configuration on rtr1, this Ansible Playbook will just keep reporting **ok=1** indicating that the configuration already exists and is configured correctly on the network device.  


#### Step 6

Now update the task to add one more SNMP RO community string named `ansible-test`.  

```
snmp-server community ansible-test RO
```

Use the text editor of your choice to open the `playbook.yml` file to add the command:

```bash
[student1@ansible storage-workshop]$ vim playbook.yml
```

The Ansible Playbook will now look like this:

``` yaml
---
- hosts: ontap
  gather_facts: no
  connection: local

  tasks:
  - name: ensure the the desired snmp strings are present
    na_ontap_snmp:
      state: present
      community_name: ansible_public
      access_control: 'ro'
      hostname: "{{ netapp_hostname }}"
      username: "{{ netapp_username }}"
      password: "{{ netapp_password }}"
      https: true
      validate_certs: false

  - name: ensure the the desired snmp strings are present
    na_ontap_snmp:
      state: present
      community_name: ansible_test
      access_control: 'ro'
      hostname: "{{ netapp_hostname }}"
      username: "{{ netapp_username }}"
      password: "{{ netapp_password }}"
      https: true
      validate_certs: false
```



#### Step 7

Re-run this playbook to push the changes.

```bash
PLAY [localhost] ****************************************************************************************************************************

TASK [ensure the the desired snmp strings are present] **************************************************************************************
ok: [localhost]

TASK [ensure the the desired snmp strings are present] **************************************************************************************
changed: [localhost]

PLAY RECAP **********************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

#### Step 10

Verify that the Ansible Playbook applied `ansible-test` community.  Login to ontap cluster manager and check the running configuration on the Ontap device.

```bash
[student1@ansible storage-workshop]$ ssh admin@<ip_address_of_ontap-mgr>

rtr1#sh run | i snmp
citi_student1::> system snmp show
contact:

location:

authtrap:
        0
init:
        1
traphosts:
        -
community:
        ro  ansible_public
        ro  ansible_test

citi_student1::>
```

# Takeaways

- the **na_ontap** modules are idempotent, meaning they are stateful
- **verbose mode** allows us to see more output to the terminal window, including which commands would be applied
- This Ansible Playbook could be scheduled in **Red Hat Ansible Tower** to enforce the configuration.  For example this could mean the Ansible Playbook could be run once a day for a particular device.  

# Solution

The finished Ansible Playbook is provided here for an answer key: [playbook.yml](../playbook.yml).

---

# Complete

You have completed lab exercise 2

[Click here to return to the Ansible Storage Automation Workshop](../README.md)
