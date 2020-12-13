# Common syntax within Ansible playbooks

**Common Ansible modules**
> The following modules are built-in unless otherwise stated

- `apt`
    - Things relating to the package manager for Ubuntu

- `copy`
    - Copies a file from the local or host machine to a location on the host machine
    - You can also use it to create a new file with some content using the `content` key word

- `file`
    - Can set attributes of files, directories and symlinks
    - Can remove files, directories or symlinks

- `blockinfile`
    - Can insert, update or remove a multi-line block of text from within a file

- `shell`
    - Can run commands on the host machine
    - Similar to `command` module but runs the command through a shell (`/bin/sh` on the remote note)

- `service`
    - Controls services on remote hosts

- `npm`
   - Used to specify commands from the npm package manager
> Run `ansible-galaxy collection install community.general` inside controller before use in playbooks

- `get_url`
    - Used to downloads files from HTTP, HTTPS or FTP from a name to the host server

- `uri`
  - Similar to `get_url`, this one interacts with HTTP and HTTPS web services
  - Can also POST not just GET, unlike `get_url`

- `systemd`
  - Controls systemd services on remote hosts

<br>

**Handlers**

- These are needed when you only want a task to run if a change is made on the machine -- otherwise you wouldn't want to
    - For example, if you change a configuration file of a service, you may want to restart it so it takes those new files into account

- How does it work?
    - A task can send a notification which a handler will be able to see. If it changes then the handler task will run
    - A single task can create more than one notifications

- Example:
```yaml
---
- name: Verify apache installation
  hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
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
    notify:
    - Restart apache

  - name: Ensure apache is running
    ansible.builtin.service:
      name: httpd
      state: started

  handlers:
    - name: Restart apache
      ansible.builtin.service:
        name: httpd
        state: restarted
```
- Explanation:
    - The second task (`Write the apache config file`) creates a notification if the status of that task is `changed` instead `ok`
    - In the `handlers` section, we have a handler named with the same notification which will run when that task is `changed`
    - If the file does change, then the task runs


<br>

## Examples
```yaml
- name: install nodejs, nginx
    become: true
    apt:
      name: 
        - nodejs
        - nginx
      state: present
      update_cache: yes
```
- You can install multiple packages within the same task using this syntax

<br>

```yaml
---
- name: multiple shell commands
  hosts: hostA

  tasks:
  - name: showing how to run multiple shell commands in one task
    shell: |
      <command 1>
      <command 2>
      <command 3>
    args:
      chdir: ~/files/
```

- Using `|` you can specify more than one command; if you do so you must specify any change of directory etc. inside the `args` keyword

<br>

```yaml
tasks:
  - name: get the needed files for nodejs
    become: true
    get_url:
      url: https://deb.nodesource.com/setup_12.x
      dest: ~/
      mode: 755

  - name: run the nodejs bash script so nodejs can be installed
    become: true
    shell:
      cmd: ~/setup_12.x
```
- This is the Ansible equivalent of `curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`
    - `curl` will download the bash script (you can see it is a bash file if you go to the URL) and it pipes it into the bash command which will run it
    - This would be the exact same as downloading the bash file into the host and then running it using a shell command

<br>

```yaml
- hosts: all
  tasks:
  - name: Creating a file with content
    copy:
      dest: "~/test.txt"
      content: |
        line 01
           line 02 showing you can even put whitespace at start
```

- Using `copy` in this way will create a `test.txt` file in ~ with the contents given in `content`
  - Note it must be a file and it will create a new file if that one is not found
  - If that file is found, it will overwrite that file