# Ansible

## What is it?
- Ansible is a software tool that provides automation for cross-platform computer support. It automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs.

<br>

## Contents
1. Inventories
2. Pinging hosts
3. Ad-hoc commands
4. Playbooks
5. Templates

<br>

## Host file or Inventory
- There are two categories of nodes: the control node and managed nodes
    - The control node is the machine running Ansible
    - A managed node is another machine that we want to change. Ansible does not need to be installed on these machines.

- The inventory file defines the hosts (managed nodes) and groups of hosts
    - The default Inventory file is found within `/etc/ansible/hosts` if using Ubuntu
    - Here you specify the hosts (within groups if wanted) and how to connect to them

- The following is an example of an inventory file using INI
```INI
example.example.example.com

[app_server]
192.168.10.100 ansible_connection=ssh ansible_ssh_private_key_file=/home/ubuntu/.ssh/eng74Jaredawskey.pem

[db_server]
192.168.10.200 ansible_connection=ssh ansible_ssh_private_key_file=/home/ubuntu/.ssh/eng74Jaredawskey.pem
```

<br>

- This is the same file written in YAML. Notice how there are repetitions. This structure should be used if each managed node has a separate connection or SSH key.

- In this case, the connection key and connection type are the exact same for both hosts so this structure is not ideal.

```yaml
all:
  hosts:
    example.example.example.com

  children:
    app_server:
      ansible_host: 192.168.10.100
      ansible_connection: ssh
      ansible_ssh_private_key_file: /home/ubuntu/.ssh/eng74Jaredawskey.pem

    db_server:
      ansible_host: 192.168.10.200
      ansible_connection: ssh
      ansible_ssh_private_key_file: /home/ubuntu/.ssh/eng74Jaredawskey.pem
```

<br>

- This is the same file but written more efficiently. We can do this because these inventory variables apply to all the groups.
```yaml
all:
  hosts:
    example.example.example.com

  children:
    app_server:
      ansible_host: 192.168.10.100

    db_server:
      ansible_host: 192.168.10.200

  vars:
    ansible_connection: ssh
    ansible_ssh_private_key_file: /home/ubuntu/.ssh/eng74Jaredawskey.pem
```

<br>

## Pinging a host
- One can check if the controller has SSH access to a host by using the ping command
    - This is an ad-hoc Ansible command

```bash
ansible all -m ping # pings all hosts
ansible host_a -m ping # pings hosts within the group host_a
```

<br>

## Ad-hoc commands
- While playbooks are good as they allow you to repeat tasks, ad-hoc commands are useful for tasks that you will rarely ever do
  - Examples could be to turning off all the machines in an area
  - Copy over some files once, and only once etc.

- They achieve a form of idempotence by checking the current state before they begin and doing nothing unless the current state is different from the specified final state

- An ad-hoc command looks like:
```yaml
$ ansible [pattern] -m [module] -a "[module options]"
```

- The Ansible way of running bash commands as sudo is with the `--become` keyword
```yaml
$ ansible all -a "apt-get update" --become
```

<br>

## Playbooks
- Ansible provisions through the use of an Ansible `playbook`. These are configuration files written in YAML providing instructions for what needs to be done in order to bring a managed node into a desired state.

- These playbooks are idempotent -- they have no negative effect on systems
    - If a system is already configured correctly then running a playbook on it will yield a system that will still be properly configured
    - It does so by checking task-by-task whether or not that part of the system is in the desired state. If it is in the desired state then the task will not run.


**Example:**
```yaml
# This is going to be our example Playbook which is written in YAML
# Your YAML playbook file starts after three dashes
---
# This example targets host_a_public

- name: install sql # can be anything
  hosts: host_a_public # specify a host or host group
  gather_facts: yes # gathers facts/states of machine before running the playbook
  become: true # become is used to run the commands as sudo

  tasks:
  - name: Install SQL DB
    apt: pkg=mysql-server state=present
```

<br>

## Templates and Variables
- `template` is another Ansible module that uses the Jinja2 templating language

- A Jinja template is simply a text file which can be generated in formats such as HTML, XML, CSV etc.

- We will be able to interpolate (substitute) variables into them, making them dynamic.

<br>

**How do you use variables in Ansible?**
- There are a couple of ways of doing so:
  1. You can create variables from the output of an Ansible task. This is done via the `register` keyword
  2. We can have a file containing variables and we indicate the path of it
  3. We can define variables within a job, on the same level as `hosts`

- Substitutions are used with `{% %}`or `{{}}`
    - `{% %}` are used for functions
    - `{{ }}` are used for variables

<br>

**Using variables**
```yaml
- hosts: all
  remote_user: root

  vars:
    app-IP:
      public: 192.168.10.100
      private: 192.168.10.200
```
- Here, we create a variable dictionary `app-IP`. To reference the public IP or private IP we can call this variable using `{{ app-IP["public"] }}