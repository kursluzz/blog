# Ansible tutorial

Ansible is an IT automation tool. It allows to define and control your cluster infrastructure by a configuration. This is my Ansible I'll go over the main concepts of this tool.

* Getting Started - https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html
* Modules - https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html

##  Documentation
### List of modules
    ansible-doc -l
### Get module documentation
    ansible-doc uri
### List specific specific type options
    ansible-doc -t become -l

## Inventory

Inventory file is a list of hosts and hosts groups. Here's some examples:

```
vim /etc/ansible/hosts
```
file contents
```
# hosts can be set in ~/.ssh/config
shuttle
saturn
triton

# as ip
[prod_servers]
10.0.0.5
10.0.0.6

# group 
[staging_servers]
svr1 ansible_host=172.1.2.3 ansible_user=ec2-user ansible_ssh_private_key_file=/home/ec2-user/.ssh/mykey.pem

# group of groups
[staging_prod:children]
staging_servers
prod_servers

# variables for a group
[staging_prod:vars]
ansible_user = ec2-user
ansible_ssh_private_key_file=/home/ec2-user/.ssh/mykey.pem
```
### View inventory
    # list of groups hosts and vars
    ansible-inventory --list
    # tree of groups and hosts
    ansible-inventory --graph
    
## Ad-Hoc commands
Ad-Hoc commands are commands that you run in command line and not in play-book

### Ping to all hosts
    ansible all -m ping

### Get remote variables
    ansible all -m setup
    # or full commant path
    ansible all -m ansible.builtin.setup

### Execute a command
	ansible all -m shell -a 'uptime'
	# or short version
	ansible all -a 'uptime'

### Copy file
    ansible all -m copy -a "src=myfile dest=~/myfile"
    
### Delete file
    ansible all -m file -a "path=~/myfile state=absent"
    
### Download file
    ansible all -m get_url -a "url=https://dlcdnet.asus.com/pub/ASUS/mb/socket775/P5K_SE/e3202_p5k-se.pdf dest=~/."

### Get uri similar to curl
    ansible all -m uri -a "url=https://asus.com"

### Apt install application
    ansible all -m apt -a "name=vim state=present" -b -K

### Service module
Start apache and autostart on startup.

    ansible all -m service -a "name=httpd state=started enabled=yes" -b -K
    
### Verbose debug
Add -vvv to see details of a command. You can use `-v, -vv, -vvv, -vvvv` to get more or less details.
    
    ansible all -m shell -a "ls" -vvv
    
### Run as another user
You can pass `-u` in order to run a command as another user. 
```
# as bruce
$ ansible all -m ping -u bruce
# as bruce, sudoing to root (sudo is default method)
$ ansible all -m ping -u bruce --become
# as bruce, sudoing to batman
$ ansible all -m ping -u bruce --become --become-user batman
```

### Run with sudo
You can run commands as sudo with `--become` (-b short), but you need to ask for prompt password with 
`--ask-become-pass` (short `K`). 
```
# shutdown all computers
ansible all -b -K -a "shutdown now"
```

### Specify parallel processes number
By default parallel number of processes is 5. If you want to increase it you can pass it in `-f` parameter.
```
ansible all --become -K -f 10 -a "shutdown now"
```

## Variables
### Variables in hosts file
    [<host/group>:vars]
    myvar = myvalue
    
### Variables in group files
In same directory with hosts file, crete directory group_vars, inside, place files with named as groups.
    
    mkdir group_vars
    vim group_vars/<group-name>

file content:
    
    myvar1 = myvalue1
    myvar2 = myvalue2

### Get defined variables
    ansible-inventory --list

### Use variables on remote host
    ansible mygroup -a 'echo {{ myvar }}'

## Playbooks

