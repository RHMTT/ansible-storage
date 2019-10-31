# Exercise 4 - Using Variables and Loops

Previous exercises showed you the basics of Ansible Core.  In the next few exercises, we are going
to teach some more advanced ansible skills that will add flexibility and power to your playbooks.

Ansible exists to make tasks simple and repeatable.  We also know that not all systems are exactly alike and often require
some slight change to the way an Ansible playbook is run.  Enter variables.

Variables are how we deal with differences between your systems, allowing you to account for a change in port, IP address
or directory.

Loops enable us to repeat the same task over and over again.  For example, lets say you want to install 10 packages.
By using an ansible loop, you can do that in a single task.



For a full understanding of variables and loops; check out our Ansible documentation on these subjects.

- [Ansible Variables](http://docs.ansible.com/ansible/latest/playbooks_variables.html)
- [Ansible Loops](http://docs.ansible.com/ansible/latest/playbooks_loops.html)
- [Ansible Handlers](http://docs.ansible.com/ansible/latest/playbooks_intro.html#handlers-running-operations-on-change)

## Section 1: Using variables in a Playbook

To begin, we are going to create a new playbook, but it should look very familiar to the one you created previously


### Step 1:

Navigate to your home directory and create a new playbook.

```bash
vim motd.yml
```


### Step 2:

Add a play definition and some variables to your playbook.

```yml
—--
– hosts: ontap
  gather_facts: false
  name: Setup ONTAP
  connection: local
  vars:
    motd_message: I am the MOTD.  I am managed by {{ ansible_user }}
    state: "present"
```


### Step 3:

Add a new task called *set motd*.

```yml
  tasks:
  - name: set motd
    na_ontap_motd:
      state: "{{ state }}"
      vserver: "*"
      hostname: "{{ netapp_hostname }}"
      username: "{{ netapp_username }}"
      password: "{{ netapp_password }}"
      message: "{{ motd_message }}"
      https: true
      validate_certs: false
```

---

**NOTE**

- `vars:` You've told Ansible the next thing it sees will be a variable name +
- `motd_message` You are defining a list-type variable called motd_message.  What follows
is the message you want to set +

### Step 4

Run your playbook.

```bash
$ ansible-playbook motd.yml

PLAY [ontap] ********************************************************************************************************************************

TASK [set motd] *****************************************************************************************************************************
changed: [ontap-mgr]

PLAY RECAP **********************************************************************************************************************************
ontap-mgr                  : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Verify that the Ansible Playbook applied set your `motd_message` string.  Login to ontap cluster manager and check the running configuration on the Ontap device.

```bash
[student1@ansible storage-workshop]$ ssh admin@<ip_address_of_ontap-mgr>

citi_student1::> security login motd show
Vserver: citi_student1
Is the Cluster MOTD Displayed?: true
Message
-----------------------------------------------------------------------------
I am the MOTD.  I am managed by student1

Vserver: svm_citi_student1
Is the Cluster MOTD Displayed?: true
Message
-----------------------------------------------------------------------------
I am the MOTD.  I am managed by student1

2 entries were displayed.

citi_student1::>
```

---

## Section 2: Creating Volumes and enabling file shares

### Step 1:
Let's start by creating some files just for variables.  In your `storage_workshop` directory, create two new directories `host_vars` and `group_vars`.

```bash
$ cd ~/storage_workshop
$ mkdir -p host_vars group_vars

```
In the `group_vars` directory create a file called `ontap` and edit it so that it looks like the following:

```yaml
aggrs:
  - { name: "aggr1", vol_name: "ansibleVol1", vol_size: "10", vol_size_unit: "gb"  }
  - { name: "aggr1", vol_name: "ansibleVol2", vol_size: "10", vol_size_unit: "gb"  }
  ```


After that, create a playbook called `create-volume.yml` to create your volumes.

```YAML
- hosts: ontap
  name: Volume Action
  gather_facts: false
  connection: local

  tasks:
  - name: Volume Create
    na_ontap_volume:
      state: present
      name: "{{ item.vol_name }}"
      vserver: "svm_{{ vserver_name }}"
      aggregate_name: "{{ item.name }}"
      size: 10
      size_unit: gb
      policy: default
      junction_path: "/{{ item.vol_name }}"
      hostname: "{{ netapp_hostname }}"
      username: "{{ netapp_username }}"
      password: "{{ netapp_password }}"
      https: true
      validate_certs: false
    with_items: "{{ aggrs }}"
```
**NOTE**

`aggrs` In your variables file you defined a list-type variable call `aggrs`.  What follows is a list of aggregates and volumes you will create on your storage device.
`item` You are telling Ansible that this will expand into a list. You can choose members of that list by adding `item.subelement` to the variable.

`with_items: "{{ aggrs }}"` This is your loop which is instructing Ansible to perform this task on every `item` in `aggrs`


### Step 2:
Run your playbook:

```bash
$ ansible-playbook create_volume.yml -v
Using /home/student1/.ansible.cfg as config file

PLAY [Volume Action] ************************************************************************************************************************

TASK [Volume Create] ************************************************************************************************************************
ok: [ontap-mgr] => (item={u'vol_size': u'10', u'vol_size_unit': u'gb', u'vol_name': u'ansibleVol1', u'name': u'aggr1'}) => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "ansible_loop_var": "item", "changed": false, "item": {"name": "aggr1", "vol_name": "ansibleVol1", "vol_size": "10", "vol_size_unit": "gb"}}
changed: [ontap-mgr] => (item={u'vol_size': u'10', u'vol_size_unit': u'gb', u'vol_name': u'ansibleVol2', u'name': u'aggr1'}) => {"ansible_loop_var": "item", "changed": true, "item": {"name": "aggr1", "vol_name": "ansibleVol2", "vol_size": "10", "vol_size_unit": "gb"}}

PLAY RECAP **********************************************************************************************************************************
ontap-mgr                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Step 3:

Verify that the Ansible Playbook created your two volumes.  Login to ontap cluster manager and check that the volumes were created using the `volume show` command:

```bash
citi_student1::> volume show
Vserver   Volume       Aggregate    State      Type       Size  Available Used%
--------- ------------ ------------ ---------- ---- ---------- ---------- -----
citi_student1-01
          vol0         aggr0_citi_student1_01
                                    online     RW      72.71GB    64.49GB    6%
svm_citi_student1
          ansibleVol1  aggr1        online     RW         10GB     9.50GB    0%
svm_citi_student1
          ansibleVol2  aggr1        online     RW         10GB     9.50GB    0%
svm_citi_student1
          svm_citi_student1_root
                       aggr1        online     RW          1GB    971.6MB    0%
4 entries were displayed.
```

## Section 3:  YAML Aliases

The previous playbook can also be written this way:

```YAML
- hosts: ontap
  name: Volume Action
  gather_facts: false
  connection: local
  vars:
    login: &login
      hostname: "{{ netapp_hostname }}"
      username: "{{ netapp_username }}"
      password: "{{ netapp_password }}"
      https: true
      validate_certs: false

  tasks:
  - name: Volume Create
    na_ontap_volume:
      state: present
      name: "{{ item.vol_name }}"
      vserver: "svm_{{ vserver_name }}"
      aggregate_name: "{{ item.name }}"
      size: 10
      size_unit: gb
      policy: default
      junction_path: "/{{ item.vol_name }}"
      <<: *login
    with_items: "{{ aggrs }}"
```


Look at the variables on lines 5-11. This is the setup and use of a YAML alias.  This is not specific to Ansible, this is a YAML feature.  It is created in a variable section and its format is as follows.
```yaml
vars:
  alias_name: &alias_name
    variable: value
    variable: value
```
This makes ‘alias_name’ represent all the variable:value pairs that are setup in it.  It is referenced with the following line.
```
<<: *alias_name
```
I am using an alias called ‘login’ and you can see how this will save you a lot of typing in the future.





---



---



---


---
