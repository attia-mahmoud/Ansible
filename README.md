# Ansible

- Agentless
- OpenSSH
- Decentralized
- New major release approximately twice a year
- Open Source with 58k stars on GitHub

## Configuration

Found under **/etc/ansible**

### Inventory (/etc/ansible/hosts)

This is where you add the IP addresses of the worker nodes and optionally organize them into groups.

```bash
[hosts]
192.168.120.201
localhost:2222

[db]
192.168.122.207
foo.example.com
```

A host can belong to more than one group.

You can assign variables to individual hosts or to a group of hosts.

```bash
[atlanta]
host1 ansible_connection=ssh     ansible_user=myuser
host2 ansible_password=p@ssword

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```

### ansible.cfg (/etc/ansible/ansible.cfg)

Configuration file for Ansible

## Ad-hoc Commands

```
$ ansible <pattern> -m <module_name> -a "<module options">
```

## Playbooks

Playbooks are automation blueprints, in `YAML` format, that Ansible uses to deploy and configure nodes in an inventory.

A playbook is composed of one or more ‘plays’ in an ordered list. Each play executes part of the overall goal of the playbook, running one or more tasks. Each task calls an Ansible module.

```
- name: Update web servers
  hosts: webservers
  remote_user: root

  tasks:
  - name: Ensure apache is at the latest version
    ansible.builtin.yum:
      name: httpd
      state: latest
  - name: Write the apache config file
    ansible.builtin.template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf

- name: Update db servers
  hosts: databases
  remote_user: root

  tasks:
  - name: Ensure postgresql is at the latest version
    ansible.builtin.yum:
      name: postgresql
      state: latest
  - name: Ensure that postgresql is started
    ansible.builtin.service:
      name: postgresql
      state: started

```

## Conditionals

In a playbook, you may want to execute different tasks, or have different goals, depending on the value of a fact (data about the remote system), a variable, or the result of a previous task. You may want the value of some variables to depend on the value of other variables. Or you may want to create additional groups of hosts based on whether the hosts match other criteria.

### `when`

```jsx
tasks:
  - name: Shut down Debian flavored systems
    ansible.builtin.command: /sbin/shutdown -t now
    when: ansible_facts['os_family'] == "Debian"
```

Using `and` and `or` operators:

```jsx
when: (ansible_facts['distribution'] == "CentOS" and ansible_facts['distribution_major_version'] == "6") or
          (ansible_facts['distribution'] == "Debian" and ansible_facts['distribution_major_version'] == "7")
```

As a list (only applies with `and` operators):

```jsx
when:
      - ansible_facts['distribution'] == "CentOS"
      - ansible_facts['distribution_major_version'] == "6"
```

Based on the success or failure of a task:

```jsx
tasks:
  - name: Register a variable, ignore errors and continue
    ansible.builtin.command: /bin/false
    register: result
    ignore_errors: true

  - name: Run only if the task that registered the "result" variable fails
    ansible.builtin.command: /bin/something
    when: result is failed

  - name: Run only if the task that registered the "result" variable succeeds
    ansible.builtin.command: /bin/something_else
    when: result is succeeded

  - name: Run only if the task that registered the "result" variable is skipped
    ansible.builtin.command: /bin/still/something_else
    when: result is skipped

  - name: Run only if the task that registered the "result" variable changed something.
    ansible.builtin.command: /bin/still/something_else
    when: result is changed
```

---

## Variables

### Registering Variables

You can create variables from the output of an Ansible task with the task keyword `register`. You can use registered variables in any later tasks in your play.

```jsx
- hosts: web_servers
  tasks:
     - name: Run a shell command and register its output as a variable
       ansible.builtin.shell: /usr/bin/foo
       register: foo_result

     - name: Run a shell command using output of the previous task
       ansible.builtin.shell: /usr/bin/bar
       when: foo_result.rc == 5
```

### Finding content within variables

```jsx

example usecase:
when: motd_contents.stdout.find('hi') != -1
```

### Converting a variable to a list

```jsx
loop: "{{ home_dirs.stdout_lines }}"
loop: "{{ home_dirs.stdout.split() }}"
```

---

## Debugging

To access the information gathered about the remote hosts, use `ansible_facts`.

```jsx
- name: Show facts available on the system
  ansible.builtin.debug:
    var: ansible_facts
```

```jsx
- name: Check contents for emptiness
        ansible.builtin.debug:
          msg: "Directory is empty"
        when: contents.stdout == ""
```

---

## Masking

Using the pipe operator to treat a string as an integer

```jsx
ansible_facts['lsb']['major_release'] | int >= 6
```

or as a boolean

```jsx
tasks:
    - name: Run the command if "monumental" is true
      ansible.builtin.shell: echo "This certainly is monumenal!"
      when: monumental | bool
```

---

## Loops

```jsx
tasks:
    - name: Run with items greater than 5
      ansible.builtin.command: echo {{ item }}
      loop: [ 0, 2, 4, 6, 8, 10 ]
      when: item > 5
```

Providing a default for undefined variables

```
- name: Skip the whole task when a loop variable is undefined
  ansible.builtin.command: echo {{ item }}
  loop: "{{ mylist|default([]) }}"
  when: item > 5
```
