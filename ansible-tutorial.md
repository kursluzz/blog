# Ansible tutorial

Ansible is an IT automation tool. It allows to define and control your cluster infrastructure by a configuration. This is my Ansible I'll go over the main consepts of this tool.

https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html

## Inventory

Let's create a basic inventory. A list of hosts in my LAN that I'm going to use. Previously I've set the hosts in `~/.ssh/config` in order to address the hosts by a name, and used ssh-copy-id in order avoid promptim password. You can use ip address as well.

```
vim /etc/ansible/hosts
```

```
shuttle
saturn
triton
```

## Run ping to all hosts
```bash
$ ansible all -m ping
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

## Run as another user
You can pass `-u` in order to run a command as another user.

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