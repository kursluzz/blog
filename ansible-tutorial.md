# Ansible tutorial

Ansible is an IT automation tool. It allows to define and control your cluster infrastructure by a configuration. This is my Ansible I'll go over the main consepts of this tool.

https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html

## Inventory

Let's create a basic inventory. A list of hosts in my LAN that I'm going to use. Previously I've set the hosts in `~/.ssh/config` in order to address the hosts by a name, and used ssh-copy-id in order avoid promptim password. You can use ip address as well.

```bash
vim /etc/ansible/hosts
```
file contents
```bash
shuttle
saturn
triton
```

## Run ping to all hosts
```bash
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

## Execute a command
	ansible all -a 'echo hello'

## Run a playbood task
In a dirictory create a yaml file.
```bash
$ vim myproject/mytask.yaml
---
- name: My task
  hosts: all
  tasks:
     - name: Leaving a mark
       command: "touch /tmp/ansible_was_here"
```
Then run 
```bash
$ ansible-playbook mytask.yaml
```

## Run as another user
You can pass `-u` in order to run a command as another user. 
```bash
# as bruce
$ ansible all -m ping -u bruce
# as bruce, sudoing to root (sudo is default method)
$ ansible all -m ping -u bruce --become
# as bruce, sudoing to batman
$ ansible all -m ping -u bruce --become --become-user batman
```

## Run with sudo
You can run commands as sudo with `--become`, but you need to ask for prompt password with `--ask-become-pass` or 
shortly `K`.
```bash
# shutdown all computers
ansible all --become -K -a "shutdown now"
```

## Specify parallel processes number
By default parallel number of processes is 5. If you want to increase it you can pass it in `-f` parameter.
```bash
ansible all --become -K -f 10 -a "shutdown now"
```

## Environment variables
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