### Run a playbood task
In a dirictory create a yaml file.
```
$ vim myproject/mytask.yaml
---
- name: My task
  hosts: all
  tasks:
     - name: Leaving a mark
       command: "touch /tmp/ansible_was_here"
```
Then run 
```
$ ansible-playbook mytask.yaml
```
### Ping playbook task example
```
- name: Ping all servers
  hosts: all
  tasks:
    - name: ping server
      ping:
```
### Print environment variable
```
- name: Basic usage
  debug:
    msg: "'{{ lookup('env', 'HOME') }}' is the HOME environment variable."
```
### Environment variables
Detailed information can be found [here](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html)
Print all available environment variables:
```
- name: Print all available facts
  ansible.builtin.debug:
    var: ansible_facts
```
### Install apache
```
- name: Install apache web server
  hosts: saturn
  become: yes
  tasks:
    - name: Install apache web server
      apt: name=apache2 state=present
```
This will generate error "Missing sudo password". Set password as described bellow.
### Set sudo password 
A simple way to pass sudo password is to set variable in hosts file or in group_vars. 
```
ansible_sudo_pass: secret-pass
```
For a secured way and more details visit: https://docs.ansible.com/ansible/latest/user_guide/become.html

### Playbook variables
```
- name: Install apache web server
  hosts: saturn
  become: yes
  vars:
    app_name: apache2
  tasks:
    - name: Install apache web server
      apt: name={{ app_name }} state=present
```

### /usr/bin/python: not found
If on local machine there is python2 and on remote host it doesn't exist you will get this error. 
You can explicitly set python3 interpreter in the playbook.
```
...
  hosts: all
  vars:
    ansible_python_interpreter: /usr/bin/python3
  tasks:
    - name: ping server
...
```

## Pass variable to playbook
As example see how you can pass hosts on which you want to execute the playbook.

    - name: ...
      hosts: "{{ nodes }}"
      ...

then use `-e` to pass the variable

    ansible-playbook -e 'nodes=all' myplaybook.yaml

### Handlers
A handler is similar to a function. 
In this example we will call the handler only when a file is copied for the first time.
```
...
  tasks:
    - name: Copy a file
      copy: src=~/some-file dest=~/some-file
      notify: Do after copy

  handlers:
    - name: Do after copy
      command: reboot
```

### Debug
Print variables using debug
```
- name: Test debug
  hosts: all
  vars:
    foo: bar
  tasks:
    - name: Print foo
      debug:
        var: foo
    - debug:
        msg: Foo value is {{ foo }}
``` 
### set_fact - concat
Concat strings
```
vars:
  str1: abc
  str2: 123
tasks:
  - set_fact: result={{ str1 }}-{{ str2 }}
  - debug:
      msg: Result string is {{ result }}
```

### setup - print ansible environment variables
    ansible all -m setup
    
### register - print output of a command
  - shell: uptime
    register: uptime_result
  - debug:
      msg: Computer is up: {{ uptime_result.stdout }}
      
## Conditions
### When
```
    - name: Install apache
      apt: name=apache2 state=latest
      when: ansible_os_family == "Debian"
    - name: Install apache
      yum: name=httpd state=latest
      when: ansible_os_family == "RedHat"
```

## Blocks
You can gather multiple tasks to a block with one condition instead of repeating in all tasks.
```
- block:
    when: ansible_os_family == "Debian"
      - name: Install apache
        apt: name=apache2 state=latest
      - name: Another task
        ping:
- block:
    when: ansible_os_family == "RedHat"
      - name: Install apache
        yum: name=httpd state=latest
      - name: Another task
        ping:      
```

## Loops
### with_items / loop
with_items was replaced by loop since around version 2.5
```
- name: Loops playbook
  hosts: shuttle
  tasks:
    - name: Loop task
      with_items:
        - 1
        - 2
        - 3
      debug:
        msg: Item {{ item }}
```

### until
```
- name: Until example
  shell: echo -n Z >> myfile.txt
  register: output
  delay: 2
  retries: 10
  until: out.stdout.find("ZZZZ") == false
```
* delay 2 - optional. wait 2 seconds
* retries 10 - optional. max 10 iterations

### with_fileglob
```
- name: Loop through files in a dir
  with_fileglob: ~/*.*
  debug:
    var: {{ item }}
```
## Jinja templates

### Copy a file from jinja template
As example we will copy a configuration file with some host specific settings.

    vim settings.ini.j2

file contents

    ip = {{ ansible_default_ipv4.address }}
    hostname = {{ ansible_hostname }}

create playbook

    vim copy-settings-template.yaml

file content
```
- name: Copy settings template
  hosts: shuttle
  tasks:
    - template: src=settings.ini.j2 dest=~/settings.ini 
```

Template module is similar to copy 
