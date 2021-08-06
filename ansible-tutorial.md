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
 
### Apt install application
    ansible all -m apt -a "name=vim state=present" -b -K

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
Print environment variable example:
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