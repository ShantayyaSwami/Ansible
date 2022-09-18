<p align="center"><img src="https://i.imgur.com/xYb2PRb.png" /></p>


<p align="center">
    <a href="LICENSE.md">
      <img src="https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square" alt="Software License">
    </a>
</p>

# Ansible Core Concepts: 
## Invenories : 
* Inventory file simply consist of list of hostnames. 
* Default inventory location will be /etc/ansible/hosts but it can be configurable.
* It also possible to define group of hosts, hosts or group varibales and groups of groups within inventory.

## Dynamic inventory : 
* Speciying an executable file as inventory considered as dynamic inventory.
* Program must respond to 2 parameters: --list and --host.
* Using dynamic invenory, you can pull information from cloud,LDAP etc
 
## Modules : 
* Modules are essentially tools for perticuler tasks and can take (usually do) take parameters.
* They return the JSON and can run from CLI (passing -m) or within playbook. 
* Ansible supports significant no of modules. Custom modules can also be written. 
**Module list: ping, setup, yum , servic, user, copy, archive, unarchive, get_url, file, Git, Lineinfile, htpasswd, shell/command,script, debug, replace etc**

## Facts : 
* Gives certain information about target host. facts gathering can be disabled.
* it is possible to write custom facts in facts.d directory under /etc/ansible.

* **example: filename.fact + ansible all -m setup -a "filter=ansible_local"**

* All valid files ending with.fact are returned under ansible_local with facts.

## Variable : 
* Typically used for configuration values and verious parameters.
* Var name should be letter, number and underscore and should always start with letter. It can be scoped by group, host and even in playbok 
* Var also store values return by commands. Ansible has few predefined var.

## Plays :
#### set of instruction which can be defined as work or task to be done on target host.
* Play may use one or more modules to achieve the desired end state on group of hosts 
* --retry : To rerun playbook on failed machines only
* --limit : To run playbooks on only few machines or limit play execution on few machines.

## Playbooks : Series of plays

## Configuration files : 
#### config files can be seen at several possible locations (in order processed)
* **ANSIBLE_CONFIG (an env variable)**
* **ansible.cfg ( in current working dir)**
* **.ansible.cfg( in home dir)**
* **/etc/ansible/ansible.cfg** 

* Ansible config file has many parameters like inv file location, lib location, modules etc. all these values are commented and if w want to change default behavior then uncomment and change value. 
* Use -v in command to see which location config file is getting used.
* Ansible has the flexibility to have multi config file so that every time changing single file depend on  env is not required.

## Ad-hoc commands : 
#### ad-hoc commands are simply one liners 
**syntax: ansible <hostname or groupname> -i <inv-file> -m <module_name> -a <argument> -f <fork number: default is 5>**

## Fetching the output : 
#### using register keyword abd can use debug module to print var.

## Use conditionals to control play execution :
#### Handlers - notify, when , with_items, with_files
**Note: handlers are defined before tasks and notify will be called from task**

## Configure error-handling :
#### ignore_errors = yes, (defined under task name)block-rescue, always
* ignore_errors- ansible stops execution when perticular task fails but if we still want to continue with remaining steps then we can use ignore_errors. it will not cover connection or play execution issues.
* rescue executes when block fails

## Run specific tasks in playbooks using tags 
* tags comes under task name like notify keyword and get called with --tags in playbook execution
* **example: ansible-playbook <playbname> --tags <tagname>**

## Ansible template and template module. 
* All varibales available in playbooks can be referred in templates.
* Template gives the ability to provide skeleton file and than can be dynamically completed using variables. 
* Templates are generally used by providing template file on control node and then using template module in your playbook to deploy file to target node
* Templates are processed in jinja2 lang.

## Working with varibale : vars, vars_files, vars_promts
* Use -e '{"var1:value1",...,"var10":"value10"}' to provide var values or -e "@users.lst" -> users.lst is file containing lists
* var_files used to include var files in playbook.
* vars_promt throws promt for taking var as input.
* dictionary var - YAML formatting allows python style dictionaries to be used a var.
    
