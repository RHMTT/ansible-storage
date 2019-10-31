# Exercise 3: Ansible Facts

## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
- [Takeaways](#takeaways)
- [Solution](#solution)

# Objective

Demonstration use of Ansible facts on network infrastructure.

Ansible facts are information derived from speaking to the remote storage elements.  Ansible facts are returned in structured data (JSON) that makes it easy manipulate or modify.  For example a network engineer could create an audit report very quickly using Ansible facts and templating them into a markdown or HTML file.

This exercise will cover:
- Building an Ansible Playbook from scratch.
- Using [ansible-doc](https://docs.ansible.com/ansible/latest/cli/ansible-doc.html).
- Using the [na_ontap_gather_facts module](https://docs.ansible.com/ansible/latest/modules/na_ontap_gather_facts_module.html#na-ontap-gather-facts-module).
- Using the [debug module](https://docs.ansible.com/ansible/latest/modules/debug_module.html).

#### Step 1

On the control host read the documentation about the `na_ontap_gather_facts` module and the `debug` module.

```bash
[student1@ansible storage-workshop]$ ansible-doc debug
```

What happens when you use `debug` without specifying any parameter?

```bash
[student1@ansible storage-workshop]$ ansible-doc na_ontap_gather_facts
```

How can you limit the facts collected ?


#### Step 2:

Ansible Playbooks are [**YAML** files](https://yaml.org/). YAML is a structured encoding format that is also extremely human readable (unlike it's subset - the JSON format)

Using your favorite text editor (`vim` is available on the control host) create a new file called `facts.yml`:  

```
[student1@ansible storage-workshop]$ vim facts.yml
```

Enter the following play definition into `facts.yml`:

```yaml
---
- name: gather information from Netapp
  hosts: ontap
  gather_facts: no
  connection: local
```

Here is an explanation of each line:
- The first line, `---` indicates that this is a YAML file.
- The `- name:` keyword is an optional description for this particular Ansible Playbook.
- The `hosts:` keyword means this playbook against the `localhost` defined in the inventory file.
- The `gather_facts: no` is required since as of Ansible 2.8 and earlier, this only works on Linux hosts, and not storage infrastructure.  We will use a specific module to gather facts for network equipment.


#### Step 3

Next, add the first `task`. This task will use the `na_ontap_gather_facts` module to gather facts about each device in the group `ontap`.


```yaml
---
- hosts: ontap
  name: Gather Netapp Ontap Facts
  gather_facts: no
  connection: local

  tasks:
  - name: Gather Netapp Facts
    na_ontap_gather_facts:
      state: info
      hostname: "{{ netapp_hostname }}"
      username: "{{ netapp_username }}"
      password: "{{ netapp_password }}"
      https: true
      validate_certs: false
    register: result

  - name: Show output
    debug: var=result
```

>A play is a list of tasks. Modules are pre-written code that perform the task.

#### Step 4

Execute the Ansible Playbook:

```
[student1@ansible storage-workshop]$ ansible-playbook facts.yml
```

The output should look as follows.

```bash
[student1@ansible storage-workshop]$ ansible-playbook facts.yml

PLAY [Gather Netapp Ontap Facts] ************************************************************************************************************

TASK [Gather Netapp Facts] ******************************************************************************************************************
ok: [localhost]

PLAY RECAP **********************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


#### Step 5

The play ran successfully and executed against the Netapp Ontap device. But where is the output? Re-run the playbook using the verbose `-v` flag.


```
[student1@ansible storage-workshop]$ ansible-playbook facts.yml -v
Using /home/student1/.ansible.cfg as config file

PLAY [Gather Netapp Ontap Facts] ************************************************************************************************************

TASK [Gather Netapp Facts] ******************************************************************************************************************
ok: [localhost]

TASK [Show output] **************************************************************************************************************************
ok: [localhost] => {
    "result": {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python",
            "ontap_facts": {
                "aggregate_info": {
                    "aggr0_citi_student1_01": {
                        "aggr_fs_attributes": {
                            "block_type": "64_bit",
                            "fsid": "598399611",
                            "type": "aggr"
.
.
 <output truncated for readability>
.
.
"vserver_name": "svm_citi_student1",
"vserver_subtype": "default",
"vserver_type": "data"
}
},
"vserver_login_banner_info": null,
---
"vserver_motd_info": null
}
},
"changed": false,
"failed": false,
"state": "info"
}
}

PLAY RECAP **********************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


> Note: The output returns key-value pairs that can then be used within the playbook for subsequent tasks. Also note that all variables that start with **ansible_** are automatically available for subsequent tasks within the play.

#### Step 6

Running a playbook in verbose mode is a good option to validate the output from a task. To work with the variables within a playbook you can use the `debug` module.

Write two additional tasks that display the devices' OS version and serial number.

<!-- {% raw %} -->
``` yaml
---
- hosts: ontap
  name: Gather Netapp Ontap Facts
  gather_facts: no
  connection: local

  tasks:
  - name: Gather Netapp Facts
    na_ontap_gather_facts:
      state: info
      hostname: "{{ netapp_hostname }}"
      username: "{{ netapp_username }}"
      password: "{{ netapp_password }}"
      https: true
      validate_certs: false

  - name: Show output
    debug:
      msg: "The ontap version number is: {{ ontap_facts.ontap_version }}"

  - name: Show node serial number
    debug:
      msg: "The serial number is: {{ ontap_facts.system_node_info['citi_student1-01'].node_serial_number }}"
```
<!-- {% endraw %} -->


#### Step 8

Now re-run the playbook but this time do not use the `verbose` flag:

```
[student1@ansible storage-workshop]$ ansible-playbook facts.yml

PLAY [Gather Netapp Ontap Facts] ************************************************************************************************************

TASK [Gather Netapp Facts] ******************************************************************************************************************
ok: [localhost]

TASK [Show output] **************************************************************************************************************************
ok: [localhost] => {
    "msg": "The ontap version number is: 160"
}

TASK [Show node serial number] **************************************************************************************************************
ok: [localhost] => {
    "msg": "The serial number is: 90232421897526747927"
}

PLAY RECAP **********************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


Using less than 20 lines of "code" you have just automated version and serial number collection. Imagine if you were running this against your production network! You have actionable data in hand that does not go out of date.

# Takeaways

- The [ansible-doc](https://docs.ansible.com/ansible/latest/cli/ansible-doc.html) command will allow you access to documentation without an internet connection.  This documentation also matches the version of Ansible on the control node.
- The [na_ontap_gather_facts](https://docs.ansible.com/ansible/latest/modules/na_ontap_gather_facts_module.html#na-ontap-gather-facts-module) gathers structured data specific for Netapp Ontap.
- The [debug module](https://docs.ansible.com/ansible/latest/modules/debug_module.html) allows an Ansible Playbook to print values to the terminal window.


# Solution

The finished Ansible Playbook is provided here for an answer key: [facts.yml](facts.yml).

---

# Complete

You have completed lab exercise 3

[Click here to return to the Ansible Storage Automation Workshop](../README.md)
