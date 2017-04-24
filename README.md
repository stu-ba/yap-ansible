<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Orchestration of Ytrium server using Ansible](#orchestration-of-ytrium-server-using-ansible)
- [Installation](#installation)
  - [Requirements](#requirements)
  - [Control machine](#control-machine)
    - [Ubuntu](#ubuntu)
    - [macOS (via homebrew)](#macos-via-homebrew)
  - [Target machine](#target-machine)
- [Usage](#usage)
  - [Configuration](#configuration)
  - [Running server-setup.yaml playbook](#running-server-setupyaml-playbook)
- [Role rundown](#role-rundown)
    - [Common](#common)
    - [Taiga](#taiga)
    - [Yap](#yap)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

>Ansible is a radically simple IT automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs.

# Orchestration of Ytrium server using Ansible

<img src="https://img.shields.io/badge/Debian-Jessie_(8)-green.svg?style=flat-square"> <img src="https://img.shields.io/badge/Ansible-2.2.1.0-blue.svg?style=flat-square">

In this repository you can find self-explanatory group of `yaml` files describing installation of Ytrium server.

Ytrium orchestration consists of one [playbook](https://github.com/stu-ba/yap-ansible/blob/master/server-setup.yaml) and couple of roles ([common](https://github.com/stu-ba/yap-ansible/tree/master/roles/common), [taiga](https://github.com/stu-ba/yap-ansible/tree/master/roles/taiga), [yap](https://github.com/stu-ba/yap-ansible/tree/master/roles/yap)). Playbook simply checks if operation system is Debian 8 and runs roles in following order:

 1. Common [(read more)](https://github.com/stu-ba/yap-ansible#common)
 2. Taiga [(read more)](https://github.com/stu-ba/yap-ansible#taiga)
 3. Yap [(read more)](https://github.com/stu-ba/yap-ansible#yap)

You can read more about each role in [role rundown](https://github.com/stu-ba/yap-ansible#role-rundown) section.

# Installation

## Requirements

 - control machine 
     - Linux, Mac or BSD
     - ssh
     - Python 2.7
     - Ansible 2.2.1.0, [detailed installation guide](http://docs.ansible.com/ansible/intro_installation.html#installation)
 - target machine
     - Debian 8
     - sshd
     - sudo user
     
 
 >**Note**: Control machine is machine which is running Ansible. It can be your workstation or any server you have control of.
 
## Control machine

### Ubuntu

```
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible

$ ansible --version
```

### macOS (via [homebrew](https://brew.sh/))

```
$ brew install ansible
$ ansible --version
```

## Target machine

Setting up server from scratch is beyond scope of this document, check out free [screencasts](https://serversforhackers.com/series/security) for basic server setup.

# Usage

## Configuration

Before running Ansible make sure you read and understand following files: 

 - ./inventory
 - *./ansible.cfg* (skippable, factory-like)
 - ./roles/common/defaults/main.yaml
 - ./roles/taiga/defaults/main.yaml
 - ./roles/yap/defaults/main.yaml

>**Note**: `/roles/*/defaults/main.yaml` are **defaults** use `/roles/*/vars/main.yaml` to change desired key-pair value
> variable order of precedence (list is shorten):
> * role defaults
> * playbook vars
> * role vars
> * task vars
> * extra vars
>
> full list can be found in Ansible [documentation](http://docs.ansible.com/ansible/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)

## Running server-setup.yaml playbook

>**Note**: before running playbook check connection using ping module 
>
```
$ ansible INVENTORY-ENTRY -m ping
```
Where **INVENTORY-ENTRY** is group or single machine.

#### Running full playbook

```
$ ansible-playbook server-setup.yaml
```

#### Running tag specific playbook

```
$ ansible-playbook server-setup.yaml -t nginx,postgresql
```

>**Note**: each role has its own tag in case you want to run just specific role


# Role rundown

### Common

 - configures [/etc/gai.conf](https://github.com/stu-ba/yap-ansible/blob/master/roles/common/files/etc/gai.conf#L52-L54)
     - connecting to ipv4 servers ([source](http://askubuntu.com/a/787491))
 - updates && upgrades system
 - removes apache2
 - installs utilities 
     - (un)zip
     - git
     - curl
     - htop
 - installs && configures NGINX
     - [h5bp](https://github.com/h5bp/server-configs-nginx) configuration
 - installs php7.1-fpm from sury.org and extensions
    - php7.1-mbstring
    - php7.1-pgsql
    - php7.1-xml
    - php7.1-curl
 - installs postgresql-9.4
     - with python-psycopg2 (used by Ansible modules [postgresql_user](http://docs.ansible.com/ansible/postgresql_user_module.html) and [postgresql_privs](http://docs.ansible.com/ansible/postgresql_privs_module.html))
 - installs composer (php package manager)
 - installs redis
 - configures iptables
     - installs iptables-persistent
     - allows loopback traffic
     - allows port 22
     - allows established connections
     - makes INPUT chain default to DROP

#### Tags
`common`, `gai`, `apt`, `apt-update`, `apt-upgrade`, `apache2`, `nginx`, `nginx-install`, `php71-fpm`, `php71-fpm-install`, `postgresql`, `postgresql-install`, `composer`, `composer-install`, `redis`, `redis-install`, `iptables`
 

### Taiga

 - creates user (without password)
 - installs dependencies to run Taiga
 - configures PostgreSQL for Taiga
     - creates user && database
     - checks and disables unnecessary privileges
 - sets up taiga-back
     - downloads taiga-back from GitHub
         - sets up settings (local.py)
     - installs requirements in virtualenv
     - installs bcrypt package in virtualenv
     - runs django manage
 - sets up taiga-front
     - downloads taiga-front from GitHub
         - sets up settings (conf.json)
 - installs, configures service-manager (circus or systemd)
 - sets up NGINX configuration for Taiga 
 - configures iptables
     - opens port "taiga_server_port"
 

#### Variables
Problematic variables that need to be curated by server-admin.

`taiga_server_ip`

>**Note**: Ansible gathers facts about target machine, it uses `$ ip -4 route get 8.8.8.8` to get current IP.

#### Tags
`taiga`, `taiga-dependencies`, `taiga-database`, `taiga-back-install`, `taiga-front-install`, `taiga-nginx-config`, `taiga-iptables`, `taiga-systemd`, `nginx`, `service-manager`, `circus`, `systemd`, `apt`, `iptables`


### Yap **unfinished**


 - creates user (without password)
 - configures postgresql for Yap
     - sets up NGINX configuration for Yap 
 - configures iptables
     - opens port "yap_server_port"
 

## Todo:

 - [] properly differentiate between production and development environmnet
 - [] finish Yap orchestration (first install, updates) 


### Thanks to

 - [serversforhackers.com](https://serversforhackers.com/series)

 - [https://leucos.github.io/](https://leucos.github.io/ansible-files-layout)