**employee:**
    
   **name: bob**
    
   **id: 123  **
    
* dictionary var can be accessed as: employee[name] or employee.name

## Magic var :
* Ansible defines special var called magic vars. It can use hostvars to look at the facts about other hosts in inventory.
* group variable that provides inventory info and jinja filters can be used to manipulate text format.
* **example :**
* **{{hostvars['node1']['ansible_distribution']}}**
* **{{groups['webservers']}}**
* **{{groups['webservers']|join(' ')}}**

## Ansible roles : 
* Roles are way of automatically loading vars, tasks, templates, handlers based on known file structure in playbooks.
* roles require perticuler dir structure and atleast require one dir with mail.yml
Example: roles directory includes below list of directories,
    
### **task** : 
Task dir should conatin main list of tasks to be executed by role.  main.yml considered as entry point of task section of role.
    
### **vars** : 
Contains variable used within role. var dir has highest level of precedence and can override inventry variables as well.
    
### **defaults** : 
Contains default variables for the role. It works similar as var directory but has lower level of precedence. this dir is only meant to provide value to var if no valu is given. var defined within role (using default or var dir) may be accessed across roles. you can pass var on CLI using -e option (highest precedence)
    
### **Handlers** : 
Contains handlers which used in role or anywhere outside this role. handlers are essentially tasks flagged to run by notify keyword. handler will only be triggered once even if they are notified by multiple tasks.
    
### **Files** : 
Contains ordinary files (not var files or templates) which can be deployed via this role. Files may be referrenced without path throughout role. 
    
### **Templates** : 
Templates inside role may be referrenced  without a path throughout role.
    
### **meta** : 
Defines certain meta data for this role. meta data includes role dependencies a db various role level configurations such as allow_dulicates etc

* Roles may inlcude other roles using dependencies keywork.
* Dependent roles are applied prior to role dependent on them.
* Roles with same parameters will not be applied more than once but having allow_duplicates:true can allow roles to be applied onre than once.

## ansible-galaxy : Ansible public repo
* ansible-galaxy is large public repo of ansible roles. roles ship with readmes detailing role use and avail var.
* contains large number of roles which are constantly evolving.  
* **'ansible-galaxy'** : can also create empty role using ansible-galaxy init <role_name>
* **ansible-galaxy install <username.role>** : used to donload roles from galaxy.ansible.com
* roles inslaled in roles_path may be listed using 'ansible-galaxy list' command
* Remove, search, login are other subcommands 

## Parallelism in ansible :
* Update forks = 100 in ansible.cfg file to run playbook on 100 worker nodes paralelly
* by Default, process will fork for 5 times.
* Can use -f to set forks in ansible ad-hoc command.
* **serial** keyword used in playbook to limit forks

## ansible-vault :Allows for encryption of file  
* **ansible-vault <encrypt/decrypt/edit/view> <file>**
* use --ask-vault-password or --ask-vault-file or --vault-id while playbook execution 
* It is possible to set no_log within module to censor sensitive log output.
* --vault-id introduced since ansible2.4 version

## ansible tower :
#### provides web server interface to ansible + free for minimal use (max 10 hosts)+ need enterprise edition for best use of it (user permissioning + autdit trail are benefits of anisble tower)


## Installation

```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
ansible --version
```

## Inventory

Example of Inventory file

```ini
[mysql]
10.0.0.13 = Env=live EcType=mysql EcName=live-mysql-0-b Az=a Nr=0
[nginx]
10.0.0.17 = Env=live EcType=nginx EcName=live-nginx-0-b Az=a Nr=0

[live:children]
mysql
nginx
```

## Check Connection

```yaml
$ ansible -m ping <host>
```

## Ad-Hoc Commands

#### Parallelism Shell Commands

