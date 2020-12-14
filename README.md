# Ansible

## What is it?
- Ansible is a software tool that provides automation for cross-platform computer support. It automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs.

<br>

## Contents

1. [Inventories](https://github.com/jaredsparta/Ansible-Notes#user-content-host-file-or-inventory)
2. [Pinging hosts](https://github.com/jaredsparta/Ansible-Notes#user-content-pinging-a-host)
3. [Ad-hoc commands](https://github.com/jaredsparta/Ansible-Notes#user-content-ad-hoc-commands)
4. [Playbooks](https://github.com/jaredsparta/Ansible-Notes#user-content-playbooks)
5. [Templates](https://github.com/jaredsparta/Ansible-Notes#user-content-templates-and-variables)
6. [Security & Ansible vaults](https://github.com/jaredsparta/Ansible-Notes#user-content-security-&-ansible-vaults)
7. [Blocks & Error Handling]()
8. [Creating EC2 instances with Ansible](https://github.com/jaredsparta/Ansible-Notes#user-content-creating-ec2-instances-with-ansible)

<br>

##  Host file or Inventory
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

[Back to top](https://github.com/jaredsparta/Ansible-Notes#user-content-contents)

<br>

## Pinging a host
- One can check if the controller has SSH access to a host by using the ping command
    - This is an ad-hoc Ansible command

```bash
ansible all -m ping # pings all hosts
ansible host_a -m ping # pings hosts within the group host_a
```

[Back to top](https://github.com/jaredsparta/Ansible-Notes#user-content-contents)

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

[Back to top](https://github.com/jaredsparta/Ansible-Notes#user-content-contents)

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

[Back to top](https://github.com/jaredsparta/Ansible-Notes#user-content-contents)

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

[Back to top](https://github.com/jaredsparta/Ansible-Notes#user-content-contents)

<br>

## Security & Ansible Vaults

- We can use a few ways to keep security tight:
  1. Secrets
  2. Environment variables
  3. Software like Ansible that use keys for what they do

- What to consider:

  **`.gitignore`**
  - Is it protected from online viewing? **YES**
  - Is it segregated from the code goes online, on GitHub? **NO**
  - Is it encrypted? **NO**
  - Is it hard to share these files with colleagues? **NO**

  **`Environment variables`**
  - Is it protected from online viewing? **YES**
  - Is it segregated from the code goes online, on GitHub? **YES**
  - Is it encrypted? **NO**
  - Is it hard to share these files with colleagues? **NO**

  **`Ansible Vault`**
  - Is it protected from online viewing? **YES**
  - Is it segregated from the code goes online, on GitHub? **YES**
  - Is it encrypted? **YES**
  - Is it hard to share these files with colleagues? **NO**

- We will need to install two things using pip while inside our controller:
  1. `pip3 install awscli`
  2. `pip3 install boto boto3`

**Creating ansible vaults**
- There are a few shell commands you will need to create an Ansible vault:

  1. Use `$ ansible-vault create <name-of-key-file>.yml`. I will name mine `aws_keys.yml`
  2. Input a memorable password
  3. You will be put into a VIM editor, escape with `<ESC>` then write `:q`
  4. The vault will now be created, albeit with no stored information

- You 

- There are a few other commands you can use:
  1. `$ ansible-vault edit aws_keys.yml` -- can edit the file
  2. `$ ansible-vault view aws_keys.yml` -- can view the contents, needing a password

> Notice how if you `$ cat aws_keys.yml` you will get an encrypted output so you will need to use the `view` option and input your password

[Back to top](https://github.com/jaredsparta/Ansible-Notes#user-content-contents)

<br>

## Blocks

- Blocks create logical groups of tasks. They offer ways to handle task errors, similar to exception handling in Python.

**Grouping tasks with blocks**
- Specifying tasks within a block allows you to set block-specific directives such as `become`, `ignore_errors` etc.

- An example is the following. If we wanted to provision a host to install, configure and start an Apache server only if the OS of the host is `CentOS` we can do
```yaml
tasks:
   - name: Install, configure, and start Apache
     block:
       - name: Install httpd and memcached
         ansible.builtin.yum:
           name:
           - httpd
           - memcached
           state: present

       - name: Apply the foo config template
         ansible.builtin.template:
           src: templates/src.j2
           dest: /etc/foo.conf

       - name: Start service bar and enable it
         ansible.builtin.service:
           name: bar
           state: started
           enabled: True
     when: ansible_facts['distribution'] == 'CentOS'
     become: true
     become_user: root
     ignore_errors: yes
```

- Ansible will run the check for the `when` condition BEFORE running the three tasks. All three tasks will also inherit the other parameters.

<br>

**Handling errors**

- Two keywords here:
  1. `rescue`
  2. `always`

- Using blocks with either of those words will change how you handle errors when running playbooks.

- You can even use both for the same block, allowing for more complex ways to handle errors

- `rescue`:
  - Rescue blocks specify what tasks to run when any task in the block fails.
  - If an error occurs in the block and the rescue task succeeds, Ansible reverts the failed status of the original task for the run and continues to run the play as if the original task had succeeded. The rescued task is considered successful, and does not trigger max_fail_percentage or any_errors_fatal configurations.
  - It is at the same level as the `block` keyword
  - This is analagous to `try-except` in Python
  - An example:
  ```yaml
    tasks:
    - name: Handle the error
      block:
      - name: Force a failure
        command: /bin/false
      - name: Never print this
        debug:
          msg: 'I never execute, due to the above task failing, :-('

      rescue:
      - name: Print when errors
        debug:
          msg: 'I caught an error, can do stuff here to fix it, :-)'
  ```
- `always`:
  - The tasks within an `always` section will run regardless of the task status of the tasks in the block
  - It is at the same level as the `block` keyword
  - An example:
  ```yaml
  - name: Showing an always block
    block:
      - name: Prints message
        debug:
          msg: 'Valid execution'
    
      - name: Failed task
        command: /bin/false

      - name: this never runs
        debug:
          msg: 'This never runs'
  
    always:
      - name:
        debug:
          msg: 'This always runs'
  ```

[Back to top](https://github.com/jaredsparta/Ansible-Notes#user-content-contents)

<br>

## Creating EC2 instances with Ansible

- There are currently two well-known modules that deal with creating and managing EC2 instances:
    1. `community.aws.ec2`
    2. `community.aws.ec2_instance`
    - Neither are too difficult a module to use but do ensure to follow documentation thoroughly

- What are the differences?
    1. `ec2` needs a Python version >= 2.6 as well as `boto` to be present on the host while `ec2_instance` needs that as well as `boto3` and `botocore`

    2. `ec2` can create EC2 spot instances but `ec2_instance` cannot.

    3. `ec2` uses the older boto Python module to interact with the EC2 API and no longer receives new features

<br>

**`community.aws.ec2`**

- Keywords for `ec2`:
    1. `aws_access_key` and `aws_secret_key` 
        - required; these are keys to be able to enter into AWS and actually do these steps
        - **you will want to store these keys inside an Ansible-vault -- read on to see how**
    2. `count`: 
        - the number of instances to launch, defaulted to 1
    3. `group` or `group_id`: 
        - refers to which security groups to associate with the instance (several can be chosen using lists in the same syntax as Python)
    4. `instance_type`: 
        - required; dictates what type the instance is (e.g. `t2.micro`)
    5. `image`: 
        - required when `state=present` -- the AMI id
    6. `instance_ids`: 
        - used if you want to terminate, start or stop pre-existing instances
    7. `key_name`: 
        - the key pair to use on the instance; must already exist on AWS
        - keys can be created/deleted using the `ec2_key` module
    8. `region`: 
        - the AWS region to associate to the instance
    9. **`state`**: 
        - if `state=absent` then you will need to input which `instance_ids` to terminate
        -  if `state` is `running`, `stopped` or `restarted` then either `instance_ids` or `instance_tags` is required
        - the default value is `present` -- if you do not specify then Ansible will assume you are trying to create this instance

- Examples:
    ```yaml
    - name: Creating an instance that is part of several SG's
      amazon.aws.ec2:
        key_name: mykey
        group: ['databases', 'internal-services', 'sshable',    'and-so-forth']
        instance_type: m1.large
        image: ami-6e649707
        wait: yes
        wait_timeout: 500
        count: 5
        instance_tags:
            db: postgres
        monitoring: yes
        vpc_subnet_id: subnet-29e63245
        assign_public_ip: yes
    ```

    - The following is an example of starting, restarting or terminating an EC2 instance -- just change the state to `restarted` or `absent` for the latter two

    ```yaml
    - name: start the instances specified in tags
      amazon.aws.ec2:
        state: running
        instance_tags:
            Name: ExtraPower
    ```


<br>

[Back to top](https://github.com/jaredsparta/Ansible-Notes#user-content-contents)

---
**Used:**

- [Ansible-vault with playbooks](https://docs.ansible.com/ansible/latest/user_guide/vault.html#vault)

- [Using the `ec2` module](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_module.html#ansible-collections-amazon-aws-ec2-module)

- [Using the `ec2_instance` module](https://docs.ansible.com/ansible/latest/collections/community/aws/ec2_instance_module.html)

