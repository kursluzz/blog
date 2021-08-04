# Ansible tutorial

Ansible is an IT automation tool. It allows to define and control your cluster infrastructure by a configuration. This is my Ansible I'll go over the main concepts of this tool.

https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html

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

### Run ping to all hosts
```
$ ansible all -m ping
```
output
```
saturn | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
triton | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
shuttle | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

### Execute a command
	ansible all -a 'echo hello'

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
You can run commands as sudo with `--become`, but you need to ask for prompt password with `--ask-become-pass` or 
shortly `K`.
```
# shutdown all computers
ansible all --become -K -a "shutdown now"
```

### Specify parallel processes number
By default parallel number of processes is 5. If you want to increase it you can pass it in `-f` parameter.
```
ansible all --become -K -f 10 -a "shutdown now"
```

### Environment variables
Detailed information can be found [here](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html)
Print all available environment variables:
```
- name: Print all available facts
  ansible.builtin.debug:
    var: ansible_facts
```
Print a raw data of a host:
```
ansible <hostname> -m ansible.builtin.setup
```
Print environment variable example:
```
- name: Basic usage
  debug:
    msg: "'{{ lookup('env', 'HOME') }}' is the HOME environment variable."
```