```bash
ssh-agnet bash $ ssh-add ~/.ssh/id_rsa
```
> Reboot remove server using anisble
```yaml
ansible mysql -a "/sbin/reboot" -f 20
```
> Run ansible using specific user
```yaml
ansible nginx -a "/usr/bin/foo" -u shantayya
```
> Run ansible using specific user
```yaml
ansible nginx -a "/usr/bin/foo" -u shantayya
```
#### File Transfer

> Transfer file to many servers
```yaml
ansible nginx -m copy -a "src=/etc/shantayya.txt dest=/tmp/shantayya.txt"
```
> Transfer file with specific ownership & permission
```yaml
ansible nginx -m file -a "src=/etc/shantayya.txt dest=/tmp/shantayya.txt mode=600"
ansible nginx -m file -a "src=/etc/shantayya.txt dest=/tmp/shantayya.txt mode=600 owner=shantayya gorup=shantayya"
```
> Create Directories
```yaml
ansible nginx -m file -a "dest=/tmp/ mode=755 owner=shantayya gorup=shantayya stage=directory"
```
> Delete Directories
```yaml
ansible nginx -m file -a "dest=/tmp/ state=absent"
```

#### Manage Packages

> Ensure package is installed, but doesn't get updated
```yaml
ansible mysql -m apt -a "name=python state=present"
```
> Ensure package is installed to a specific version
```yaml
ansible mysql -m apt -a "name=python-2.6 state=present"
```
> Ensure package is installed with latest version
```yaml
ansible mysql -m apt -a "name=python state=latest"
```
> Ensure package is installed is not installed
```yaml
ansible mysql -m apt -a "name=python state=absent"
```

#### Manage Services

> Ensure a service is started on all nginx servers
```yaml
ansible nginx -m service -a "name=nginx state=started"
```
> Restart service on all nginx servers
```yaml
ansible nginx -m service -a "name=nginx state=restarted"
```
> Ensure a service is stopped
```yaml
ansible nginx -m service -a "name=nginx state=stopped"
```

## Playbooks

#### Playbook: Update system (Debian based)

```yaml
- hosts: local
  tasks:
    - name: Update system
      apt:
        update_cache: yes
        upgrade: yes
    - name: Remove dependencies
      apt:
        autoremove: yes
    - name: Remove useless packages from the cache
      apt:
        autoclean: yes
```

#### Playbook: List Kubernetes Cluster Nodes

```yml
- name: Kubernetes Cluster Health Check
  hosts: k8s
  become: true
  gather_facts: false
  tasks:
    - name: Checking the Kubernetes Nodes
      shell:
        kubectl get nodes 
      register: results

    - name: Print the Kubernetes Nodes
      debug:
        msg: "{{ results.stdout.split('\n') }}"
```

#### Sample Playbooks

```yaml
- name: dpkg --configure -a
  shell: dpkg --configure -a
  tags:
    - dpkg

- name: install system pakcages and utils
  apt:
    name: "{{ item }}"
    state: latest
    update_cache: yes
    cache_valid_time: 5400
    allow_unauthenticated: yes
  with_items:
   - ntp
   - git
   - git-core
   - htop
   - vim
   - curl
   - unzip
   - jq
   - python-setuptools
   - python-dev
   - build-essential
  tags:
    - packages
```

#### Writing Playbooks
> Create a playbook
```yaml
- hosts: live-node-01
  become: true
  roles:
    - { role: common,    tags: [ 'common'    ] }
    - { role: docker,    tags: [ 'docker'    ] }
    - { role: jenkins,   tags: [ 'agent'     ] }
    - { role: selenoid,  tags: [ 'selenoid'  ] }
```

#### Ansible Vault
> Using the argument “ — ask-vault-pass”
```
ansible-playbook users.yml --ask-vault-pass
```
> Using the argument “ — vault-password-file”
```
ansible-playbook users.yml --vault-password-file /anmol/.ansible/vault-passwd
```

More about Ansible:

- https://docs.ansible.com/ansible/latest/index.html

## 👬 Contribution
- Open pull request with improvements
- Discuss ideas in issues
